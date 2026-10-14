# Task 4 – Operation Identification and Schema (Conservation Chamber)

## 4.1 Chamber states (used for state changes)

| State | Meaning |
|---|---|
| OFF | No power |
| SELF_CHECK | Power-on sensor test |
| MONITORING | All essential sensors OK, waiting for batch and closed door |
| CONSERVING | Active conservation with batch loaded |
| CORRECTING | Temperature or humidity being corrected |
| PROTECTION | Correction failed; light reduced, operator alerted |
| SUSPENDED | Normal operation paused (door open or high vibration) |
| POWER_EMERGENCY | Running on emergency power |
| SAFE_SHUTDOWN | Incident recorded and chamber shut down safely |

## 4.2 Operations (18)

| ID | Operation | Purpose |
|---|---|---|
| OP-01 | RunSelfCheck | Test all essential sensors at power-on |
| OP-02 | LoadBatchProfile | Load batch ID and temperature/humidity limits |
| OP-03 | StartConservation | Start active conservation when door is closed and profile loaded |
| OP-04 | ReadSensors | Sample temperature, humidity, vibration, door, power |
| OP-05 | CorrectTemperature | Drive heater/cooler toward range |
| OP-06 | CorrectHumidity | Drive humidifier/dehumidifier toward range |
| OP-07 | VerifyRecovery | Confirm by sensor readings that values are back in range |
| OP-08 | ActivateProtection | Enter protection response after failed recovery |
| OP-09 | ReduceLightExposure | Dim or close the light path |
| OP-10 | AlertOperator | Send alert to Mission Control |
| OP-11 | SuspendRiskyActivities | Stop risky actions on significant vibration |
| OP-12 | ResumeAfterStabilisation | Resume when vibration stays below threshold for the stabilisation period |
| OP-13 | SuspendOnDoorOpen | Suspend normal operation when the door opens |
| OP-14 | VerifyAndResume | Resume only after door, sensors and values are verified |
| OP-15 | SwitchToEmergencyPower | Change to backup supply on power loss |
| OP-16 | RecordIncident | Write incident to non-volatile log |
| OP-17 | ShutdownSafely | Safe shutdown when no emergency power exists |
| OP-18 | AuthoriseBatchRemoval | Allow removal only if chamber safe and no protection active |

## 4.3 Operation schemas (8)

| Field | RunSelfCheck | LoadBatchProfile | StartConservation | CorrectTemperature |
|---|---|---|---|---|
| Description | Check each essential sensor at power-on | Load batch identity and limits | Begin active conservation | Heat or cool toward profile range |
| Inputs | power_on | batch_id | none | target range |
| Outputs | self_check_result | profile_loaded | conservation_active | correction_status |
| Reads | sensor status | batch database | door, profile_loaded | temperature sensor |
| Changes | chamber_state | profile | chamber_state | actuator output |
| Sends / Alerts | Alert on failure | Operator acknowledgement | Status to robot | none |
| Assumes (pre) | chamber_state = OFF | State = MONITORING, batch present | door_closed ∧ profile_loaded ∧ MONITORING | CONSERVING ∧ temperature out of range |
| Result (post) | All essential sensors OK gives MONITORING; else FAULT and alert | profile_loaded = TRUE with limits | chamber_state = CONSERVING | Command issued; success NOT assumed until VerifyRecovery |

| Field | ActivateProtection | SuspendOnDoorOpen | SwitchToEmergencyPower | AuthoriseBatchRemoval |
|---|---|---|---|---|
| Description | Protect batch after failed recovery | Suspend normal operation on door open | Handle power loss | Allow batch removal |
| Inputs | recovery_timeout | door_open event | power_lost event | removal_request |
| Outputs | protection_active | suspended | power_source | removal_allowed |
| Reads | temperature, humidity, light | door sensor | power sensor, backup level | chamber_state, protection flag |
| Changes | chamber_state, light | chamber_state | power_source | none |
| Sends / Alerts | Operator alert within 5 s | Alert; robot informed | Alert; incident log | Log; refusal alert |
| Assumes (pre) | Correction active longer than recovery period ∧ still out of range | CONSERVING | power_lost | chamber_state = SAFE ∧ ¬protection_active |
| Result (post) | PROTECTION; light reduced; alert sent; incident logged | SUSPENDED; robot must stop (C-07) | Backup available gives POWER_EMERGENCY; else incident recorded then ShutdownSafely | Removal allowed; otherwise refused with reason and logged |

## 4.4 Chamber state changes linked to the Task 3 robot model

| Operation | Chamber state change | Robot state link |
|---|---|---|
| SuspendOnDoorOpen | CONSERVING to SUSPENDED | A moving robot (NAVIGATING / AVOIDING_OBSTACLE / RETURNING) goes to SAFE_MODE (C-07). At the destination, opening the door is the authorised handover. |
| ActivateProtection | CORRECTING to PROTECTION | DELIVERING goes to RETURNING (C-08); robot returns to warehouse with alert. |
| SwitchToEmergencyPower | CONSERVING to POWER_EMERGENCY | Robot goes to RETURNING, or SAFE_MODE on BatteryEmergency. |

## 4.5 Postconditions that must be verified by sensor readings

| Operation | Why it cannot be assumed |
|---|---|
| CorrectTemperature then VerifyRecovery | Issuing a heater/cooler command only shows the command was sent. The actuator may be broken or too weak. Only temperature readings back in range (for example for 30 s) prove the batch is safe. |
| VerifyAndResume (after door open) | A door-closed signal does not prove the batch is cold again or that sensors still work. Temperature and humidity must be read in range and sensors confirmed healthy before normal operation resumes. |
