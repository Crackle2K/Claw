# Configuration Guide for Different Robot Types

This guide helps you configure the Claw Drive Control Library for different VEX robot drivetrain configurations.

## Quick Configuration Selector

Choose your robot type:

1. **Standard 4-Motor Tank Drive** (Most common) ✅ Fully Supported
2. **6-Motor Tank Drive** (High power) ⚠️ See notes below
3. **Mecanum Wheel Drive** (Omnidirectional) ✅ Fully Supported
4. **X-Drive** (Advanced omnidirectional) ✅ Fully Supported
5. **H-Drive** (Strafing capable) ⚠️ Requires customization

---

## 1. Standard 4-Motor Tank Drive (Default)

**What it is**: Two motors on each side, standard wheels

**Status**: ✅ Fully supported by ClawDrive

### Setup

```cpp
#include "drive-control.h"

// Define your motors (adjust ports as needed)
motor LeftFront = motor(PORT1, ratio18_1, false);
motor LeftBack = motor(PORT2, ratio18_1, false);
motor RightFront = motor(PORT3, ratio18_1, true);   // Reversed
motor RightBack = motor(PORT4, ratio18_1, true);    // Reversed

// Create ClawDrive instance
ClawDrive drive(LeftFront, LeftBack, RightFront, RightBack);
```

### Usage

```cpp
void usercontrol(void) {
  while (true) {
    double forward = Controller1.Axis3.position(percent);
    double turn = Controller1.Axis1.position(percent);
    
    // Smooth double movement!
    drive.arcadeDrive(forward, turn);
    
    wait(20, msec);
  }
}
```

**Diagonal Movement**: Works perfectly with arcade drive

---

## 2. Six-Motor Tank Drive

**What it is**: Three motors on each side for more power

**When to use**: Heavy robots, need more torque

**Status**: ⚠️ Requires workaround (ClawDrive expects 4 individual motors)

### Option A: Use 4 Motors with ClawDrive

Use the two front and two back motors (or any 4 motors):

```cpp
motor LeftFront = motor(PORT1, ratio18_1, false);
motor LeftMiddle = motor(PORT2, ratio18_1, false);
motor LeftBack = motor(PORT3, ratio18_1, false);
motor RightFront = motor(PORT4, ratio18_1, true);
motor RightMiddle = motor(PORT5, ratio18_1, true);
motor RightBack = motor(PORT6, ratio18_1, true);

// Create ClawDrive with 4 of the motors
ClawDrive drive(LeftFront, LeftBack, RightFront, RightBack);

// Separately control the middle motors to follow
// In usercontrol or autonomous:
void syncMiddleMotors() {
  LeftMiddle.spin(fwd, LeftFront.velocity(pct), pct);
  RightMiddle.spin(fwd, RightFront.velocity(pct), pct);
}
```

### Option B: Create Motor Groups and Extend ClawDrive

For full 6-motor support, you would need to modify the ClawDrive class to accept motor_group references. This is an advanced customization.

**Diagonal Movement**: Works with Option A

---

## 3. Mecanum Wheel Drive

**What it is**: Four wheels with diagonal rollers, can move in any direction

**When to use**: Need true omnidirectional movement

**Status**: ✅ Fully supported with fieldCentricDrive

### Setup

```cpp
#include "drive-control.h"

// Mecanum motors - verify roller orientation!
motor LeftFront = motor(PORT1, ratio18_1, false);
motor LeftBack = motor(PORT2, ratio18_1, false);
motor RightFront = motor(PORT3, ratio18_1, true);
motor RightBack = motor(PORT4, ratio18_1, true);

// Create ClawDrive instance
ClawDrive drive(LeftFront, LeftBack, RightFront, RightBack);
```

### Usage

```cpp
void usercontrol(void) {
  while (true) {
    // Get all three control axes
    double forward = Controller1.Axis3.position(percent);  // Left Y
    double strafe = Controller1.Axis4.position(percent);   // Left X
    double turn = Controller1.Axis1.position(percent);     // Right X
    
    // Use field-centric drive for omnidirectional control
    drive.fieldCentricDrive(forward, strafe, turn);
    
    wait(20, msec);
  }
}
```

