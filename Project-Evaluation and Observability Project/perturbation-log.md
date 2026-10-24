# Perturbation Log

For each system, make one deliberate change to an input or configuration, predict the outcome, run
it, and record what actually happened. See the starters in the Instructions, or design your own (your
own experiment earns more credit).

---

### System 1 — validated, routed pipeline

- **Change I made (file + what I changed):** Used the existing test scenario in `tests/test_us01_retry.py` that sets the required `endorsements` field to `None`, representing a missing Schedule A source.
- **Command I ran:** `.venv/bin/pytest tests/test_us01_retry.py -v`
- **What I predicted:** The pipeline would identify the `endorsements_absent` pattern and escalate immediately instead of retrying when the source information was missing.
- **What actually happened (paste the key output line):** `test_ac_01_04_missing_source_halts_immediately PASSED`; the test asserts `RetryFutileEscalation`, `field == "endorsements"`, `detected_pattern == "endorsements_absent"`, and `client.call_count == 1`. Test suite: `13 passed, 1 skipped`.
- **How this differs from the unperturbed run:** The missing-source case stops after one API call rather than attempting further retries. This is a test-based experiment using a recorded client, not a live API run. The live extraction test was skipped because `ANTHROPIC_API_KEY` was not set.

---

### System 2 — schema-enforced two-pass extraction

- **Change I made (file + what I changed):** Created `/tmp/income_sum_corrected.txt` from `fixtures/documents/income_sum_mismatch.txt` and changed only the stated `TOTAL MONTHLY EARNINGS` from 10,892.17 to 9,642.17. The original fixture was not modified.
- **Command I ran:** `.venv/bin/mortgage-extract /tmp/income_sum_corrected.txt --mode replay` (full extraction could not run because no recorded response existed for the changed input); then ran an offline Python check using the project's `Income`, `MortgageExtraction`, and `validate()` implementation to compare the original and corrected totals.
- **What I predicted:** The original total would fail consistency validation by $1,250.00, while the corrected total would match the calculated component sum and pass.
- **What actually happened (paste the key output line):** Original: `Consistent: False`, discrepancy `{'field': 'total_monthly_income', 'calculated': 9642.17, 'stated': 10892.17, 'delta': -1250.0}`. Corrected: `Consistent: True`, `Discrepancies: []`.
- **How this differs from the unperturbed run:** Correcting the stated total removed the $1,250.00 arithmetic discrepancy in the real offline validator. The changed document's full replay extraction was not verified because its request had no cached response; the result above isolates and verifies the validator behavior without an API call.

---

### System 3 — multi-source synthesis

- **Change I made (file + what I changed):** Enabled the supply-chain coordinator's simulated logistics timeout using the `--simulate-timeout` option; no source files were edited.
- **Command I ran:** `.venv/bin/supply-chain-investigate meridian --offline --simulate-timeout`
- **What I predicted:** The coordinator would mark logistics-dependent evidence unavailable, avoid presenting the missing logistics values as confirmed facts, and identify metrics that could no longer be corroborated.
- **What actually happened (paste the key output line):** `Sources unavailable: logistics unavailable (timeout)`. The briefing marked `late_shipment_count` as `[missing source: timeout reading logistics]`; `on_time_delivery_rate` had only the supplier_audit value of 95.0 percent; and `production_capacity_utilization` remained incomplete because no source reported it.
- **How this differs from the unperturbed run:** The normal offline run included logistics evidence and showed a contested on-time delivery rate of 95.0 percent from supplier_audit versus 78.0 percent from logistics. With the timeout, the logistics value was unavailable, so the conflict disappeared from the report but the uncertainty did not mean the 95.0 percent value was independently confirmed. The briefing explicitly reported the source timeout.
