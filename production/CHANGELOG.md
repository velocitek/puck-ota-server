# Firmware Changelog — production

## 2026-08-26
STM32 v1.0.8.8  
ESP32 v1.0.8.8  
PCB Rev: D

Source: 03db9c3d

- dummy version rev to test OTA for 1.0.8.7 & 1.0.8.8

## 2026-08-26
STM32 v1.0.8.7  
ESP32 v1.0.8.7  
PCB Rev: D

Source: e935c2e1

- Firmware (STM32)
- Base now logs its position to VTK: 1 Hz VtkTrackpoint per PPS + the RTCM 1006 surveyed ARP (sm_base_station.c)
- New portable RTCM 1006 parser: CRC-24Q validated, 38-bit ECEF → WGS84 (Bowring, double), zero-ARP guarded (rtcm_1006.c/h)
- New VTK record VtkBaseSurveyedPosition (oneof tag 31), schema 3→4, nanopb regenerated
- Surveyed emits: once per boot + on-change, rate-limited to 1/5 s while wandering, trailing frozen value guaranteed, no epoch-zero latch
- RTCM hardening: frame_offset on RtcmMessage, caller-bounded enumeration (was a latent stack overflow), edge-triggered table-full warning
- Five review-found reliability fixes: sm_mark Unix-millis trackpoint bug; cal-retry marker misdiagnosis (code 6 vs 10 reboot loop); VtkSendStmReset success-latch; stale noinit recovery-cell gate in fault_record; staged I2C packet age-out (scoped to TIME_SYNC only)
- VtkSendTrackpointNow() captures the anchor clock internally (kills the hand-supplied-millis bug class); VTK_SCHEMA_VERSION named constant
- Decoders
- decode_vtk.py + puck_manager.html: wall-clock anchor now re-anchors on every session header (multi-boot files rendered 128 s off before); schema checked per header; tag 31 rendering (SURVEYED_POS, --type surveyed, MID-SURVEY marker); byte-parity between the two verified
- New pytest module (wire format, rendering, anchor, schema cross-check vs firmware)
- Docs
- All four VTK docs to schema 4: §6.9 for tag 31, §8 multi-session anchoring rule, the "post-survey base track is the pinned ARP, not a live track" warning, stale VtkSendStmReset/missing VtkSendStmFault rows fixed, Appendix A re-pasted byte-exact, HTML pandoc-regenerated

## 2026-08-25
STM32 v1.0.8.6  
ESP32 v1.0.8.6  
PCB Rev: D

Source: 942691bc-dirty

- -FW Version bump to test OTA on latest FW - 1.0.8.5 = 1.0.8.6

## 2026-08-25
STM32 v1.0.8.5  
ESP32 v1.0.8.5  
PCB Rev: D

Source: 85450d6d-dirty

- Outgoing-to-ESP32 ring made interrupt-safe: every ring mutation now runs in a critical section, and the I2C consumer copies packets out atomically instead of transmitting from live ring memory — fixes frame corruption, mid-transfer overwrites, and queue-wedge races between the main loop, radio IRQ, and LED IRQ (the calibration-hang defect class, live in the race path). Worst-case interrupt-masked window measured ~450 µs (typical < 100 µs).
- Tests: native two-thread concurrency hammer (400k packets, zero corruption) + on-target ISR stress harness (RTC-interrupt producer vs main-loop consumer).

## 2026-08-25
STM32 v1.0.8.4  
ESP32 v1.0.8.4  
PCB Rev: D

Source: 5ce77e0b-dirty

- STM32 firmware
- OTA reliability: boot check-in now runs before calibration retrieval. Pucks with missing/invalid calibration data (never-calibrated spares, burn-in sentinels) no longer fail OTA verification with a phantom "missed check-in." Calibration retrieval also gained bounded retries, and failures split into two flash codes: new code 10 = bad/missing calibration data (permanent, needs field cal), code 6 = actual boot-comms failure (auto-recovers).
- IWDG watchdog + fault handlers. Crashes and hangs now self-recover in ~16 s instead of requiring a power cycle.
- Crash forensics on the SD card. Faults write a new STM_FAULT record to the VTK log (faulting PC, fault status register, fault type, last main-loop breadcrumb) — diagnosable after race day, not just on a live console.
- Every STM_RESET record now carries the last main-loop breadcrumb, so repeated resets of a misbehaving puck cluster on the code section that hung (0 = cold boot / older firmware).
- LoRa RX hardening: wire-declared lengths and chunk counts are now bounds-checked, closing the unbounded-packet-length exposure from the August reliability review..

