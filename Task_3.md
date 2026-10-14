# Task 3 – State Modelling and Transition Verification

## 3.1 States

| ID | State | Meaning | Entry condition | Exit condition |
|---|---|---|---|---|
| S1 | IDLE | Powered on, waiting at warehouse | Switch-on or ArrivedWarehouse | DeliveryRequest accepted; Safe Mode trigger |
| S2 | NAVIGATING | Driving to destination | DeliveryRequest; ObstacleCleared | Obstacle, destination, battery, safe or link events |
| S3 | AVOIDING_OBSTACLE | Navigation suspended while avoiding obstacle | ObstacleDetected in NAVIGATING | ObstacleCleared; CriticalBattery; safe or link events |
| S4 | DELIVERING | Handing the package over at the clinic | DestinationReached | DeliveryComplete; safe or link events |
| S5 | RETURNING | Driving back to warehouse | DeliveryComplete; CriticalBattery; LinkRestored; OperatorReset | ArrivedWarehouse; safe or link events |
| S6 | SAFE_MODE | Stopped, chamber protected, waiting for operator | Thermal critical, battery emergency or critical fault | OperatorReset once conditions are normal |
| S7 | LINK_LOST | No heartbeat for more than 15 s; stopped in a safe place | LinkLost in an active state | LinkRestored or LinkTimeout; Safe Mode trigger |

## 3.2 Events / conditions

| ID | Event | Meaning |
|---|---|---|
| E1 | DeliveryRequest | Valid command from an authorised operator |
| E2 | ObstacleDetected | Sensors see an obstacle on the path |
| E3 | ObstacleCleared | Path is free again |
| E4 | DestinationReached | Robot is at the clinic |
| E5 | DeliveryComplete | Package handed over |
| E6 | CriticalBattery | Battery below the critical level (return to warehouse) |
| E7 | ArrivedWarehouse | Robot is back at base |
| E8 | ThermalCritical / BatteryEmergency / CriticalFault | Safe Mode triggers (CR-01) |
| E9 | LinkLost / LinkRestored / LinkTimeout | Link supervision |
| E10 | OperatorReset | Supervisor clears Safe Mode |

**State vs event:** a state is a condition the robot remains in for a period of time (it has duration and can be observed as "the robot is NAVIGATING"). An event is an instantaneous occurrence or condition (ObstacleDetected) that may cause the robot to change from one state to another.

## 3.3 State transition table

| ID | Current state | Event | Next state | Req. | Valid? | Verification evidence |
|---|---|---|---|---|---|---|
| T1 | IDLE | DeliveryRequest | NAVIGATING | FR-02 | Valid | TC-01; test_full_mission_trace |
| T2 | NAVIGATING | ObstacleDetected | AVOIDING_OBSTACLE | FR-03 | Valid | TC-05 |
| T3 | AVOIDING_OBSTACLE | ObstacleCleared | NAVIGATING | FR-03 | Valid | TC-05 |
| T4 | NAVIGATING | DestinationReached | DELIVERING | FR-02 | Valid | test_full_mission_trace |
| T5 | DELIVERING | DeliveryComplete | RETURNING | FR-02 | Valid | test_full_mission_trace |
| T6 | RETURNING | ArrivedWarehouse | IDLE | FR-02 | Valid | test_full_mission_trace |
| T7 | NAVIGATING | CriticalBattery | RETURNING | FR-04 | Valid | TC-06 |
| T8 | AVOIDING_OBSTACLE | CriticalBattery | RETURNING | FR-04 | Valid | TC-06 (added by verifier) |
| T9 | DELIVERING | CriticalBattery | DELIVERING (return after handover) | FR-04 | Valid | TC-13 (added by verifier) |
| T10 | RETURNING | CriticalBattery | RETURNING (operator alert) | FR-04 | Valid | TC-13 |
| T-IDLE-DEL | IDLE | DestinationReached | DELIVERING | SR-03 | **Invalid** | Not in model; illegal_direct() |
| T-AVOID-DEL | AVOIDING_OBSTACLE | DestinationReached | DELIVERING | SR-03 | **Invalid** | Not in model; illegal_direct() |

## 3.4 Extension: Safe Mode (CR-01) and Link Lost

| ID | Current state | Event | Next state | Req. | Valid? | Evidence |
|---|---|---|---|---|---|---|
| SM1 | IDLE, NAVIGATING, AVOIDING_OBSTACLE, DELIVERING, RETURNING, LINK_LOST | ThermalCritical / BatteryEmergency / CriticalFault | SAFE_MODE | FR-04 | Valid | TC-07; test_safe_mode_from_every_non_safe_state |
| SM2 | SAFE_MODE | OperatorReset | RETURNING | FR-04 | Valid | Manual test |
| LL1 | NAVIGATING, AVOIDING_OBSTACLE, DELIVERING, RETURNING | LinkLost | LINK_LOST | SR-02 | Valid | TC-11 |
| LL2 | LINK_LOST | LinkRestored / LinkTimeout | RETURNING | SR-02 | Valid | TC-11 |

**Which states may enter the new states**

| New state | Allowed from | Not allowed from | Decision reason |
|---|---|---|---|
| SAFE_MODE | Every state including IDLE and LINK_LOST | SAFE_MODE itself | A thermal or battery emergency endangers the medicine in any state, so safety overrides everything. |
| LINK_LOST | NAVIGATING, AVOIDING_OBSTACLE, DELIVERING, RETURNING | IDLE (robot only flags the link and rejects new requests), SAFE_MODE (already stopped) | Only a moving or active robot needs to stop when unsupervised. |

## 3.5 Verifier investigation

| Question | Finding | Action |
|---|---|---|
| (a) Can IDLE to DELIVERING happen? | Not in the model. If it existed it would violate SR-03 and FR-02 (a waiting robot must never begin delivering directly): a package would be handed over without any navigation, route or crossing checks. | Keep invalid; automated check illegal_direct(). |
| (b) AVOIDING_OBSTACLE has no transition back? | Deadlock. The only exit is ObstacleCleared. Without it the robot never reaches IDLE and can never deliver (DELIVERING is reachable only through NAVIGATING). It stays stuck while the chamber drains the battery. | Mutant test test_mutant_without_obstacle_cleared_deadlocks. |
| (c) Can AVOIDING_OBSTACLE to DELIVERING occur? | No. It would break "never deliver while handling an obstacle". The robot must return to NAVIGATING and reach the destination first. | Invalid row T-AVOID-DEL. |
| (d) CriticalBattery in AVOIDING_OBSTACLE or DELIVERING? | The baseline has no transition, so the event is ignored while the battery keeps draining. The chamber may lose power and the medicine spoils (unsafe, missing transition). | Added T8, T9, T10; verified by missing(). |
