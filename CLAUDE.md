# CLAUDE.md — Godot-AI-Dialogic

A **Godot-MCP extension** that wraps the third-party [**Dialogic**](https://github.com/dialogic-godot/dialogic)
addon (dialogue / visual-novel authoring), shipped as a **source-only NuGet package**
(`com.IvanMurzak.Godot.MCP.Dialogic`) that compiles inside a consumer's Godot project against the
consumer's own GodotSharp. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template). The packaging recipe is
the load-bearing detail — read `docs/source-only-nuget-recipe.md`.

This is an **addon-dependent ("Class B")** extension: Dialogic's classes (`DialogicTimeline`,
`DialogicCharacter`, the `Dialogic` autoload, `DialogicResourceUtil`) are **not** in GodotSharp, and the
package **must not depend on the addon**. So the tools reference Dialogic's classes **only by string
name**, resolved + invoked at runtime, and **presence-gate** every editor tool so a missing addon returns
a clean structured `installed: false` result instead of crashing. The addon is **never** vendored,
submoduled, or downloaded by this repo — installing Dialogic is the consumer's own responsibility (CI
downloads a pinned copy, **2.0-alpha-19**, only to exercise the e2e leg).

**Scope: resource-level authoring.** The tools author Dialogic **timeline** (`.dtl`) and **character**
(`.dch`) **resources** — the AI-friendly surface — not Dialogic's game-runtime API (`Dialogic.start(...)`,
`Dialogic.current_timeline`). Dialogic's timeline/event internals are the most version-volatile of the
Class-B addons, so the first tool family is deliberately scoped to **create / append / read** at the
resource level, not the full event-type catalogue.

## Layout

- `src/Godot-AI-Dialogic/` — the source-only package (`Godot.NET.Sdk`).
  - `Runtime/Tools/Tool_Dialogic.cs` — the `[AiToolType]` family (one partial class).
  - `Runtime/Tools/Tool_Dialogic.Ids.cs` — all tool-id consts (pure-managed; pinned by tests).
  - `Runtime/Tools/Tool_Dialogic.Defaults.cs` — `dialogic-defaults` (pure-managed tool).
  - `Runtime/Interop/AddonInterop.cs` — dynamic name-resolution helper (global script-class list →
    `GD.Load` → `New()`); pure-managed resolution/result-shaping, the `Resource`/`Node`-constructing calls
    stay in `#if TOOLS` editor tools.
  - `Runtime/Interop/AddonGate.cs` — the shared `AddonGateResult` shape + `NotInstalled(...)` factory
    (pure-managed, unit-tested).
  - `Runtime/Dialogic/DialogicNames.cs` — the addon's class/member **snake_case** name constants + event
    type / file-extension (`.dtl`/`.dch`) constants (no compile-time types exist, so the constants ARE the
    contract — unit-tested).
  - `Editor/Tools/Tool_Dialogic.{CreateCharacter,CreateTimeline,AddEvent,GetTimeline}.cs` — editor tools
    behind `#if TOOLS` (touch `ResourceSaver`/live resources; main-thread-marshalled; presence-gated FIRST
    line; E2E-verified).
  - `build/com.IvanMurzak.Godot.MCP.Dialogic.props` — the source-injection props (auto-imported by NuGet
    in the consumer; MUST stay named `<PackageId>.props`).
- `tests/Godot-AI-Dialogic.Tests/` — xUnit specs for the pure-managed sources only (no Godot binary):
  the tool-id consts, the `AddonGateResult` shape + hint text, the Dialogic name + event-type constants.
- `testbed/Dialogic-Testbed.csproj` — a consumer `Godot.NET.Sdk` project that restores the local-packed
  package; `dotnet build` of it is the source-injection proof.

## Tools

| Tool | Kind | File |
| --- | --- | --- |
| `dialogic-defaults` | pure-managed | `Runtime/Tools/Tool_Dialogic.Defaults.cs` |
| `dialogic-create-character` | editor | `Editor/Tools/Tool_Dialogic.CreateCharacter.cs` |
| `dialogic-create-timeline` | editor | `Editor/Tools/Tool_Dialogic.CreateTimeline.cs` |
| `dialogic-add-event` | editor | `Editor/Tools/Tool_Dialogic.AddEvent.cs` |
| `dialogic-get-timeline` | editor | `Editor/Tools/Tool_Dialogic.GetTimeline.cs` |

The editor tool set is confirmed/adjusted against the installed Dialogic **2.0-alpha-19** API in the
implement step. The presence gate probes Dialogic's installed classes (the `Dialogic` autoload / the
`DialogicTimeline` + `DialogicCharacter` global script-classes) — verify the exact presence signal against
the pinned addon in step 02, since the resource-level tools need those resource types to be registered.

## Build / test (no Godot binary, addon absent)

```bash
dotnet build src/Godot-AI-Dialogic/Godot-AI-Dialogic.csproj   # source-only package compiles tools (addon NOT needed)
dotnet test  tests/Godot-AI-Dialogic.Tests/Godot-AI-Dialogic.Tests.csproj
dotnet pack  src/Godot-AI-Dialogic/Godot-AI-Dialogic.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/Dialogic-Testbed.csproj                  # consumes the local package (injection proof)
```

`Godot.NET.Sdk` supplies GodotSharp from NuGet, so no Godot install is needed to build/test/pack or to
prove the source-injection recipe (the testbed build is a faithful proxy for `godot --build-solutions`).
**`dotnet build -c Debug` MUST exit 0 with the Dialogic addon ABSENT** — the Class-B no-dependency gate:
the package compiles on a machine that never installed the addon, because it never names an addon type
(only string names). When proving locally, note `dotnet pack` re-uses the **global NuGet cache** for an
already-cached version: if you re-pack the same `Version`, clear
`~/.nuget/packages/com.ivanmurzak.godot.mcp.dialogic/<ver>` (or pack a unique version) before re-restoring
the testbed, or you'll silently build the stale cached source.

## Conventions

- Root namespace `com.IvanMurzak.Godot.MCP.Dialogic`. Every `.cs` starts with the Apache-2.0 header.
- Pure-managed tools + the `AddonInterop` resolution/`AddonGate` result shape + the name/event-type
  constants → `Runtime/` (outside `#if TOOLS`, unit-testable); editor-driving tools → `Editor/` (behind
  `#if TOOLS`, every Godot call via `MainThread.Instance.Run(...)`, the presence gate as the FIRST line,
  E2E-verified).
- **No GodotSharp and no addon dependency** — the package declares ONLY the `com.IvanMurzak.McpPlugin` /
  `com.IvanMurzak.ReflectorNet` min-version pins; Dialogic is referenced **by string name only** (CI
  asserts the nuspec). Keep the MCP pins in lockstep with the core Godot-MCP addon; bump with
  `commands/update-core.ps1`.
- Dialogic member names are **GDScript `snake_case`** (not C# PascalCase); event types and resource file
  extensions (`.dtl`, `.dch`) are plain strings. Centralize + unit-test them (there are no compile-time
  types to lean on).
- One `[AiToolType] partial class Tool_Dialogic`; one `[AiTool]` method per partial-class file. New
  pure-managed sources must be added to the test csproj `<Compile Include>` list to be unit-tested.

## Find detail in

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece) + the consumer story.
- `docs/ci.md` — workflows, the version gate, multi-Godot matrix, the publish secrets.
