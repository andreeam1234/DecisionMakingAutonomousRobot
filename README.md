# Autonomous Mobile Robot with Local Decision-Making

## General Description

This project implements an **autonomous mobile robot** that navigates a physical environment and avoids obstacles using local sensing and onboard decision logic.

The robot operates as a **self-contained embedded system**:
* It senses its environment
* Makes decisions locally
* Controls actuators in real time
* Does not depend on external computation, networking, or cloud services

The goal of this project is not complexity, but **architectural correctness**:
* Choosing the right platform
* Defining a clear system boundary
* Placing intelligence in a single, well-defined location
* Solving one meaningful technical problem well

## System Architecture

### System Block Diagram
```
┌─────────────────────────────────────────────────────────┐
│                    ENVIRONMENT                          │
│                   (obstacles, walls)                    │
└────────────────────┬────────────────────────────────────┘
                     │ physical distance
                     ▼
              ┌──────────────┐
              │   HC-SR04    │
              │   Ultrasonic │
              │    Sensor    │
              └──────┬───────┘
                     │ distance reading
                     ▼
         ┌──────────────────────────┐
         │     ARDUINO UNO          │
         │  (Decision Logic / FSM)  │◄────── Battery Pack
         │                          │        (7.4V Li-ion)
         └────┬─────────────────┬───┘
              │                 │
              │ control         │ status
              │ signals         │ signals
              ▼                 ▼
       ┌────────────┐    ┌─────────────┐
       │   L298N    │    │ LEDs/Buzzer │
       │   Motor    │    │  (Feedback) │
       │   Driver   │    └─────────────┘
       └─────┬──────┘
             │ power
             ▼
      ┌─────────────┐
      │  DC Motors  │
      │ (L + R)     │
      └─────────────┘
```

### System Boundary

**Inside the system:**
* Arduino Uno microcontroller
* HC-SR04 ultrasonic sensor
* L298N motor driver
* 2× DC motors
* LEDs and buzzer
* Onboard power supply

**Outside the system:**
* Physical environment
* Obstacles
* Lighting conditions
* User

The robot is a **closed embedded system**. The boundary is clear and well-defined.

## Bill of Materials (BOM)

### Required Hardware

| Component | Specification | Purpose | Quantity |
|-----------|--------------|---------|----------|
| Microcontroller | Arduino Uno or Nano | Control logic & FSM | 1 |
| Motors | DC motors (6V) | Locomotion | 2 |
| Motor Driver | L298N | Motor control | 1 |
| Distance Sensor | HC-SR04 Ultrasonic | Obstacle detection | 1 |
| Chassis | 2-wheel robot chassis | Physical platform | 1 |
| Caster Wheel | Ball caster | Stability | 1 |
| Power Supply | 7.4V Li-ion or 6× AA (9V) | System power | 1 |

### Feedback & Debugging

| Component | Purpose | Quantity |
|-----------|---------|----------|
| LEDs | State indication | 3-4 |
| Buzzer | Event signaling | 1 |
| Resistors | LED current limiting (220Ω) | 3-4 |
| Jumper wires | Connections | As needed |


## Tutorial Source

**No full tutorial is followed.**

This project uses:
* Basic ultrasonic sensor documentation
* Motor driver datasheets (L298N)
* General robotics concepts discussed in class
* Custom finite state machine design


## Questions

### Q1 – What is the system boundary?

**Inside the system:**
* Arduino microcontroller
* Ultrasonic sensor
* Motors and motor driver
* LEDs and buzzer
* Onboard power supply

**Outside the system:**
* Environment
* Obstacles
* Lighting conditions
* User

---

### Q2 – Where does intelligence live?

**All intelligence lives on the microcontroller.**

Specifically:
* In the control logic
* In the finite state machine
* In the decision rules that process sensor input and choose actions

There is **one brain**, one place where decisions are made.

---

### Q3 – What is the hardest technical problem?

The hardest problem is **behavioral stability**.

More precisely:
* Preventing oscillatory behavior
* Avoiding infinite left-right avoidance loops
* Handling corner cases in confined spaces
* Doing all of this without blocking delays

---

### Q4 – What is the minimum demo?

The minimum demo is **clear, testable, and reliable**:

1. Robot powers on
2. Moves forward autonomously
3. Detects an obstacle
4. Stops
5. Chooses an avoidance direction
6. Executes avoidance maneuver
7. Resumes forward motion

**Success criterion:** Robot navigates around a single obstacle without getting stuck.

---

### Q5 – Why is this not just a tutorial?

This project is **not a tutorial** because:
* There is no single reference implementation
* Behavior emerges from design decisions
* A finite state machine is explicitly designed
* Timing is handled without blocking delays
* Failure modes are considered and addressed
* Recovery strategies are implemented

---

### Q6 – Do you need an ESP32?

**No.**
---

## 7. Finite State Machine Design

