# Data schema (minimal, APP-compliant)

Design principle: **data minimisation**. Store only what the job intake and compliance evidence require; caller phone numbers are stored **hashed** with a per-tenant salt, and raw numbers are held only transiently for the active conversation. No caller data is sent to external services beyond the SMS gateway.

```sql
-- Tradie (tenant / accountable sender)
CREATE TABLE tradie (
    id              UUID PRIMARY KEY,
    business_name   TEXT NOT NULL,
    abn             CHAR(11) NOT NULL,          -- shown in every message
    mobile          TEXT NOT NULL,
    setup_completed_at TIMESTAMPTZ,             -- median target < 5 min
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Caller (minimised; phone hashed, never stored raw at rest)
CREATE TABLE caller (
    id              UUID PRIMARY KEY,
    tradie_id       UUID NOT NULL REFERENCES tradie(id),
    phone_hash      TEXT NOT NULL,              -- HMAC(number, per-tenant salt)
    first_seen_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    opted_out       BOOLEAN NOT NULL DEFAULT FALSE,
    opted_out_at    TIMESTAMPTZ,
    UNIQUE (tradie_id, phone_hash)
);

-- Missed-call event (the trigger)
CREATE TABLE missed_call_event (
    id              UUID PRIMARY KEY,
    tradie_id       UUID NOT NULL REFERENCES tradie(id),
    caller_id       UUID NOT NULL REFERENCES caller(id),
    occurred_at     TIMESTAMPTZ NOT NULL,
    source          TEXT NOT NULL               -- 'sim' | 'voip'
);

-- Consent ledger (evidence of inferred consent; demonstrable to a regulator)
CREATE TABLE consent_log (
    id              UUID PRIMARY KEY,
    caller_id       UUID NOT NULL REFERENCES caller(id),
    tradie_id       UUID NOT NULL REFERENCES tradie(id),
    basis           TEXT NOT NULL,              -- 'inferred_inbound_call'
    evidence_event  UUID REFERENCES missed_call_event(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Message (in/out; delivery + template for audit)
CREATE TABLE message (
    id              UUID PRIMARY KEY,
    event_id        UUID NOT NULL REFERENCES missed_call_event(id),
    direction       TEXT NOT NULL,              -- 'outbound' | 'inbound'
    template_id     TEXT,                       -- outbound only
    body            TEXT NOT NULL,
    sent_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
    delivery_status TEXT                        -- 'queued'|'delivered'|'failed'
);

-- Booking / job intake (the recovered value)
CREATE TABLE booking (
    id              UUID PRIMARY KEY,
    event_id        UUID NOT NULL REFERENCES missed_call_event(id),
    job_type        TEXT,
    location        TEXT,
    preferred_time  TEXT,
    status          TEXT NOT NULL DEFAULT 'captured', -- 'captured'|'confirmed'|'lost'
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

## Retention

- `caller`, `message`, `booking`: retained only for the pilot evaluation window, then aggregated and purged.
- `consent_log`: retained as compliance evidence per the tradie's obligations.
- Opt-out (`caller.opted_out = TRUE`) is **terminal** and suppresses all future messaging for that `(tradie, phone_hash)`.
