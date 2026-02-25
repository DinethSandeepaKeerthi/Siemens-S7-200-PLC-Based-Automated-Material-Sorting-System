# 🏭 Siemens-S7-200-PLC-Based-Automated-Material-Sorting-System — Siemens S7-200 PLC

![PLC](https://img.shields.io/badge/PLC-Siemens%20S7--200%20CPU%20224-blue)
![Software](https://img.shields.io/badge/Software-STEP%207%20Micro%2FWIN%20v4.0.9.25-orange)
![Language](https://img.shields.io/badge/Language-Ladder%20Logic-green)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

> A conveyor-based automated material sorting system that detects and separates **metal** and **non-metal** objects using proximity sensors, a pneumatic cylinder, and a **Siemens S7-200 CPU 224** PLC programmed in Ladder Logic using STEP 7 Micro/WIN.

📹 **[Watch Demo Video on LinkedIn](#)** ← 

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [System Architecture](#-system-architecture)
- [Components List with Specifications](#-components-list-with-specifications)
- [I/O Address Mapping](#-io-address-mapping)
- [Ladder Logic Explanation](#-ladder-logic-explanation)
- [Wiring Diagram](#-wiring-diagram)
- [How to Run / Test](#-how-to-run--test)
- [Demo Object Trick](#-demo-object-trick)
- [Author](#-author)

---

## 📌 Project Overview

This project implements an **automated conveyor belt sorting system** that classifies objects as **metal** or **non-metal** and sorts them accordingly — without any manual intervention.

The system was developed using:
- **PLC:** Siemens S7-200 CPU 224 (Firmware REL 02.01)
- **Programming Software:** STEP 7 Micro/WIN Version 4.0.9.25
- **Program File:** `M_AND_NON_M_SORTING.mwp`
- **Main POU:** MAIN program block

### How It Works

The conveyor belt has three sensor stations labeled **A**, **B**, and **C**:

| Station | Sensor | Symbol | PLC Address | Action |
|---------|--------|--------|-------------|--------|
| A | Object Detection Sensor | D1 | I0.2 | Detects any object → starts conveyor |
| B | Metal Detection Sensor | D2 | I0.1 | Detects metal → fires pneumatic cylinder |
| C | Non-Metal Detection Sensor | — | — | Non-metal passes here → conveyor stops |
| — | Emergency Stop Button | D3 | I0.3 | Halts all operations immediately |

**Sorting Flow:**

1. Object placed at **Station A** → D1 (I0.2) detects it → conveyor motor starts (CONVAYER/Q0.2 ON) → green lamp ON (GREEN/Q0.0)
2. Object reaches **Station B** → metal sensor D2 (I0.1) checks it:
   - **METAL** → pneumatic cylinder fires (ACTUATOR/Q0.3) + red lamp ON (RED/Q0.1) → object is pushed off the belt. A **TON timer** controls the cylinder hold time.
   - **NON-METAL** → object passes through Station B untouched
3. Non-metal object reaches **Station C** → detected → conveyor stops
4. Pressing **Emergency Stop D3 (I0.3)** halts everything immediately at any point

---

## 🏗 System Architecture

```
 ┌──────────────────────────────────────────────────────────┐
 │                     CONVEYOR BELT                        │
 │                                                          │
 │  [A] ─────────────── [B] ─────────────── [C]            │
 │  D1 / I0.2          D2 / I0.1          Non-Metal        │
 │  Object Sensor      Metal Sensor         Sensor          │
 │                          │                               │
 │                   [Pneumatic Cylinder]                   │
 │                   ACTUATOR / Q0.3                        │
 │                   (ejects metal objects sideways)        │
 └──────────────────────────────────────────────────────────┘
                            │
                   ┌────────────────┐
                   │ S7-200 CPU 224 │
                   │  REL 02.01     │
                   └────────────────┘
            ┌───────────┼─────────────┐
       GREEN/Q0.0   RED/Q0.1     D3/I0.3
      (Conveyor ON) (Cylinder)  (E-Stop NC)
```

---

## 🔩 Components List with Specifications

### PLC Unit

| Component | Specification |
|-----------|--------------|
| **PLC Model** | Siemens S7-200 CPU 224 |
| **Firmware Version** | REL 02.01 |
| **Power Supply** | 24V DC |
| **Digital Inputs** | 14 × DI (24V DC) |
| **Digital Outputs** | 10 × DO (Relay / Transistor) |
| **Programming Software** | STEP 7 Micro/WIN v4.0.9.25 |
| **Communication Port** | RS-485 / PPI |
| **Project File** | M_AND_NON_M_SORTING.mwp |

### Sensors

| Sensor | Symbol | Type | Detection Range | Output Type |
|--------|--------|------|----------------|------------|
| Object Detection Sensor — Station A | D1 | Capacitive / Inductive Proximity | 0–10 mm | PNP, 24V DC |
| Metal Detection Sensor — Station B | D2 | Inductive Proximity Sensor | 0–8 mm | PNP, 24V DC |
| Non-Metal Detection Sensor — Station C | — | Capacitive Proximity Sensor | 0–10 mm | PNP, 24V DC |

### Actuators & Control Devices

| Component | Symbol | Specification |
|-----------|--------|--------------|
| Conveyor Belt Motor | CONVAYER | 24V DC Motor controlled via relay output Q0.2 |
| Pneumatic Cylinder | ACTUATOR | Double-acting cylinder with 5/2 solenoid valve |
| Solenoid Valve | — | 24V DC coil, 5/2 directional control valve |
| Green Indicator Lamp | GREEN | 24V DC panel mount LED — conveyor running |
| Red Indicator Lamp | RED | 24V DC panel mount LED — cylinder active |
| Emergency Stop Button | D3 | NC (Normally Closed) push button, 24V DC |
| On-Delay Timer | TON | Built-in S7-200 timer — controls cylinder hold duration |

---

## 📡 I/O Address Mapping

> ✅ All addresses below are read directly from the symbol table inside `M_AND_NON_M_SORTING.mwp`

### Digital Inputs (I) — Real Addresses from File

| PLC Address | Symbol | Component | Logic Type | Description |
|-------------|--------|-----------|------------|-------------|
| **I0.1** | D2 | Metal Detection Sensor (Station B) | NO — Normally Open | HIGH when metal object detected |
| **I0.2** | D1 | Object Detection Sensor (Station A) | NO — Normally Open | HIGH when any object detected |
| **I0.3** | D3 | Emergency Stop Button | NC — Normally Closed | LOW when E-Stop is pressed |

### Digital Outputs (Q) — Real Addresses from File

| PLC Address | Symbol | Component | Description |
|-------------|--------|-----------|-------------|
| **Q0.0** | GREEN | Green Indicator Lamp | ON when conveyor belt is running |
| **Q0.1** | RED | Red Indicator Lamp | ON when pneumatic cylinder is active |
| **Q0.2** | CONVAYER | Conveyor Belt Motor Relay | Starts and stops the conveyor belt |
| **Q0.3** | ACTUATOR | Pneumatic Solenoid Valve | Fires cylinder to eject metal objects |

### Timer (Found in Program)

| Identifier | Type | Function |
|------------|------|----------|
| TON (Txx) | On-Delay Timer | Controls the duration the pneumatic cylinder stays extended |

---

## 🧠 Ladder Logic Explanation

The Ladder Logic is written in the **MAIN** block. Below are the key logic networks:

### Network 1 — Conveyor Start (Object at Station A)

```
|  I0.2        I0.3       Q0.2  |      Q0.2       Q0.0    |
|──[ ]─────────[/]────────[ ]───|────────( )────────( )───|
|  D1          D3        Latch  |      CONVAYER    GREEN   |
| Object     E-Stop(NC)         |
```

When object sensor D1 (I0.2) activates AND Emergency Stop is NOT pressed → Conveyor motor (Q0.2) starts and self-latches. Green lamp (Q0.0) illuminates simultaneously.

### Network 2 — Metal Detection & Cylinder Actuation (Station B)

```
|  I0.1      Q0.2    |     Q0.3        Q0.1    |
|──[ ]────────[ ]────|──────( )──────────( )───|
|  D2      CONVAYER  |  ACTUATOR        RED     |
| Metal     Running  |  Pneumatic    Red Lamp   |
```

When metal sensor D2 (I0.1) detects AND conveyor is running → Pneumatic cylinder (Q0.3) fires and red lamp (Q0.1) turns ON.

### Network 3 — TON Timer (Cylinder Hold Time)

```
|  Q0.3     |       TON         |
|───[ ]─────|──[TON  T37        |──
| ACTUATOR  |   PT:  xxx ms ]───|
```

While cylinder is active, the TON timer controls how long the cylinder stays extended before the output resets.

### Network 4 — Non-Metal Detected at Station C / Conveyor Stop

```
|  Non-Metal Sensor (Station C)  |   Q0.2 (RESET)   |
|────────────[ ]─────────────────|──────(R)──────────|
```

When the non-metal sensor at Station C detects the object → Conveyor motor (Q0.2) is RESET and belt stops.

### Network 5 — Emergency Stop

```
|   I0.3    |   Q0.2(R)      Q0.3(R)    |
|────[/]────|─────(R)───────────(R)──────|
|  D3(NC)   |  CONVAYER     ACTUATOR     |
```

When Emergency Stop D3 (I0.3) is pressed → Conveyor and cylinder outputs are immediately RESET.


---

## 🔌 Wiring Diagram

```
  24V DC Power Supply (+24V and 0V)
          │
  ────────┴──────────────────────────────────────────────
  │                   S7-200 CPU 224                    │
  │   INPUT TERMINALS              OUTPUT TERMINALS      │
  │                                                      │
  │   1M  ──── 0V (Common Ground)                       │
  │                                                      │
  │   I0.1 ──── D2 Metal Sensor (Black wire)            │
  │   I0.2 ──── D1 Object Sensor (Black wire)           │
  │   I0.3 ──── D3 Emergency Stop (NC terminal)         │
  │                                                      │
  │   1L+ ──── +24V                                     │
  │   Q0.0 ──── GREEN Indicator Lamp (+)                │
  │   Q0.1 ──── RED Indicator Lamp (+)                  │
  │   Q0.2 ──── CONVAYER Motor Relay coil (+)           │
  │   Q0.3 ──── ACTUATOR Solenoid Valve coil (+)        │
  └──────────────────────────────────────────────────────┘

Sensor Wiring (PNP 3-Wire):
  Brown  ──► +24V (Power)
  Blue   ──► 0V / 1M (Ground)
  Black  ──► PLC Input Terminal (Signal)

Emergency Stop Button (NC Contact):
  One terminal ──► +24V
  Other terminal ──► I0.3
  When NOT pressed: I0.3 = HIGH (circuit closed)
  When pressed: I0.3 = LOW (circuit opens → system stops)
```

---

## ▶ How to Run / Test

### Prerequisites

- Siemens **STEP 7 Micro/WIN v4.0** or later on PC
- **PPI Cable** or USB-PPI adapter (COM port or USB)
- **24V DC power supply** wired to PLC and all devices
- Sensors positioned correctly at Stations A, B, C

### Upload Steps

**1. Open Program**
```
Open STEP 7 Micro/WIN
→ File → Open → M_AND_NON_M_SORTING.mwp
```

**2. Connect to PLC**
```
→ PLC → Communications
→ Set PPI baud rate: 9.6 kbps
→ Select correct COM port
→ Click "Refresh" to confirm PLC is found
```

**3. Download to PLC**
```
→ Switch PLC to STOP mode first
→ PLC → Download (F8)
→ Switch PLC to RUN mode
→ Confirm CPU green LED is solid
```

### Test Scenarios

**Test 1 — Metal Object**

| Step | Expected I/O | Indicator |
|------|-------------|-----------|
| Place metal object at Station A | I0.2 = ON | — |
| Conveyor starts | Q0.2 = ON | 🟢 Green ON |
| Object reaches Station B | I0.1 = ON | — |
| Cylinder fires & ejects object | Q0.3 = ON | 🔴 Red ON |
| Cylinder retracts after timer | Q0.3 = OFF | 🔴 Red OFF |

**Test 2 — Non-Metal Object**

| Step | Expected I/O | Indicator |
|------|-------------|-----------|
| Place non-metal object at Station A | I0.2 = ON | — |
| Conveyor starts | Q0.2 = ON | 🟢 Green ON |
| Object passes Station B | I0.1 = OFF | — |
| Object reaches Station C | — | — |
| Conveyor stops | Q0.2 = OFF | 🟢 Green OFF |

**Test 3 — Emergency Stop**

| Step | Expected Result |
|------|----------------|
| Press E-Stop during any operation | I0.3 goes LOW |
| All outputs immediately OFF | Q0.0, Q0.1, Q0.2, Q0.3 = OFF |
| System holds safe state | No restart until manually initiated |

### Troubleshooting

| Problem | Possible Cause | Fix |
|---------|---------------|-----|
| Conveyor not starting | Sensor not detecting or E-Stop active | Check I0.2 signal and I0.3 = HIGH |
| Cylinder not firing | Metal sensor misaligned / Q0.3 not toggling | Verify I0.1 in PLC status monitor |
| Green LED always OFF | Q0.0 wiring or PLC not in RUN | Check Q0.0 output and PLC mode |
| No PC–PLC communication | Wrong COM port or baud rate | Set to 9.6 kbps PPI in Micro/WIN |
| Download fails | PLC still in RUN mode | Switch to STOP mode first |

---

## 💡 Demo Object Trick

For the project demonstration, a **single dual-sided test object** was used instead of two separate test pieces:

```
         ┌──────────────────────────────┐
         │      NON-METAL FACE         │  ← Top: plastic / non-conductive
         │  Passes through Station B   │
         ├──────────────────────────────┤
         │        METAL FACE           │  ← Bottom: metal plate
         │  Triggers D2 at Station B   │
         └──────────────────────────────┘
```

**Flip the object before placing on conveyor:**
- **Metal side facing sensor** → triggers I0.1 → cylinder ejects → Red LED ON
- **Non-metal side facing sensor** → passes B → reaches C → conveyor stops → Green LED OFF

This technique made live demonstrations **fast, clean, and repeatable** — showcasing both sorting scenarios with a single test object. 🎯

---

📹 **Full demo video:** [View on LinkedIn](#) ← 

---

## 👤 Author

**[DINETH SANDEEPA KEERTHI]**

- 🔗 LinkedIn: [https://bit.ly/DSK-linkedin](#)
- 💻 GitHub: [https://bit.ly/DinethSK-Github](#)
- 📧 Email: dinethsandeepa425@gmail.com

---
