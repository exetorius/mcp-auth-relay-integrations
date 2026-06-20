# VibeUE — Agent Instructions

You are connected to an Unreal Engine editor via the VibeUE MCP server. You have 11 tools for Python execution, asset management, domain skill loading, log reading, terrain generation, web research, and direct editor control.

---

## Core workflow

**Discover before you execute.** Never write Python code for a class or function you haven't confirmed exists with that exact signature. The UE Python API is large and wrong names fail silently or error at runtime.

```
vibeue-skills-manager (load relevant domain pack)
  → discover_python_class / discover_python_function (confirm signatures)
    → execute_python_code (run the code)
      → read_logs (if something went wrong)
```

**Load skills proactively, not reactively.** Before working on blueprints, animation, materials, niagara, UMG, state trees, enhanced input, audio, or landscape — load the relevant skill pack first with `vibeue-skills-manager`. Call `action='list'` for the current skill menu, `action='suggest'` to match a task to a skill, `action='load'` to pull one in. Load multiple skills in one call; it is more efficient than separate calls.

**Use manage_asset for all asset operations.** Do not reach for raw Python when manage_asset covers the operation. It handles Content Browser paths, reference tracking, and edge cases that raw Python gets wrong. Always use Content Browser paths (`/Game/Blueprints/BP_Player`), never file system paths. Never simulate a move with duplicate + delete — use the `move` action.

---

## Python rules

- `import unreal` — always lowercase. `import Unreal` fails.
- Subsystems: `unreal.get_editor_subsystem(unreal.EditorActorSubsystem)` — `unreal.EditorLevelLibrary` is removed in UE 5.7+.
- After adding variables, functions, or components to a Blueprint: always call `unreal.BlueprintService.compile_blueprint(path)`. Re-read the graph before claiming the task is complete.
- 60 second execution timeout. For long operations, break them into smaller steps.
- Known limitation: `add_set_variable_node` and `add_get_variable_node` fail for object reference variable types. Use `execute_python_code` with the direct Python API as a workaround.

---

## Loop prevention

- Track outcomes, not just arguments. If you get the same result twice from the same tool call, stop — do not retry.
- Maximum 3 attempts at the same operation.
- After 2 failures: explain what you tried and ask the user. Do not keep hammering.
- Try at most one alternative approach after a failure. If that also fails, stop.

---

## Editor control (PIE, standalone, profiling)

Use the `editor_control` tool to drive the editor without writing Python. Key actions:

- `start_pie` / `stop_pie` / `pie_status` — Play In Editor lifecycle
- `start_standalone` / `stop_standalone` / `standalone_status` — launch the game as a separate process with Unreal Insights trace attached
- `profiler_start` / profiler controls — start/stop Unreal Insights traces with specific channels (frame, cpu, gpu, log, loadtime, object, stats)
- `analyse` — read results from the last trace file
- `bookmark` / `region_begin` / `region_end` — annotate a running trace

Load the `profiling` skill before using trace or frame-time features — it covers channel name format, gotchas, and patterns.

---

## When UE is not running

`tools/list` is served from the cached manifest and will still work. `tools/call` will fail with a clear error. Tell the user to start Unreal Engine and wait for the VibeUE MCP server to come online (check UE Output Log for "MCP Server started at http://127.0.0.1:8088").
