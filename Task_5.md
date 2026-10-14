# Task 5 – Constraints, Formalisation and Violations

## 5.1 Constraints (11), with logical notation

| ID | Constraint in simple English | Formal notation | Why necessary |
|---|---|---|---|
| C-01 | A train must never be present while the barrier is open. | Train_Present → ¬Barrier_Open | Prevents collision at the crossing. |
| C-02 | The robot may enter only when the barrier is open and no train approaches. | Robot_Entering → (Barrier_Open ∧ ¬Train_Approaching) | Core railway safety rule. |
| C-03 | The barrier opens only after the crossing is confirmed clear. | Barrier_Opening → Crossing_Clear_Confirmed | Avoids opening onto a train or a trapped vehicle. |
| C-04 | When a train approaches, warning lights and alarm must be on. | Train_Approaching → (Warning_Lights ∧ Audible_Alarm) | Warns people and robots. |
| C-05 | If ARLCCS has a sensor, barrier or communication fault, the robot must not enter. | ARLCCS_Fault → ¬Robot_Entering | Fail-safe under faults (sensor failure / comm loss). |
| C-06 | If the two train sensors disagree, assume a train is present. | (Sensor_A ≠ Sensor_B) → Train_Present_Assumed | Incorrect readings must not cause unsafe entry. |
| C-07 | The robot must not be moving with the chamber door open. | Robot_State ∈ {NAVIGATING, AVOIDING_OBSTACLE, RETURNING} → ¬Door_Open | Protects batch and cargo (robot-chamber link). |
| C-08 | The robot must not be delivering while chamber protection is active. | Protection_Active → ¬Robot_Delivering | A compromised batch must not be handed over (robot-chamber link). |
| C-09 | If the link is lost for more than 15 s, the robot is in LINK_LOST or SAFE_MODE and not entering the crossing. | (Link_Lost_s > 15) → (Robot_State ∈ {LINK_LOST, SAFE_MODE} ∧ ¬Robot_Entering) | An unsupervised robot must stop (communication loss). |
| C-10 | A critical temperature must put the robot in Safe Mode within 3 s. | (Temp_Critical ∧ Elapsed > 3 s) → Safe_Mode | CR-01 timing requirement. |
| C-11 | A command may execute only if the operator is authenticated and authorised. | Command_Executed → (Authenticated ∧ Authorised) | CR-03 security requirement. |

Robot-chamber constraints: C-07, C-08. Sensor failure / communication loss constraints: C-05, C-06, C-09.

## 5.2 Violation scenarios, detection and response

| ID | Violation (variable values) | How the system detects it | Expected response |
|---|---|---|---|
| C-01 | Train_Approaching = TRUE, Barrier_Open = TRUE | ARLCCS safety monitor evaluates the rule every cycle | Close barrier, sound alarm, stop robots, log event |
| C-02 | Robot_Entering = TRUE, Barrier_Open = FALSE | Robot guard plus ARLCCS status check | Emergency stop before the stop line, operator alert, log |
| C-03 | Barrier_Opening = TRUE, Crossing_Clear_Confirmed = FALSE | Safety-monitoring unit interlock | Inhibit opening, keep barrier closed, alert, log |
| C-04 | Train_Approaching = TRUE, Warning_Lights = TRUE, Audible_Alarm = FALSE | Output read-back compared with command | Activate alarm, raise ARLCCS fault, log |
| C-05 | ARLCCS_Fault = TRUE, Robot_Entering = TRUE | Fault flag or stale status (older than 2 s) | Robot holds, crossing treated as closed, operator alert |
| C-06 | Sensor_A = TRUE, Sensor_B = FALSE, Train_Present_Assumed = FALSE | Cross-check of redundant sensors | Assume train present, close barrier, log |
| C-07 | Robot_State = NAVIGATING, Door_Open = TRUE | Door sensor sampled together with robot state | Stop, enter SAFE_MODE, alert, log |
| C-08 | Robot_State = DELIVERING, Protection_Active = TRUE | Chamber status in telemetry | Abort delivery, go to RETURNING, alert, log |
| C-09 | Link_Lost_s = 20, Robot_State = NAVIGATING | Heartbeat timer | Enter LINK_LOST, stop outside the crossing, log |
| C-10 | Temp_Critical = TRUE, Elapsed = 5 s, Robot_State = NAVIGATING | Timer started on the critical flag | Force SAFE_MODE, alert operator, log |
| C-11 | Command_Executed = TRUE, Authenticated = TRUE, Authorised = FALSE | Audit check of the command log | Block or roll back command, security alert, log |

These rules are executable in `src/railway_monitor.py`, and each scenario is checked by `tests/test_constraints.py`.
