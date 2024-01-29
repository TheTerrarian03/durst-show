# -- Kachow 2024 Robot Outline --

Outline and layout of subsystems, devices, commands, etc. for this year's code and robot.

## Table of Contents
- [Subsystems](#subsystems)
    - [Chassis And Navigation](#chassis-and-navigation)
        - [Swerve Drivetrain](#swerve-drivetrain)
        - [Autonomous](#autonomous)
        - [Vision](#vision)
    - [Gamepiece Manipulation](#gamepiece-manipulation)
        - [Arm](#arm)
        - [Climber](#climber)
        - [Elevator](#elevator)
        - [Intake](#intake)
        - [Shooter](#shooter)
    - [Controllers/Gamepads](#controllers-and-gamepads)
        - [Pilot](#pilot-gamepad)
        - [Copilot](#copilot-gamepad)
    - [Misc](#misc)
        - [Rotary Switch](#rotary-switch)
        - [LEDs](#leds)
- [Physical Devices](#physical-devices)
    - [CAN Bus Devices](#can-bus-devices)
    - [DIO Devices](#dio-devices)
    - [ANALOG Devices](#analog-devices)
- [Naming Conventions](#naming-conventions)

# Subsystems

<sup>[(Back to top)](#table-of-contents)</sup>

> Subsystems defined in code that control physical motors and sensors to make the robot act like... well, a _robot_.

## Chassis and Navigation

> Subsystems that are critical to the base robot, allowing us to drive the chassis and navigate the field.

### Swerve Drivetrain

Organized under the `drivetrain` folder, the `DrivetrainSubSys.java` class defines a four-module swerve drivetrain, run by 8 Falcon 500's and 4 CTRE CanCoders. Our particular modules are [SDS Mk4i's](https://www.swervedrivespecialties.com/products/mk4i-swerve-module).

Usage:

1. In `Robot.java`, declare your drivetrain in the class, then instantiate the drivetrain in `robotInit()`.

```java
public class Robot extends TimedRobot {
    public static DrivetrainSubSys swerve;

    @Override
    public void robotInit() {
        swerve = new DrivetrainSubSys();
    }
}
```

2. When you need to drive the robot and therefore the drivetrain and its swerve modules, you can call the `SwerveDriveCmd.java` command under `drivetrain/commands/` in the following ways (I am simplifying the actual command calling and running for clarity's sake):

```java
// (pretend we're in a file that can call commands)
import frc.robot.drivetrain.commands.SwerveDriveCmd;

// the most common way to drive the robot:
SwerveDriveCmd(
    () -> Robot.pilot.getDriveFwdPositive(),          // get input for forward/backward, with forward being positive
    () -> Robot.pilot.getDriveLeftPositive(),         // get input for side to side, with left being positive
    () -> Robot.pilot.getDriveRotationCCWPositive(),  // get input for rotation, possibly from triggers
    true                                              // drive robot in field-relative mode.
);
```

3. And optionally, implement ways to activate nice-to-have features such as:
    - `DrivetrainCmds.LockSwerveCmd()`: 'lock' the wheels in place by turning them all to face towards the center of the robot
    - `DrivetrainCmds.ZeroGyroHeadingCmd()`: re-calibrate the gyro, with zero being the direction the robot is facing at the moment the command is issued 

Important Functions in `DrivetrainSubSys.java`:

- `drive()`: these overloaded functions are used to tell the swerve modules to drive in the desired way. The main function (not overloaded) handles converting target x, y, and rotation speeds to speeds and angles the wheels should be at.
- `stop()`: used to, obviously, stop the motors right where they are. Calls the deprecated `WPI_TalonFX.stopMotor()` method.
- `resetFalconAngles()`: resets each module's rotation motor's encoder to be what it should be, based on the absolute CanCoder angle.
- `resetGyro()`: used to set the gyro's heading to 0 by default, or otherwise another given angle.
- `getPoseMeters()`: returns the current position of the robot on the field, based on Odometry.
- `setModuleStateS()`: while not important to manually control, this method is critical in driving the robot. Given a list of `SwerveModuleStates[] desiredStates`, this tells each module where they should be and what they should be doing.

Finally, the `DrivetrainConfig.java` file has constants that can be adjusted to tune alignment of wheels (offset from each absolute encoder's '0' heading to the true robot '0' heading), PID parameters for smoother driving, and robot chassis sizes (kinematics).

### Autonomous

> _[TBD]_

### Vision

> _[TBD]_

## Gamepiece Manipulation

> Subsystems tied to physical mechanisms that allow us to interact with the field and manipulate game pieces during matches.

### Arm

Organized under the `arm` folder, `ArmSubSys.java` defines an arm controlled by either (1 or 2) Redlines or Vex Pro motors, that swings from inside the robot to outside the robot. This year, the arm is mounted to a [2-stage elevator](#elevator) for vertical motion, and has an [intake](#intake) mounted on one end.

Responsibilities of the Arm:

- Ground-pickup gamepieces
- Score in Amp
- Score in Trap
- Feed gamepiece to [Shooter](#shooter) for scoring in Speaker

> [insert code stuff for arm here]

### Climber

Organized under the `climber` folder, `ClimberSubSys.java` defines a climber articulated by 1 Falcon 500 motor. Mechanically, the climber is made up of two ["Climber-In-A-Box"-es](https://www.andymark.com/products/climber-in-a-box) with a CF (Constant-Force) spring inside, winched by the 1 motor.

Responsibilities of the Climber:

- Raise to let go of chains on the Stages
- Raise to lower robot to the ground
- Lower to grab hold of chains on the Stages
- Lower to raise robot up in the air

> [insert code stuff for climber here]

### Elevator

Organized under the `elevator` folder, `ElevatorSubSys.java` defines a 2-stage Elevator raised and lowered by either (1 or 2) Falcon 500's. Mechanically, the Elevator is a cascading elevator. The [Arm](#arm) is attached to the Elevator.

Responsibilities of the Elevator:

- raise to allow [Arm](#arm) and [Intake](#intake) to score in Amp
- raise to allow [Arm](#arm) and [Intake](#intake) to score in Trap
- lower to allow [Intake](#intake) to pick up gamepieces off of the ground
- lower to allow [Intake](#intake) to feed the [Shooter](#shooter) a gamepiece to score in Speaker

> [insert code stuff for elevator here]

### Intake

Organized under the `intake` folder, `IntakeSubSys.java` defines an intake run by 1 motor. The Intake is attached to one end of the [Arm](#arm) and can run in both directions. The Intake can manipulate game pieces by spinning the sets of horizontal rollers. There is also a small cage to hold the gamepiece, with a sensor to tell us if a gamepiece has been captured or not.

Responsibilities of the Intake:

- Suck in gamepieces from the ground or possibly Human Player Station
- Eject gamepieces into the Amp or Trap
- Feed gamepieces into the shooter for scoring in the Speaker

> [insert code stuff for intake here]

### Shooter

Organized under the `shooter` folder, `ShooterSubSys.java` defines a fixed shooter, that only ejects gamepieces with a single motor, at a single angle, possibly at a single speed. The Shooter stays in one place and is fed by the [Intake](#intake).

Responsibilities of the Shooter:

- Shoot gamepieces fed by the [Intake](#intake) to score in the Speaker.

> [insert code stuff for shooter here]

## Controllers and Gamepads

> Controllers used by the Human Drivers to control the robot together.

### Pilot Gamepad

> [pilot text here]

### Copilot Gamepad

> [coplilot text here]

## Misc

### Rotary Switch

> [rotary switch text here]

### LEDs

> [led text here]

## Physical Devices

<sup>[(Back to top)](#table-of-contents)</sup>

### CAN Bus Devices

> [can text here]

### DIO Devices

> [dio text here]

### ANALOG Devices

> [analog text here]

## Naming Conventions