## 2026-08-25
STM32 v1.0.8.3  
ESP32 v1.0.8.3  
PCB Rev: D

Source: ec6f4b90

- **Line-end link telemetry** (`db1ca5d2c` + `ec6f4b909`, unreleased)
- **Marks now monitor each other.** RC and Pin report how well they hear the opposite end, so the app can flag a mark whose transmit range has collapsed — which the base cannot see from alongside it. No airtime cost.
- **`MarkReport` layout changed** (still 25 bytes). **Flash both marks together**; a mixed pair reports "link unknown".
- **Base publishes silence instead of stale data** when a report fails to parse (it was re-sending the previous period's as fresh), and no longer ACKs a mark slot it couldn't decode.
- **Mark's "RSSI seen by rover" corrected** — it was reporting whatever frame was last received rather than the base's broadcast.
- **Slot receive-timeout underflow fixed** — could open a 65-second RX window at a slot boundary.
- **Caveat:** the RC's receiver duty rises ~56% → ~97%; unmeasured, needs a bench

## 2026-08-20
STM32 v1.0.8.2  
ESP32 v1.0.8.2  
PCB Rev: D

Source: c6213bc2

- quick bump to test OTA updates on this 8.1 & 8.2

## 2026-08-20
STM32 v1.0.8.1  
ESP32 v1.0.8.1  
PCB Rev: D

Source: d833ef11

- adds base race update summary logging to BASE SD card. untested

## 2026-08-05
STM32 v1.0.7.6  
ESP32 v1.0.7.6  
PCB Rev: D

Source: 1adc6ec6

- reverting back to rev d (same as v1.0.7.4)

## 2026-08-05
STM32 v1.0.7.5  
ESP32 v1.0.7.5  
PCB Rev: B

Source: b9bbe2ab

- special Rev B release for Wind Rudder update

## 2026-08-04
STM32 v1.0.7.4  
ESP32 v1.0.7.4  
PCB Rev: D

Source: 54920c86-dirty

- dummy release for demo purposes

## 2026-08-04
STM32 v1.0.7.3  
ESP32 v1.0.7.3  
PCB Rev: D

Source: d4e1d6f7-dirty

- Provisioning version no longer orders images — any CRC-valid image is accepted, so a base reused from a past regatta no longer silently rejects a new fleet's v1
- Blank/all-zero provisioning images now rejected (they pass a CRC check on their own)
- Hardcoded provisioning is now purely a fallback for blank or corrupt EEPROM
- Fixed stale pending provisioning surviving a GPS dropout and being re-broadcast over newer data on re-entry to base state
- ESP32 BLE mirror now reflects only what the STM32 actually committed, so the web app's provisioning sync check can be trusted
- Fixed ~1 Hz "Invalid packet" log spam on any base waiting for GPS
- Burn-in LoRa summary no longer hangs part way through printing

## 2026-07-23
STM32 v1.0.6.5  
ESP32 v1.0.6.5  
PCB Rev: B

Source: 92997aac

- TracTrac Specific Release - latest OTA Provisioning FW with TracTracc trac hard_coded_provisioning_fallback. No Calibration missing workaround

## 2026-07-10
STM32 v1.0.6.4  
ESP32 v1.0.6.4  
PCB Rev: D

Source: 46061356-dirty

- rev D, same as v.1.0.6.2

## 2026-07-10
STM32 v1.0.6.3  
ESP32 v1.0.6.3  
PCB Rev: B

Source: 5054126a-dirty

- Updating rev B puck to latest FW

## 2026-07-02
STM32 v1.0.6.2  
ESP32 v1.0.6.2  
PCB Rev: D

Source: 2662445d-dirty

- last deploy failed on github...

## 2026-07-02
STM32 v1.0.6.1  
ESP32 v1.0.6.1  
PCB Rev: D

Source: b1899bec-dirty

- Release Candidate for Sail Newport 2026
- OTA Reliability Improvements - base broadcasting, stm verification
- GPS time sync now driven from Base puck
- Finish line improvement (no turning blue)
- ms accurate cross time now added
- LED ring ghost light fix

## 2026-07-01
STM32 v1.0.5.138  
ESP32 v1.0.3.138  
PCB Rev: D

Source: 6fbccec4-dirty

- removes finish state (no more Blue lights on finishing)
- reduces Base OTA Broadcast to 30seconds

## 2026-06-29
STM32 v1.0.5.136  
ESP32 v1.0.3.136  
PCB Rev: D

Source: 77f7de66-dirty

- adds LED fix for post OTA update

## 2026-06-29
STM32 v1.0.5.135  
ESP32 v1.0.3.135  
PCB Rev: D

Source: a7b8fa57-dirty

- fixes slow >255 log loading

## 2026-06-29
STM32 v1.0.5.134  
ESP32 v1.0.3.134  
PCB Rev: D

Source: d6e1f9df-dirty

- reverts to keeping legacy track count list and delete file mechanisms to preserve Charted sails usability 

## 2026-06-26
STM32 v1.0.5.132  
ESP32 v1.0.3.132  
PCB Rev: D

Source: 57030e13-dirty

- removes 60s finish crossing timer, now records actual start crossing times

## 2026-06-26
STM32 v1.0.5.129  
ESP32 v1.0.3.129  
PCB Rev: D

Source: ddcdb99e-dirty

- adds more robust LED code to avoid Ghost LEDs

## 2026-06-25
STM32 v1.0.5.127  
ESP32 v1.0.3.127  
PCB Rev: D

Source: 25f26549-dirty

- adds Timer Sync improvements - all time based on Puck PPS
- adds new I2C timer sync traffic for timer syncronization between stm & esp
- new BLE characteristic for time syncing with prosmart app

## 2026-06-22
STM32 v1.0.5.125  
ESP32 v1.0.3.125  
PCB Rev: D

Source: 57a8c389-dirty

- testing the OTA Update LEDs

## 2026-06-22
STM32 v1.0.5.124  
ESP32 v1.0.3.124  
PCB Rev: D

Source: 4b52a571-dirty

- more testing

## 2026-06-22
STM32 v1.0.5.123  
ESP32 v1.0.3.123  
PCB Rev: D

Source: e2762e14-dirty

- FW version bump to test

## 2026-06-22
STM32 v1.0.5.122  
ESP32 v1.0.3.122  
PCB Rev: D

Source: ce4c8794-dirty

- testing STM FW Verifcation and I2C Buffer improvements

## 2026-06-18
STM32 v1.0.5.121  
ESP32 v1.0.3.121  
PCB Rev: D

Source: 4f50f340

- finalizes >255 Track count fix - TESTED
- also improves mass delete functionality

## 2026-06-18
STM32 v1.0.5.120  
ESP32 v1.0.3.120  
PCB Rev: D

Source: 1d23c007

- fixes 255 track limit on Puck FW and puck_manager
- this was a BT protocol issue so will require ChartedSails & TracTrac updates
- UNTESTED - this release is to complete initial testing

## 2026-06-03
STM32 v1.0.5.119  
ESP32 v1.0.3.119  
PCB Rev: D

Source: f9946b34-dirty

- Now with the right Commit to include the below updates to fix ChartedSails No Track issue

## 2026-06-03
STM32 v1.0.5.118  
ESP32 v1.0.3.118  
PCB Rev: D

Source: f2a8152c-dirty

- hopefully fixes CHartedSails NO TRACKS found error

## 2026-06-03
STM32 v1.0.5.117  
ESP32 v1.0.3.117  
PCB Rev: D

Source: 8df85201

- adds more retries for STM Checkin to improve STM Verifcation on OTA reliability

## 2026-06-03
STM32 v1.0.5.116  
ESP32 v1.0.3.116  
PCB Rev: D

Source: 0ee2a4d1

- fixes OTA stack overflow issues
- changes OTA LED flashes to 1 in 6 LEDs

## 2026-06-03
STM32 v1.0.5.115  
ESP32 v1.0.3.113  
PCB Rev: D

Source: e4741838-dirty

- OTA Testing new v112 OTA capability on ESP

## 2026-06-03
STM32 v1.0.5.114  
ESP32 v1.0.3.112  
PCB Rev: D

Source: 54a0c52d

- testing OTA on ESPv111

## 2026-06-02
STM32 v1.0.5.113  
ESP32 v1.0.3.111  
PCB Rev: D

Source: 1e664dd1-dirty

- Adds Session Header to every log file (fixes CHartedSails NoFix error)
- New log files now created when start times are set not on actual start times (ie timer = 0)
- adds STM & ESP reset on ESP Error states - tries 3x before persisting error

## 2026-05-28
STM32 v1.0.5.111  
ESP32 v1.0.3.109  
PCB Rev: D

Source: 0858188d

- fixes NO ROLE -> WITH ROLE provisioning reset. No longer requires power cycle after being provisioned with a role

## 2026-05-20
STM32 v1.0.5.107  
ESP32 v1.0.3.105  
PCB Rev: D

Source: 6a223fd6

- first "Production" release of: Finishes, Resurvey Base, and Read Provisioning changes

## 2026-05-15
STM32 v1.0.5.103  
ESP32 v1.0.3.101  
PCB Rev: D

Source: e929fcdc-dirty

- FLEET WAVES -> COMPETITOR 1 -> 001
- TIDE SHELLS -> COMPETITOR 2 -> 002

## 2026-05-14
STM32 v1.0.5.102  
ESP32 v1.0.3.100  
PCB Rev: D

Source: cfadbdbe

- Fleet Waves as RC2 in hardcoded-provisioning for backup
- Tide Shells as PIN2 in hardcoded-provisioning for backup

## 2026-05-14
STM32 v1.0.5.101  
ESP32 v1.0.3.99  
PCB Rev: D

Source: edaa5205-dirty

- dummy Version bump for charles to test OTA update with No Role

## 2026-05-14
STM32 v1.0.5.100  
ESP32 v1.0.3.98  
PCB Rev: D

Source: 959deff9-dirty

- adds OTA capability while NO ROLE

## 2026-05-13
STM32 v1.0.5.98  
ESP32 v1.0.3.96  
PCB Rev: D

Source: fd45d51f

- OTA Provisioning
- Ready for Charles to test before Sonar NYYC Member champs

## 2026-05-08
STM32 v1.0.5.91  
ESP32 v1.0.3.89  
PCB Rev: D

Source: c5686234-dirty

- Test OTA 

## 2026-05-08
STM32 v1.0.5.90  
ESP32 v1.0.3.88  
PCB Rev: D

Source: d04388db-dirty

- Update Provisioning data test coyote Point May,8

## 2026-05-07
STM32 v1.0.5.88  
ESP32 v1.0.3.86  
PCB Rev: D

Source: ca78e966-dirty

- update Provisioning data

## 2026-05-07
STM32 v1.0.5.87  
ESP32 v1.0.3.85  
PCB Rev: D

Source: bd6dafca-dirty

- include wake beacon as competitor

## 2026-05-07
STM32 v1.0.5.86  
ESP32 v1.0.3.84  
PCB Rev: D

Source: 115318de-dirty

- health check Sea Skippers

## 2026-05-07
STM32 v1.0.5.85  
ESP32 v1.0.3.83  
PCB Rev: D

Source: dbeb6731-dirty

- Change base to mast depths

## 2026-05-06
STM32 v1.0.5.84  
ESP32 v1.0.3.82  
PCB Rev: D

Source: ca0a7ae1-dirty

- SHowing Felix new FW OTA Flow

## 2026-05-06
STM32 v1.0.5.83  
ESP32 v1.0.3.81  
PCB Rev: D

Source: d5a594f3-dirty

- testing auto commit of Version bump

## 2026-05-06
STM32 v1.0.5.81  
ESP32 v1.0.3.79  
PCB Rev: D

Source: 47f4d1ae-dirty

- now auto archives old binaries

## 2026-05-06
STM32 v1.0.5.80  
ESP32 v1.0.3.78  
PCB Rev: D

Source: 20ac003d-dirty

- testing new prepare FW Flow V2

## 2026-05-06
STM32 v1.0.5.79  
ESP32 v1.0.3.77  
PCB Rev: D

Source: 20ac003d-dirty

- testing new PrepareFW Flow
