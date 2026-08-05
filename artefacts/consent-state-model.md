# Consent state model

State machine for a `(tradie, caller)` relationship. Opt-out is terminal.

```
                 genuine inbound missed call
                             │
                             ▼
                    ┌─────────────────┐
                    │ INFERRED_CONSENT│  (consent_log written)
                    └────────┬────────┘
             business hours + not opted out
                             ▼
                    ┌─────────────────┐
                    │    MESSAGED     │  (first SMS: name + ABN + STOP)
                    └───┬────────┬────┘
            reply       │        │        STOP received
        ┌───────────────┘        └───────────────┐
        ▼                                          ▼
┌────────────────┐   no reply (1 follow-up) ┌────────────────┐
│   RESPONDED    │        then stop         │   OPTED_OUT    │ ← terminal
│ (job captured, │                          │ (suppress all  │
│ tradie alerted)│                          │  future msgs)  │
└───────┬────────┘                          └────────────────┘
        ▼
┌────────────────┐
│  NO_RESPONSE   │  (sequence ends after single follow-up)
└────────────────┘
```

## Rules

1. A caller enters `INFERRED_CONSENT` **only** via a genuine inbound missed call — never a purchased or scraped number.
2. `MESSAGED` is skipped outside business hours or if `opted_out = TRUE`.
3. `STOP` at any point → `OPTED_OUT` (terminal); logged with timestamp; honoured automatically.
4. Maximum **one** follow-up after the first message; then the sequence ends.
5. The tradie can pause or take over at any state (human-in-the-loop override).