### State Machine Diagram
```
                    ┌─────────────┐
                    │   STARTUP   │
                    └──────┬──────┘
                           │
                           ▼
                  ┌─────────────────┐
            ┌────►│ MOVE_FORWARD    │◄────┐
            │     └────────┬────────┘     │
            │              │               │
            │    [distance < 20cm]         │
            │              │               │
            │              ▼               │
            │     ┌─────────────────┐     │
            │     │ OBSTACLE_DETECTED│    │
            │     └────────┬────────┘     │
            │              │               │
            │     [evaluate direction]     │
            │              │               │
            │         ┌────┴────┐          │
            │         ▼         ▼          │
            │  ┌──────────┐ ┌──────────┐  │
            │  │AVOID_LEFT│ │AVOID_RIGHT│ │
            │  └─────┬────┘ └─────┬────┘  │
            │        │             │       │
            │   [timeout 500ms]    │       │
            │        │             │       │
            │        └──────┬──────┘       │
            │               │              │
            │      [clear path detected]   │
            │               │              │
            └───────────────┘              │
                                           │
            ┌──────────────────────────────┘
            │
            │ [repeated obstacles detected]
            │
            ▼
    ┌───────────────┐
    │   RECOVERY    │
    │ (back + 180°) │
    └───────┬───────┘
            │
            │ [timeout 800ms]
            │
            └────► MOVE_FORWARD
```

### State Definitions

| State | Behavior | Exit Condition |
|-------|----------|----------------|
| **MOVE_FORWARD** | Both motors forward at cruise speed | `distance < 20cm` |
| **OBSTACLE_DETECTED** | Stop motors, evaluate sensor reading | Decision made (< 50ms) |
| **AVOID_LEFT** | Left motor reverse, right motor forward | `millis() - stateStartTime > 500` |
| **AVOID_RIGHT** | Right motor reverse, left motor forward | `millis() - stateStartTime > 500` |
| **RECOVERY** | Both motors reverse, then 180° turn | `millis() - stateStartTime > 800` |


## 8. Timing and Control Strategy

**Control loop frequency:** 50Hz (20ms period)  
**Sensor reading:** Every 100ms (HC-SR04 limitation)  
**State timeouts:** Managed via `millis()`

### Timing Parameters
```cpp
// Distance thresholds (cm)
#define OBSTACLE_THRESHOLD  20  // Stop and avoid
#define CLEAR_THRESHOLD     30  // Resume forward motion

// State durations (ms)
#define TURN_DURATION       500  // Minimum rotation time
#define BACKUP_DURATION     300  // Reverse duration in recovery
#define SENSOR_INTERVAL     100  // Ultrasonic read interval
```

### Hysteresis for Stability

To prevent oscillation around threshold:
* **Start avoiding:** `distance < 20cm`
* **Resume forward:** `distance > 30cm`

This 10cm hysteresis prevents rapid state switching due to sensor noise.

---

## 9. Failure Modes & Recovery Strategies

### HC-SR04 Sensor Constraints

| Limitation | Value | Impact on Design |
|------------|-------|------------------|
| Beam width | ~15° | Cannot detect obstacles to the side |
| Minimum range | 2cm | Very close objects may be missed |
| Maximum reliable range | 4m | Not an issue for this application |
| Update rate | ~60ms | Limits control loop frequency |

---

### Identified Failure Scenarios

#### Scenario 1: Corner Trap
**Problem:** Robot detects wall, turns, immediately detects adjacent wall, turns back infinitely.

**Solution:**
* Minimum turn duration (500ms state timeout)
* Backing up before turning in recovery mode
* Obstacle counter triggers recovery after 3 failed avoidance attempts

#### Scenario 2: Threshold Oscillation
**Problem:** Distance reading fluctuates around 20cm threshold, causing rapid state switching.

**Solution:** Hysteresis thresholds
* Stop avoiding: 20cm
* Resume forward: 30cm
* Creates a "dead zone" that prevents oscillation

#### Scenario 3: Tight Space / No Escape
**Problem:** Multiple obstacles, no clear forward or side escape route.

**Solution:** Recovery state
* After N failed avoidance attempts → backup + 180° turn
* Provides escape from confined spaces
* Resets obstacle counter after successful recovery

---

## 10. Demo Levels

| Demo Level | Description | Success Criterion |
|------------|-------------|-------------------|
| **Minimum** | Forward → obstacle → avoid → continue | Single obstacle navigation without getting stuck |
| **Extended** | Deadlock recovery + direction memory | Navigate 3+ obstacles with recovery behavior |
| **Bonus** | State visualization via LEDs | LED patterns indicate current FSM state |

### Minimum Demo 

- [ ] Robot powers on and enters MOVE_FORWARD state
- [ ] Detects obstacle at 20cm distance
- [ ] Transitions to OBSTACLE_DETECTED state
- [ ] Chooses avoidance direction (left or right)
- [ ] Executes turn for minimum 500ms
- [ ] Returns to MOVE_FORWARD when path is clear
- [ ] Process completes without manual intervention

