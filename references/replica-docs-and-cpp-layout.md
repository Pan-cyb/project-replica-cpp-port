# Replica Docs And C++ Layout

Use these templates when a replica project needs durable context. Keep them concise and update incrementally.

## `docs/ORIGINAL_PROJECT.md`

```markdown
# Original Project

## Source

- Repository:
- Local source path:
- Branch/tag/commit:
- License:
- Last checked:

## What It Does

Briefly describe the original project's purpose, target hardware/platform, and core user-visible behavior.

## Build And Run

- Language/runtime:
- Build system:
- Main commands:
- Required dependencies:

## Important References

- Upstream docs:
- Protocol docs:
- Hardware docs:
```

## `docs/ORIGINAL_ARCHITECTURE.md`

```markdown
# Original Architecture

## Module Map

| Module | Responsibility | Key Files | Notes |
|---|---|---|---|

## Runtime Flow

Describe startup, main loops/tasks, callbacks/interrupts, state machines, and data flow.

## Interfaces And Protocols

Record message formats, bus protocols, topics, CAN IDs, serial frames, units, timing, and error handling.

## Hardware Assumptions

List boards, sensors, motors, drivers, pins, buses, rates, physical constraints, and safety assumptions.

## Porting Notes

Record behavior that must be preserved and behavior that can be simplified or redesigned.
```

## `docs/REPLICA_STATUS.md`

```markdown
# Replica Status

## Current Goal

## Current Architecture

## Implemented Features

## Missing Features

## Known Issues

## Build / Run / Test Commands

## Verified Hardware Or Simulation
```

## `docs/REPLICA_PROGRESS.md`

```markdown
# Replica Progress

| Original Feature | Local Target | Status | Evidence | Next Step |
|---|---|---|---|---|
|  |  | not started / studying / scaffolded / implemented / tested / deferred |  |  |
```

## `docs/PORTING_PLAN.md`

```markdown
# Porting Plan

## Target C++ Layers

| Layer | Responsibility | Example Files |
|---|---|---|
| Application | App entry, robot modes, orchestration, task wiring | `App/app_entry.cpp`, `UserApp/robot_app.cpp` |
| Domain / Control | Hardware-independent logic, algorithms, state machines | `Domain/Chassis/*`, `Ctrl/Motor/*` |
| Interfaces / Contracts | Pure interfaces, shared data contracts, protocol structs | `Interfaces/Can/can_bus.h` |
| Platform / Adapters | Hardware, OS, middleware, and board-specific adapters | `Platform/Stm32/can_bus.cpp`, `Port/can_bus_stm32.cpp` |
| BSP / HAL / Generated | Vendor/HAL/generated code, startup, board config | `Core/Src/main.c`, `BSP/board_init.c` |
| Tests / Simulation | Host-side tests, fake hardware, replay, simulation | `Tests/fake_can_bus.cpp`, `Sim/` |

## Dependency Rule

`Application -> Domain -> Interfaces <- Platform -> BSP/HAL/generated code`

The domain/control layer must not include hardware vendor headers or call HAL/SDK/middleware APIs directly.

## Vertical Slices

| Slice | Goal | Interfaces | Port | Verification |
|---|---|---|---|---|
```

## Embedded Architecture Principles

The exact directory names are flexible. Preserve the responsibility boundaries.

### Application Layer

- Own startup orchestration, mode selection, task scheduling, and high-level feature wiring.
- Keep hardware calls out of this layer except through injected interfaces or services.
- Good names: `App`, `UserApp`, `Application`, `Tasks`, `RobotApp`.

### Domain / Control Layer

- Own robot behavior, algorithms, kinematics, state machines, safety policies, protocol parsing/encoding when protocol is part of the product behavior.
- Keep it portable enough to compile on a PC with fake interfaces when practical.
- Good names: `Domain`, `Ctrl`, `Control`, `Robot`, `Chassis`, `Motion`, `Motor`.

### Interfaces / Contracts Layer

- Define hardware and middleware boundaries as small interfaces.
- Define DTOs/data contracts: CAN frames, IMU samples, motor commands, odometry, timestamps, units.
- Avoid depending on concrete platform headers.
- Good names: `Interface`, `Interfaces`, `Contracts`, `Abstraction`, `HALInterface`.

### Platform / Adapter Layer

- Implement interfaces using concrete APIs: STM32 HAL, Arduino libraries, Linux file descriptors, ROS publishers/subscribers, serial/CAN/SPI/I2C drivers.
- Translate interrupts/callbacks/events into domain-facing data.
- Good names: `Port`, `Platform`, `Adapters`, `Drivers`, `Board`.

### BSP / HAL / Generated Layer

- Keep vendor-generated startup, CubeMX, linker scripts, board pin initialization, RTOS configuration, and low-level C drivers isolated.
- Regenerating vendor code should not overwrite domain logic.
- Good names: `Core`, `BSP`, `Generated`, `CubeMX`, `Vendor`, `HAL`.

### Tests / Simulation Layer

- Provide fake ports, protocol replay, host-side unit tests, and simulation harnesses.
- Use this to verify the domain/control layer before touching hardware.
- Good names: `Tests`, `test`, `Sim`, `Simulation`, `Host`.

## Recommended C++ Patterns

- Define small pure interfaces for hardware boundaries: CAN bus, serial bus, GPIO, timers, clocks, IMU, motor driver, display.
- Use plain structs for protocol data and sensor samples. Document units in field names or comments.
- Keep parsing/encoding close to the domain protocol, not inside HAL callback code.
- Make `Port` classes thin adapters from interfaces to platform APIs.
- Put safety limits in domain logic when they are platform-independent; put electrical/pin/bus constraints in `Port`.
- Provide fake ports for host-side verification when hardware is unavailable.
- Prefer constructor injection or explicit `init(...)` wiring over global singleton hardware access.
- Use compile-time configuration only for platform selection; do not scatter `#ifdef STM32` through domain logic.
- Wrap RTOS/threading primitives behind small scheduler/timer/clock interfaces if timing behavior must be portable.
- Keep units explicit: `rpm`, `rad_s`, `mm`, `mps`, `ticks`, `us`.

Example:

```cpp
class CanBus {
public:
    virtual ~CanBus() = default;
    virtual bool send(const CanFrame& frame) = 0;
};

class Stm32CanBus final : public CanBus {
public:
    bool send(const CanFrame& frame) override;
};
```

## Optional Layout Example

One valid embedded layout is:

```text
UserApp/
Ctrl/
Interface/
Port/
Core/
```

This resembles the project example at:

```text
G:\Github\Electronics Design Contest\Train_2_1\Myproj\Ctrl
```

Treat this as a preferred style, not a mandatory path.

Another equally valid layout is:

```text
App/
Domain/
Interfaces/
Platform/
BSP/
Tests/
```

Choose names that fit the target project, but keep dependencies pointing inward.
