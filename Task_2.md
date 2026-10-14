# Task 2 – Change Requests and Traceability

## 2.1 Version-controlled change log (v1.0 to v1.1)

| CR | Req. ID | Version | Status | Old text | New text | Reason |
|---|---|---|---|---|---|---|
| CR-01 | FR-04 | 1.0 to 1.1 | **Modified** | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | The robot shall enter Safe Mode within 3 seconds when chamber or battery temperature exceeds the critical threshold or battery capacity falls below the emergency level. | Medicine spoils quickly; a time limit and thresholds are needed. |
| CR-02 | NFR-03 | 1.0 to 1.1 | **Modified** | The system shall support communication with multiple robots simultaneously. | The system shall support at least 20 simultaneously connected robots. | Pilot serves 20 clinics. |
| CR-03 | FR-01 | 1.0 to 1.1 | **Modified** | Only authenticated operators shall be permitted to issue robot commands. | The system shall require authenticated and role-authorised operators before accepting any robot command. | Authentication alone does not stop an authenticated Viewer from dispatching robots. |

## 2.2 Why the new wording is more measurable / testable

| CR | Explanation |
|---|---|
| CR-01 | The original has no time limit and no definition of "critical", so a late reaction cannot be called a failure. The new text gives a pass/fail limit (3 s), named triggers (chamber temperature, battery temperature, emergency capacity) and thresholds that a test can inject and time. |
| CR-02 | "Multiple" could mean 2 robots. "At least 20" is a number, so a load test can connect 20 simulated robots and check telemetry and command latency. |
| CR-03 | The original checks identity only. The new text has two separate checks (authentication AND role authorisation), giving distinct tests: valid user + valid role (accept), valid user + wrong role (reject), invalid user (reject). |

## 2.3 Robot operations used in the RTM

| ID | Operation |
|---|---|
| OP-C01 | AuthenticateAndAuthorise |
| OP-C02 | AcceptDeliveryRequest |
| OP-C03 | HandleObstacle |
| OP-C04 | EnterSafeMode |
| OP-C05 | SendTelemetry |
| OP-C06 | LogEvent |
| OP-C07 | RequestCrossingClearance |
| OP-C08 | DetectLinkLoss |

(OP-05 to OP-18 are chamber operations from Task 4.)

## 2.4 Multi-layer RTM

| Req. | State / Transition (Task 3) | Operation (Task 4 / 2.3) | Constraint (Task 5) | Test case | Risk level |
|---|---|---|---|---|---|
| FR-01 | Gate before T1 | OP-C01 | C-11 | TC-01, TC-02, TC-03 | High |
| FR-02 | T1; T-IDLE-DEL (invalid) | OP-C02 | C-08 | TC-01, TC-04 | High |
| FR-03 | T2, T3 | OP-C03 | none | TC-05 | High |
| FR-04 | T7–T10, SM1, SM2 | OP-C04 | C-10 | TC-06, TC-07, TC-13 | Critical |
| FR-05 | Chamber: MONITORING to CORRECTING | OP-05, OP-07 | C-08 | TC-08 | High |
| FR-06 | Chamber: CORRECTING to PROTECTION | OP-08 | C-08 | TC-08 | Critical |
| FR-07 | All active states | OP-C05 | C-09 | TC-01, TC-12 | Medium |
| FR-08 | All transitions | OP-C06 | none | TC-01 | Medium |
| FR-09 | Chamber: CONSERVING to SUSPENDED | OP-13, OP-14 | C-07 | TC-09 | High |
| FR-10 | Chamber: SAFE to removal | OP-18 | C-08 | TC-09 | Medium |
| NFR-03 | none | OP-C05 | none | TC-12 | High |
| SR-01 | NAVIGATING to crossing | OP-C07 | C-01, C-02, C-03 | TC-10 | Critical |
| SR-02 | LL1, LL2 | OP-C08 | C-09 | TC-11 | Critical |
| SR-03 | T-IDLE-DEL, T-AVOID-DEL (invalid) | OP-C02 | C-07 | TC-04, TC-09 | Critical |
| SR-04 | NAVIGATING to hold | OP-C07 | C-04, C-05, C-06 | TC-10 | Critical |

## 2.5 Test cases (command received, authenticated, executed, status reported, logged)

Scenario steps: **S1** command received, **S2** authenticated / authorised, **S3** executed, **S4** status reported, **S5** logged.

| Test | Steps | Description | Expected result |
|---|---|---|---|
| TC-01 | S1–S5 | Authenticated Dispatcher sends valid DeliveryRequest to an IDLE robot | Accepted, NAVIGATING within 2 s, status reported, log entry has operator ID |
| TC-02 | S1–S2 | Command with expired or invalid session | Rejected, robot unchanged, rejection logged |
| TC-03 | S2 | Authenticated Viewer sends DeliveryRequest | Rejected (role), logged |
| TC-04 | S3 | DeliveryRequest while robot is NAVIGATING or DELIVERING | Rejected with reason code; no IDLE to DELIVERING |
| TC-05 | S3–S5 | Obstacle detected, then cleared | AVOIDING_OBSTACLE within 500 ms, back to NAVIGATING, events logged |
| TC-06 | S3–S5 | CriticalBattery in NAVIGATING and in AVOIDING_OBSTACLE | Delivery abandoned, RETURNING, operator notified |
| TC-07 | S3–S5 | Chamber temperature above critical threshold | SAFE_MODE within 3 s, alert, log |
| TC-08 | S3–S5 | Temperature out of range: (a) correction works, (b) fails for 120 s | (a) recovery confirmed by sensor; (b) PROTECTION, light reduced, alert within 5 s |
| TC-09 | S3–S5 | Door opened during conservation, then batch removal attempted | Suspended; resumes only after verification; removal refused during protection |
| TC-10 | S3–S5 | ARLCCS reports train approaching / stale status / sensor mismatch | Robot holds before crossing, event logged |
| TC-11 | S4 | Heartbeat lost for 20 s | LINK_LOST, stops outside crossing, telemetry buffered |
| TC-12 | S3–S5 | 20 simulated robots send telemetry and receive commands | All delivered, latency within NFR-01 |
| TC-13 | S3–S5 | CriticalBattery in DELIVERING and in RETURNING | Handover completes then RETURNING; RETURNING stays and alerts |

## 2.6 Tests to re-run per change request

| CR | Must re-run | Reason |
|---|---|---|
| CR-01 | TC-06, TC-07, TC-11, TC-13 | Safe Mode timing, triggers and entry from every state |
| CR-02 | TC-12, TC-01 | Load at 20 robots; regression of the single-robot flow |
| CR-03 | TC-01, TC-02, TC-03, TC-04 | Authentication plus new role-authorisation check |
