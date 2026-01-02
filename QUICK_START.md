# Quick Start Guide - Add Double Movement to Your Robot

Get Claw's smooth double movement in your VEX V5 project in 3 easy steps!

## Step 1: Copy Library Files (30 seconds)

Copy these two files into your project:

```
Your_Project/
├── include/
│   └── drive-control.h    ← Copy from Claw/include/
└── src/
    └── drive-control.cpp  ← Copy from Claw/src/
```

## Step 2: Create ClawDrive Instance (1 minute)

Add to your main.cpp or robot initialization:

```cpp
#include "drive-control.h"

// Your existing motors
motor LeftFront = motor(PORT1, ratio18_1, false);
motor LeftBack = motor(PORT2, ratio18_1, false);
motor RightFront = motor(PORT3, ratio18_1, true);
motor RightBack = motor(PORT4, ratio18_1, true);

// Create ClawDrive - that's it!
ClawDrive drive(LeftFront, LeftBack, RightFront, RightBack);
```

## Step 3: Use in Your Control Code (30 seconds)

Replace your existing drive code:

```cpp
void usercontrol(void) {
  while (true) {
    double forward = Controller1.Axis3.position(percent);
    double turn = Controller1.Axis1.position(percent);
    
    // One line for smooth double movement!
    drive.arcadeDrive(forward, turn);
    
    wait(20, msec);
  }
}
```

## You're Done! 🎉

Your robot now has smooth double movement capabilities!

### Test It

Run your robot and try:
- **Forward only**: Push left stick forward
- **Turn only**: Push right stick left/right
- **Diagonal**: Push both sticks simultaneously

You should see smooth diagonal arcs instead of jerky movements!

## Optional: Customize Settings

```cpp
// Adjust sensitivity (0.5 = slower, 1.5 = faster)
drive.setSensitivity(0.8);

// Adjust deadzone (prevents drift)
drive.setDeadzone(10.0);
```

## Need More Help?

- **LIBRARY_USAGE.md** - Complete integration guide
- **EXAMPLES.md** - Code examples for various scenarios
- **CONFIGURATION.md** - Configure for different robot types
- **README.md** - Full documentation

## Troubleshooting

### Robot turns when going straight?
One side is wired backward - toggle the `true`/`false` on motor creation:
```cpp
motor RightFront = motor(PORT3, ratio18_1, false);  // Changed to false
```

### Robot goes backward when pushing forward?
Reverse all motors - swap all `true`/`false` values.

### Movement too fast/slow?
Adjust sensitivity:
```cpp
drive.setSensitivity(0.7);  // 70% speed
```

### Controller drifting?
Increase deadzone:
```cpp
drive.setDeadzone(10.0);  // 10% deadzone
```

---

**That's it!** Enjoy smooth double movement in your VEX competitions! 🤖
