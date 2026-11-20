---
name: solidcraft-mcp
description: Use when an agent first connects to SolidCraft over its MCP server - choosing a headless or bridged session, running commands through execute and batch, waiting for long requests with pending and cancel, and which files it may reach - before the modelling skills take over.
---

# Using SolidCraft over MCP

SolidCraft's MCP server speaks JSON-RPC over stdio (`solidcraft-cli mcp`). Every tool runs engine commands, the same ones the toolbar and scripts use, so no dialog ever opens. Full reference in `docs/mcp.md`. The task skills carry the modelling rules; an agent without files reads them with `skill_reference {"skill":"solidcraft-modeling"}` (also `-editing`, `-checking`, `-mechanisms`, `-sheetmetal`).

## Pick a session
- **Headless** (default): `solidcraft-cli mcp`, or `--in part.sc` to start from a file. A design session in memory, no window; screenshots are GPU renders.
- **Bridged**: the app with its control channel on (Preferences > Assistant > "Let AI agents connect", or `--control 7878`), then `solidcraft-cli mcp --connect 127.0.0.1:7878`: the agent models in the open app.
- A bridged server starts even when the app is not running yet. Until it is, every call answers `no SolidCraft app is listening at HOST:PORT` and how to start it; safe to retry once it runs. The same error means the app quit mid-session, or hit its 16-connection limit: inspect the design before sending a change again.
- Client config: `solidcraft-cli mcp setup` writes a `solidcraft` entry into every client it finds, or add `"command": "solidcraft-cli", "args": ["mcp"]` by hand.

## Call rhythm
- One feature is several commands; send them together in one `batch {"commands":[...]}` (default `rollback: true` restores the design if a step fails). A free mistake costs one `undo`.
- Find commands with `list_commands {"filter":"fillet"}` and `describe_command {"id":"solid.fillet"}`, never the unfiltered catalog.
- Keep replies small: `"verbose": false` in the `execute` or `batch` arguments, `--compact` where output is paid for.
- Verify with `measure {}` against numbers computed by hand; a cut that misses still reports success. `execute` `view.pick` answers ray casts headless too, so picks do not depend on a window.

A feature is commands such as `"command":"sketch.create", "params":{"plane":"XY"}`, then `sketch.finish` and `solid.extrude`.
## Long requests
A call that runs past 60 seconds answers `still running:` - not failed, not cancelled; do not send it again. Poll `pending {}` (60 s per call) or `cancel {}` (removed, stopped, or `cannot be cancelled:` when it is already applying). Set the client's per-call timeout above 330 s to see the error.

## Files
`open`, `save`, `export` and a `screenshot` with a `path` reach only the agent's folders: the design's folder (inside your home, not hidden), folders the agent opened or saved in this session, the scratch and recovery folders, and every `--allow-path DIR`. A refusal such as `outside the folders this agent may use` is fixed by adding `--allow-path DIR` to the server's args in the client config and restarting - not by saving some other way.

## Errors
An unknown tool or argument is a JSON-RPC `-32602` error; an engine failure is a result with `isError: true` and the engine's message, which you can read and correct. `--param-check warn` adds `param_warnings` for misspelled keys (`strict` refuses them); the default is `off`.

For what to build and the traps (sketch axes, edge picks), load the task skills before modelling. Written against SolidCraft 0.18 and later.
