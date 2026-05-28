---
name: project-replica-cpp-port
description: Use when the user wants to replicate, reimplement, port, or study an existing project step by step, especially when the source project is in another language and the target should be rebuilt in C++ with application logic separated from hardware compatibility layers. Also use when maintaining reusable replica-project context, recording the original project URL, analyzing original architecture, tracking local feature/progress status, creating or updating project documentation, or continuing a replica project from saved docs without re-explaining background.
---

# Project Replica C++ Port

Guide replica projects as a documented, incremental port rather than a one-off explanation. Preserve reusable context in repo docs, read those docs on every use, and favor a C++ architecture that separates business/application logic from hardware-specific adapters.

## First Step

Find the repository root from the current working directory. Before advising or editing, read existing context when present:

```text
AGENTS.md
docs/ORIGINAL_PROJECT.md
docs/ORIGINAL_ARCHITECTURE.md
docs/REPLICA_STATUS.md
docs/REPLICA_PROGRESS.md
docs/PORTING_PLAN.md
docs/handoff/
```

If similar docs already exist under different names, use them instead of duplicating. Do not rewrite existing docs wholesale; append, correct, or add focused sections.

When the user provides an original project URL or local source path, record it in `docs/ORIGINAL_PROJECT.md` so later sessions do not need to ask again.

## Documentation Set

Create missing docs only when useful for the current task.

```text
AGENTS.md
  Durable project rules, architecture boundaries, commands, naming conventions, and safety constraints.

docs/ORIGINAL_PROJECT.md
  Original repository URL, local source path, license notes, target version/commit, setup notes, and important external docs.

docs/ORIGINAL_ARCHITECTURE.md
  Original project module map, runtime flow, data/control paths, dependencies, protocols, and hardware assumptions.

docs/REPLICA_STATUS.md
  Current local implementation state, supported features, missing features, known bugs, build/run commands, and verified hardware.

docs/REPLICA_PROGRESS.md
  Checklist or table mapping original features to local replica progress: not started, studying, scaffolded, implemented, tested, deferred.

docs/PORTING_PLAN.md
  Step-by-step migration plan, target architecture, interfaces to define, hardware adapters to write, and test strategy.

docs/handoff/YYYY-MM-DD_HHMM_handoff.md
  Per-session handoff containing decisions, changed files, verified commands, blockers, and next steps.
```

For detailed templates and the recommended C++ layout, read `references/replica-docs-and-cpp-layout.md` when creating or updating these docs.

## Analysis Workflow

When starting or resuming a replica:

1. Identify the original project source: URL, branch/commit/tag, local path, license, language, build system, and target hardware.
2. Map the original architecture before coding: modules, entry points, protocols, data models, timing loops, state machines, drivers, and external dependencies.
3. Compare with the local replica: existing files, build system, implemented features, hardware assumptions, and missing abstractions.
4. Propose a small next slice. Prefer tracer-bullet progress: one vertical feature path from interface to implementation to test.
5. Update the docs after meaningful discoveries or implementation changes.

If the user asks for teaching or step-by-step guidance, explain the next step plainly, then execute or prepare the smallest practical task unless they only want an explanation.

## C++ Porting Architecture

Prefer this separation unless the existing project strongly suggests a better local convention:

```text
UserApp/ or App/
  Application entry points, task orchestration, user-facing behavior, high-level robot modes.

Ctrl/ or Domain/
  Hardware-independent control logic, state machines, algorithms, motor/chassis/sensor abstractions.

Interface/
  Pure interfaces and shared data types: bus interfaces, sensor base classes, protocol frames, DTOs.

Port/ or Platform/
  Hardware compatibility layer: STM32/Arduino/Linux/ROS adapters, HAL calls, serial/CAN/SPI/I2C implementations.

Core/ or Generated/
  Vendor-generated startup/HAL/CubeMX files. Keep isolated and avoid mixing domain logic here.

Tests/ or Sim/
  Host-side tests, fake ports, simulation harnesses, protocol replay, regression checks.
```

Dependency direction must point inward:

```text
UserApp -> Ctrl -> Interface <- Port -> Core/HAL
```

`Ctrl` may depend on `Interface`, but not on `Port`, `Core`, HAL headers, board pins, or vendor generated code. `Port` implements interfaces and translates to hardware APIs. This keeps later hardware replacement cheap.

When source project code is in another language, translate behavior and contracts before translating syntax. Preserve protocols, message fields, timing assumptions, state transitions, error handling, and calibration semantics in docs.

## Working Rules

- Keep original-project analysis and local implementation status separate.
- Record uncertain claims as assumptions with evidence paths.
- Prefer interfaces before concrete hardware ports.
- Use fake/mock ports or host-side tests for control logic before touching hardware-specific code.
- Keep generated/vendor code isolated.
- Avoid large rewrites unless the docs show why the target architecture requires them.
- When changing hardware compatibility code, document the expected hardware, pins, buses, baud rates, CAN IDs, units, and safety constraints.
- After each meaningful change, update progress docs and include verified commands.

## Handoff

When ending a session or when the user asks to preserve context, update the status/progress docs and create a handoff note under `docs/handoff/`. Include:

```text
Current goal
Original project source and version
Local architecture decisions
Completed work
Changed files
Verified commands
Known issues and assumptions
Next recommended steps
```

