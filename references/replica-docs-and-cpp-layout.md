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
| UserApp/App | Application entry, robot modes, orchestration | `UserApp/app_entry.cpp` |
| Ctrl/Domain | Hardware-independent logic and algorithms | `Ctrl/Chassis/*`, `Ctrl/Motor/*` |
| Interface | Pure interfaces and shared data contracts | `Interface/Can/can_bus.h` |
| Port/Platform | Hardware-specific implementations | `Port/can_bus_stm32.cpp` |
| Core/Generated | Vendor/HAL/generated code | `Core/Src/main.c` |
| Tests/Sim | Host-side tests and fake hardware | `Tests/fake_can_bus.cpp` |

## Dependency Rule

`UserApp -> Ctrl -> Interface <- Port -> Core/HAL`

`Ctrl` must not include hardware vendor headers or call HAL/SDK functions directly.

## Vertical Slices

| Slice | Goal | Interfaces | Port | Verification |
|---|---|---|---|---|
```

## Recommended C++ Patterns

- Define small pure interfaces for hardware boundaries: CAN bus, serial bus, GPIO, timers, clocks, IMU, motor driver, display.
- Use plain structs for protocol data and sensor samples. Document units in field names or comments.
- Keep parsing/encoding close to the domain protocol, not inside HAL callback code.
- Make `Port` classes thin adapters from interfaces to platform APIs.
- Put safety limits in domain logic when they are platform-independent; put electrical/pin/bus constraints in `Port`.
- Provide fake ports for host-side verification when hardware is unavailable.

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

## Example Layout From User Preference

The user prefers a layout like:

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

