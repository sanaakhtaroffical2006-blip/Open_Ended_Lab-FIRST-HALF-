# Task 1 – Requirement Extraction (Baseline v1.0)

## 1.1 Functional Requirements (10)

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| FR-01 | Only authenticated operators shall be permitted to issue robot commands. | FR | High | Test (TC-01, TC-02, TC-03) |
| FR-02 | The robot shall accept a DeliveryRequest only in IDLE with a valid clinic destination, start navigation within 2 s, and reject requests in any other state with a reason code. | FR | High | Test (TC-01, TC-04) |
| FR-03 | On ObstacleDetected while NAVIGATING, the robot shall suspend navigation within 500 ms, enter AVOIDING_OBSTACLE, and resume navigation on ObstacleCleared. | FR | High | Test / Simulation (TC-05) |
| FR-04 | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | FR | High | Test (TC-07) |
| FR-05 | When chamber temperature or humidity leaves the batch profile range, the chamber shall start correction and declare recovery only after sensor readings are back in range for 30 s. | FR | High | Test (TC-08) |
| FR-06 | If correction fails within the recovery period (default 120 s), the chamber shall enter protection mode, reduce light exposure and alert the operator within 5 s. | FR | High | Test (TC-08) |
| FR-07 | Each robot shall send telemetry (state, position, battery, temperature, humidity, door) at least every 5 s. | FR | Medium | Test (TC-01, TC-12) |
| FR-08 | Every command, state transition and alarm shall be logged with timestamp, robot ID and operator ID, and be retrievable within 2 s. | FR | High | Test / Inspection (TC-01) |
| FR-09 | If the chamber door opens during conservation, normal operation shall be suspended and resumed only after door closed, sensors healthy and values in range are verified. | FR | High | Test (TC-09) |
| FR-10 | A batch may be removed only when the chamber is safe and no protection response is active; otherwise removal is refused and logged. | FR | Medium | Test (TC-09) |

## 1.2 Non-Functional Requirements (5)

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| NFR-01 | Performance: command acknowledgement shall reach the operator within 10 s over a link delay of up to 5 s; the authentication check shall take 1 s or less. | NFR (Performance) | High | Load / latency test |
| NFR-02 | Security: all command and telemetry traffic shall use TLS 1.2 or higher; an account shall lock after 5 failed logins. | NFR (Security) | High | Inspection / Test |
| NFR-03 | The system shall support communication with multiple robots simultaneously. | NFR (Scalability) | High | Load test (TC-12) |
| NFR-04 | Reliability: chamber monitoring availability of at least 99.5 % per mission; logs persisted within 2 s so no event is lost on power loss. | NFR (Reliability) | High | Analysis / Fault test |
| NFR-05 | Usability: a trained operator shall issue a delivery command in at most 5 steps and 30 s; alerts shall appear within 2 s using colour and text. | NFR (Usability) | Medium | Demonstration / Usability test |

## 1.3 High-Risk (Safety) Requirements (4)

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| SR-01 | Railway crossing: the robot shall not enter the crossing unless the ARLCCS reports barrier open and no train approaching, in a status message no older than 1 s. | Safety | Critical | Simulation (TC-10) |
| SR-02 | Link loss: if no heartbeat for 15 s, the robot shall enter LINK_LOST, stop outside any crossing and keep the chamber monitored. | Safety | Critical | Fault injection (TC-11) |
| SR-03 | Interlock: the robot shall never be moving (NAVIGATING, AVOIDING_OBSTACLE, RETURNING) with the chamber door open, and shall never enter DELIVERING directly from IDLE or AVOIDING_OBSTACLE. | Safety | Critical | Model check + Test (TC-04, TC-09) |
| SR-04 | If ARLCCS status is stale (more than 2 s), faulty, or its sensors disagree, the robot shall treat the crossing as closed and hold at least 5 m before it. | Safety | Critical | Fault injection (TC-10) |
