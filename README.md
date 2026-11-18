<div align="center">

# 🏭 Automated Multi-Station Sorting & Packing Line

### Simulated PLC control in Siemens TIA Portal V14 SP1

![TIA Portal](https://img.shields.io/badge/TIA%20Portal-V14%20SP1-009999?style=for-the-badge)
![PLC](https://img.shields.io/badge/PLC-S7--1511--1%20PN-0a7bc2?style=for-the-badge)
![HMI](https://img.shields.io/badge/HMI-KTP400F%20Mobile-5b6770?style=for-the-badge)
![Simulation](https://img.shields.io/badge/Mode-Simulation%20Only-orange?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

*Study project for the Computer Control course, Riga Technical University*

</div>

---

## 📖 Overview

A simulated sorting and packing line controlled by a Siemens **S7-1511-1 PN** CPU, with a **KTP400F Mobile** operator panel. The line weighs each package, rejects overweight items, counts accepted, rejected and remaining batch items, ramps the conveyor speed with the load, logs sensor faults, and shows everything on the panel.

> ⚠️ **Simulation only.** No physical hardware is used. Results apply to the simulated CPU and panel.

---

## 🎬 Video Demo

<img width="1331" height="746" alt="image" src="https://github.com/user-attachments/assets/954bb25a-612d-421a-a300-788f86c5e22c" />

A six-minute walkthrough of the program, a live run with an overweight reject, the emergency stop, and the operator panel. **[▶ Watch on Google Drive][(VIDEO_LINK](https://drive.google.com/file/d/1WRO-6jpwKwBkxr3EvZm6gZTQb519PXd8/view))**

---

## 🧰 Software

| Component | Version |
|---|---|
| 🛠️ Engineering | TIA Portal V14 SP1 (STEP 7 Professional V14 SP1) |
| 🧪 PLC simulator | S7-PLCSIM V14 SP1 (14.00.01.00) |
| 🖥️ HMI runtime | WinCC Runtime Advanced V14 SP1 (14.00.01.00) |
| 🖥️ HMI simulator | WinCC Runtime Advanced Simulator 14.0.1.0 |
| 💻 PC | Windows 7 Professional SP1 |

---

## ⚙️ What It Does

1. ⚖️ **Weighs** each package from a 4–20 mA load-cell signal
2. 🚫 **Rejects** packages above **25 kg** through the diverter
3. 🔢 **Counts** accepted, rejected and remaining batch items
4. 🚚 **Ramps** the conveyor up over 5 s and down over 2 s, reducing speed under load
5. 🚨 **Logs** wire-break and speed-feedback faults
6. 📊 **Displays** weight, speed, counts, reject rate and throughput on the panel

### Key formulas

**Weight:** `W = (raw − 5530) / 22118 × 50 kg`

**Speed command:** `setpoint (27648) × load factor × ramp fraction`, where `load factor = clamp(1 − 0.5 × W / 25, 0.5, 1)`

At **16000** raw counts the weight is **23.67 kg** and the command settles at **14560** counts.

---

## 🔌 Hardware Configuration (Simulated)

| Slot | Module | Use |
|:---:|---|---|
| CPU | CPU 1511-1 PN | Controller, `192.168.1.2` |
| 2 | DI 16x24VDC HF | Digital inputs. Channel 0 (E-stop) triggers OB 40 on falling edge |
| 3 | DQ 16x24VDC/0.5A ST | Motor, diverter and beacon outputs |
| 4 | AI 4xU/I/RTD/TC ST | Load cell (4–20 mA, ch. 0), speed feedback (0–10 V, ch. 1) |
| 5 | AQ 2xU/I ST | VFD speed command (ch. 0) |
| HMI | KTP400F Mobile | Operator panel, `192.168.1.3` |

<details>
<summary><b>🔤 Input & Output Signals</b></summary>

| Signal | Address | Notes |
|---|---|---|
| `E_Stop` | `%I0.0` | Normally closed wiring. `1` = healthy |
| `Mode_Auto` | `%I0.1` | `1` = automatic, `0` = manual |
| `Start` | `%I0.2` | Normally open. Held by the seal-in |
| `Stop` | `%I0.3` | Normally closed contact in logic. `0` = healthy |
| `Reset` | `%I0.4` | Combined with screen reset into `Reset_Req` |
| `IR_Prox` | `%I0.5` | Each rising edge counts one product |
| `Load_Cell` | `%IW64` | Raw count. `5530` = 4 mA, `27648` = 20 mA |
| `Speed_Feedback` | `%IW66` | Monitored by diagnostics |
| `Conveyor_Motor` | `%Q0.0` | Motor run command |
| `Diverter_Solenoid` | `%Q0.1` | `1` = reject |
| `Alarm_Beacon` | `%Q0.2` | Alarm or E-stop latch |
| `VFD_Speed_Cmd` | `%QW80` | `27648` = full speed |

</details>

---

## 🧩 Program Structure

| Block | Language | Role |
|---|:---:|---|
| 🟦 **OB 1** | LAD | Empty by design. Control runs on a fixed time base in OB 30 |
| 🟦 **OB 30** | LAD | Cyclic 100 ms. Scaling, run logic, latches, FB/FC calls, panel bridge |
| 🟦 **OB 35** | SCL | Cyclic 5 s. Wire-break and speed checks, run-time accumulation |
| 🟦 **OB 40** | SCL | Hardware interrupt on E-stop. Motor stop and latch |
| 🟩 **FB_ConveyorController** | SCL | TON ramp-up, TOF stop grace, XOR mode change, ROL state ring, MUL load factor |
| 🟩 **FB_SorterLogic** | SCL | GT overweight, EQ sensor fault, AND/OR logic, NOT AND interlock, CTU_INT/CTD_INT counters, ADD/SUB totals, alarm latch |
| 🟩 **FB_ProductSimulator** | SCL | Generates product presence and load when `Sim_Enable` is TRUE |
| 🟨 **FC_ScaleAnalogInput** | SCL | Load-cell count → kilograms |
| 🟨 **FC_CalculateStatistics** | SCL | Reject rate (%) and throughput (items/min), with status return |
| 🗂️ **DB_Configuration** | Data | Thresholds, times, batch size, simulator settings |
| 🗂️ **DB_Runtime** | Data | Weight, latch, alarm log, run time, panel source values |

---

## 🖥️ Operator Panel

The panel reads only PLC tags written by six copy networks at the end of OB 30. Its **Start**, **Stop** and **Reset** buttons set memory bits while pressed. Speed is shown as a percentage of full scale, since the project has no measured belt speed.

---

## ▶️ Running the Simulation

1. Start **S7-PLCSIM** with PLC_1 at `192.168.1.2` in **RUN**
2. Download the program from TIA Portal
3. Open the watch table and click **Monitor all**
4. Set `Sim_Enable` to **TRUE** in `DB_Configuration` for automatic products, or **FALSE** to drive `IR_Prox` and `Load_Cell` by hand
5. Start the WinCC simulator for HMI_1 to view the dashboard

---

## ✅ Test Results

| Test | Outcome |
|---|:---:|
| Seal-in holds the motor after Start is released | ✅ Passed |
| Speed ramp settles at 14560 counts at 23.67 kg | ✅ Passed |
| Stop grace lowers speed, then the motor stops | ✅ Passed |
| E-stop drops the motor and sets the latch | ✅ Passed (latch set by scan network) |
| Latch clears only on Reset with E-stop healthy | ✅ Passed |
| Screen Stop breaks the run rung | ✅ Passed |
| Overweight product rejected, alarm latched | ✅ Passed |
| Accepted product increments accepted count | ✅ Passed |
| Batch countdown and reload | ✅ Passed |
| Wire-break log entry | ✅ Passed |
| Reject rate and throughput on panel | ✅ Passed |
| OB 40 hardware interrupt fires on E-stop | ⚠️ Not verified |

> **Interrupt note:** DI channel 0 is configured with a falling-edge trigger linked to OB 40. The interrupt counter stayed at zero in PLCSIM after a watch-table change, so the simulator did not deliver the module event. The latch is also set from a scan-based network in OB 30, so the safety behaviour does not depend on the interrupt.

---

## ⚠️ Known Limitations

- Simulation only. Physical wiring is documented in tag comments, not verified on hardware
- Hardware interrupt not verified in PLCSIM (see note above)
- Overweight alarm trips on every heavy product, not on a rate window
- Alarm log entries have no timestamps
- Second analogue output reserved and unused; buzzer not implemented

---

## 📁 Repository Contents

| File | Description |
|---|---|
| `README.md` | This document |
| `LICENSE` | MIT License |
| `Sorting_Line.zap14` | TIA Portal V14 SP1 project archive. Restore it with **Project → Retrieve** |

The video is hosted on Google Drive (linked above) to keep the repository small.

---

## 🤖 AI Usage Statement

An AI assistant (Claude Haiku 5.5) was used during this project as a support tool. It was consulted for planning the program structure, suggesting code for the function blocks and networks, interpreting compiler messages, and structuring parts of this report. The assistant had no access to the engineering software, the PLC, or the simulator; it worked only from the text and screenshots provided by me.

Not all suggestions were used as given. Many were found to be incorrect or inapplicable in TIA Portal and were corrected through compiling and testing in the simulator. No code was accepted into the final program unless it was examined by me, compiled and produced the expected behavior.

All programming, hardware configuration, testing, screenshots, and measured results were carried out by me myself. To conclude, I wrote the entire report by hands, and the aforesaid AI assistant was only used to correct my grammar mistakes and give corrections on the report structure.

---

## 📜 License

Released under the **MIT License**. See [LICENSE](LICENSE) for details.

**Copyright © 2026 Rostyslav Hrabenko**

---

<div align="center">

*Built with patience, a lot of compiles, and curiosity about the unknown.* 🔭

</div>