**Important**: Verify wheel orientation - rollers should form an "X" pattern when viewed from above

**Diagonal Movement**: Best diagonal movement capability!

### Mecanum Control Tips
- Forward + Strafe = True diagonal (no rotation)
- Forward + Turn = Curved diagonal  
- All three = Complex movement paths

---

## 4. X-Drive Configuration

**What it is**: Four wheels at 45° angles with omni wheels

**When to use**: Fast omnidirectional movement, good for offense

**Status**: ✅ Supported with fieldCentricDrive (may need motor reversal adjustments)

### Setup

```cpp
#include "drive-control.h"

// X-Drive motors are at 45° angles
// Adjust reversed flags based on your wiring
motor LeftFront = motor(PORT1, ratio18_1, false);
motor LeftBack = motor(PORT2, ratio18_1, true);    // May need reversal
motor RightFront = motor(PORT3, ratio18_1, true);  // May need reversal  
motor RightBack = motor(PORT4, ratio18_1, false);

// Create ClawDrive instance
ClawDrive drive(LeftFront, LeftBack, RightFront, RightBack);
```

### Usage

```cpp
void usercontrol(void) {
  while (true) {
    double forward = Controller1.Axis3.position(percent);
    double strafe = Controller1.Axis4.position(percent);
    double turn = Controller1.Axis1.position(percent);
    
    // Use field-centric drive (handles X-Drive kinematics)
    drive.fieldCentricDrive(forward, strafe, turn);
    
    wait(20, msec);
  }
}
```

**Note**: Test and adjust motor directions based on actual movement

**Diagonal Movement**: Excellent, faster than mecanum

---

## 5. H-Drive Configuration

**What it is**: Tank drive + center strafe wheel(s)

**When to use**: Want strafing but keep tank drive simplicity

