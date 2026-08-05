# Model / prompt card — LLM message step

**Purpose:** compose a short, compliant SMS reply and extract light job-intake fields (what / where / when) from a caller's reply. The LLM does **not** decide whether to message — that is gated by the consent/opt-out ledger and business-hours rules upstream.

**Inputs:** business name, ABN, caller's inbound text (transient), template context.
**Outputs:** message body (must include business name + ABN + STOP), and structured intake fields.

**Constraints & guardrails**
- Templates constrain tone and content; the model fills slots, it does not free-compose outbound marketing.
- Confidence below threshold on intent → hand off to the tradie rather than guess.
- No caller identifiers are persisted to the model provider beyond the transient request.

**Limitations:** English-only MVP; not a diagnostic or quoting tool; does not negotiate price or commit to appointment times — it captures intent and promises a human callback.

**Responsible-AI mapping (NIST AI RMF):** Govern/Manage via the compliance checklist + consent ledger; Measure via pilot metrics; Map via this card.