**Status**: ⚠️ Requires customization (ClawDrive doesn't support strafe motor)

### Workaround

Use ClawDrive for tank movement, control strafe separately:

```cpp
#include "drive-control.h"

// Tank drive motors
motor LeftFront = motor(PORT1, ratio18_1, false);
motor LeftBack = motor(PORT2, ratio18_1, false);
motor RightFront = motor(PORT3, ratio18_1, true);
motor RightBack = motor(PORT4, ratio18_1, true);

// Strafe motor (separate)
motor StrafeMotor = motor(PORT5, ratio18_1, false);

// Create ClawDrive for tank movement
ClawDrive drive(LeftFront, LeftBack, RightFront, RightBack);

void usercontrol(void) {
  while (true) {
    double forward = Controller1.Axis3.position(percent);
    double turn = Controller1.Axis1.position(percent);
    double strafe = Controller1.Axis4.position(percent);
    
    // Use ClawDrive for forward/turn
    drive.arcadeDrive(forward, turn);
    
    // Control strafe separately
    StrafeMotor.spin(fwd, strafe, pct);
    
    wait(20, msec);
  }
}
```

**Diagonal Movement**: Good, combines tank curves with strafing

---

## ClawDrive Configuration Settings

### Deadzone

Controls how much controller drift is ignored:

```cpp
// Set in constructor
ClawDrive drive(LeftFront, LeftBack, RightFront, RightBack, 
                10.0);  // 10% deadzone

// Or set later
drive.setDeadzone(10.0);
```

- **Default**: 5.0 (5%)
- **Range**: 0-20 recommended
- **Higher values**: Less sensitive to small movements
- **Lower values**: More responsive but may drift

### Sensitivity

Controls overall drive speed/power:

```cpp
// Set in constructor
ClawDrive drive(LeftFront, LeftBack, RightFront, RightBack,
                5.0,   // deadzone
                0.8);  // 80% sensitivity

// Or set later  
drive.setSensitivity(0.8);
```

- **Default**: 1.0 (100%)
- **Range**: 0.1-2.0 recommended
- **Lower values** (0.5): Slower, more precise control
- **Higher values** (1.5): Faster, more aggressive driving

### Getting Current Settings

```cpp
double currentDeadzone = drive.getDeadzone();
double currentSensitivity = drive.getSensitivity();
```

---

## Motor Port Reference

### Common VEX V5 Port Layouts

**Standard 4-Motor**:
```
LEFT SIDE:        RIGHT SIDE:
PORT1 - LF        PORT3 - RF
PORT2 - LB        PORT4 - RB
```

**6-Motor Drive**:
```
LEFT SIDE:        RIGHT SIDE:
PORT1 - LF        PORT4 - RF
PORT2 - LM        PORT5 - RM
PORT3 - LB        PORT6 - RB
```

**Mecanum/X-Drive**:
```
FRONT:
PORT1 - LF /   \ RF - PORT3

BACK:
PORT2 - LB \   / RB - PORT4
```

---

## Gear Ratio Selection

Choose gear ratio based on your needs:

| Ratio | Speed | Torque | Best For |
|-------|-------|--------|----------|
| 36:1 | Slow | High | Heavy robots, climbing |
| 18:1 | Medium | Medium | **General use (default)** |
| 6:1 | Fast | Low | Lightweight, speed-focused |

Update when creating motors:

```cpp
motor LeftFront = motor(PORT1, ratio18_1, false);  // Change ratio here
//                            ^^^
//                   ratio36_1, ratio18_1, or ratio6_1
```

---

## Troubleshooting

### Problem: Robot turns when going "straight"

**Solution**: One side is wired backward

```cpp
// Try reversing one side
motor RightFront = motor(PORT3, ratio18_1, true);  // Toggle this
motor RightBack = motor(PORT4, ratio18_1, true);   // Toggle this
```

### Problem: Robot goes backward when joystick pushed forward

**Solution**: Reverse all motors

```cpp
motor LeftFront = motor(PORT1, ratio18_1, true);   // Add 'true'
motor LeftBack = motor(PORT2, ratio18_1, true);
motor RightFront = motor(PORT3, ratio18_1, false); // Remove 'true'
motor RightBack = motor(PORT4, ratio18_1, false);
```

### Problem: Mecanum wheels don't strafe correctly

**Solution**: Check wheel orientation - rollers should form X pattern

```
Top view:
  / \    Correct X pattern
  \ /
```

### Problem: Diagonal movement too sensitive

**Solution**: Reduce sensitivity

```cpp
drive.setSensitivity(0.7);  // Reduce from 1.0
```

### Problem: Controller drift affecting diagonal movement

**Solution**: Increase deadzone

```cpp
drive.setDeadzone(10.0);  // Increase from 5.0
```

---

## Testing Your Configuration

Run this test in autonomous to verify correct setup:

```cpp
void testConfiguration(void) {
  Brain.Screen.print("Testing configuration...");
  
  // Test 1: Forward
  drive.arcadeDrive(50, 0);
  wait(1, seconds);
  drive.stop();
  wait(0.5, seconds);
  // Should move straight forward
  
  // Test 2: Turn right
  drive.arcadeDrive(0, 50);
  wait(1, seconds);
  drive.stop();
  wait(0.5, seconds);
  // Should rotate clockwise
  
  // Test 3: Diagonal forward-right
  drive.arcadeDrive(50, 50);
  wait(1, seconds);
  drive.stop();
  // Should curve forward-right smoothly
  
  Brain.Screen.print("Test complete!");
}
```

If any test fails, check your motor directions and ports!

---

## Summary Table

| Drive Type | Motors | ClawDrive Support | Strafe? | Complexity |
|------------|--------|-------------------|---------|------------|
| 4-Motor Tank | 4 | ✅ Full | No | Low |
| 6-Motor Tank | 6 | ⚠️ Partial | No | Low |
| Mecanum | 4 | ✅ Full | Yes | Medium |
| X-Drive | 4 | ✅ Full | Yes | Medium |
| H-Drive | 5-6 | ⚠️ Partial | Yes | Medium |

**Recommendation**: Start with standard 4-motor tank drive and arcade control using ClawDrive. It provides excellent diagonal movement with the simplest configuration!
