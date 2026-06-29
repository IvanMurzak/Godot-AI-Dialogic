<h1 align="center">Godot AI Dialogic</h1>

<p align="center">
  AI <b>MCP tools</b> for the <b>Dialogic</b> Godot addon (dialogue / visual-novel authoring) —
  an extension for
  <a href="https://github.com/IvanMurzak/Godot-MCP">Godot-MCP / AI Game Developer</a>.
</p>

`Godot-AI-Dialogic` adds a focused MCP tool family for the community
[**Dialogic**](https://github.com/dialogic-godot/dialogic) addon — a dialogue / visual-novel system for
Godot built around **timeline** and **character** resources. The tools are authored in C# with
`[AiToolType]` / `[AiTool]` (the same model as Unity-MCP and the core Godot-MCP addon) and shipped as a
**source-only NuGet package** that compiles inside any consumer's Godot project against the consumer's
own GodotSharp — no bundled Godot, no version lock. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template).

This is an **addon-dependent** extension: it drives Dialogic's resources **by name at runtime** (resolved
from Godot's global script-class list) and never takes a compile-time dependency on the addon. The tools
work at the **resource level** — they author **timeline** (`.dtl`) and **character** (`.dch`) resources,
which is the AI-friendly surface; Dialogic's game-runtime API (`Dialogic.start(...)`) is out of scope.
Every editor tool is **presence-gated** — if Dialogic is not installed, the tool returns a clean,
structured `installed: false` result (with an install hint) instead of crashing.

## Required prerequisite — Dialogic (install it yourself)

This extension **does NOT include** the Dialogic addon and does not download or vendor it. You must
install Dialogic into your own Godot project separately:

- Install **Dialogic** from the **Godot Asset Library**, or from
  **https://github.com/dialogic-godot/dialogic** (tested against **2.0-alpha-19**).
- Enable it under **Project Settings → Plugins** (this also registers the `Dialogic` autoload).

Without Dialogic installed and enabled, the editor tools below report `installed: false` and take no
action (by design).

> Dialogic is © Emilio Coppola and contributors, distributed under the **MIT License**. This extension is
> **not affiliated with, endorsed by, or sponsored by** the Dialogic project — it merely provides AI tools
> that author its resources. See the addon's own repository for its licence and terms.

## Tools

| Tool | Kind | Description |
| --- | --- | --- |
| `dialogic-defaults` | pure-managed | Return a recommended starter timeline + character description (no addon needed). |
| `dialogic-create-character` | editor (`#if TOOLS`) | Create + save a `DialogicCharacter` resource (`.dch`) at a `res://` path. |
| `dialogic-create-timeline` | editor (`#if TOOLS`) | Create + save a `DialogicTimeline` resource (`.dtl`) with a few starter text events. |
| `dialogic-add-event` | editor (`#if TOOLS`) | Append an event (text / character / choice) to an existing timeline resource. |
| `dialogic-get-timeline` | editor (`#if TOOLS`) | Read back the events in a timeline resource (read-only). |

The exact editor tool set is finalized against the installed Dialogic **2.0-alpha-19** API in the
implement step (Dialogic's timeline/event internals are version-volatile, so the first family is scoped
to resource-level **create / append / read**, not the full event-type catalogue). Pure-managed tools (no
Godot native API — the `*-defaults` tool plus the addon class/member name constants) live under
`src/Godot-AI-Dialogic/Runtime/` and are CI-unit-tested; editor-driving tools live under `Editor/` behind
`#if TOOLS`, marshal every Godot call onto the editor main thread via `MainThread.Instance.Run(...)`, and
resolve Dialogic's classes dynamically through `Runtime/Interop/AddonInterop.cs`.

## Install (in a consumer Godot project)

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon **and** the Dialogic addon
(see the prerequisite above). Then either:

- **Extensions dock** — pick it inside the Godot editor (Install → adds the `<PackageReference>` → rebuild).
- **CLI** — `godot-cli install-extension com.IvanMurzak.Godot.MCP.Dialogic`.
- **By hand** — add `<PackageReference Include="com.IvanMurzak.Godot.MCP.Dialogic" Version="x.y.z" />`
  to the consumer `.csproj` and rebuild.

After a rebuild the `[AiToolType]` tool family is auto-discovered — no registry edit.

## Build & test (no Godot binary, addon absent)

`Godot.NET.Sdk` pulls GodotSharp from NuGet, so the package builds and unit-tests headless. Because the
package references Dialogic **only by string name**, it compiles cleanly with the addon **absent**:

```bash
dotnet build src/Godot-AI-Dialogic/Godot-AI-Dialogic.csproj            # compiles tools (Godot API resolves; addon NOT needed)
dotnet test  tests/Godot-AI-Dialogic.Tests/Godot-AI-Dialogic.Tests.csproj   # pure-managed unit tests
dotnet pack  src/Godot-AI-Dialogic/Godot-AI-Dialogic.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/Dialogic-Testbed.csproj                           # consumer build = source-injection proof
```

The testbed build proves the source-injection recipe: the package's `.cs` are injected as `<Compile>`
items into the consumer and compile against the consumer's own GodotSharp. CI runs this across a
multi-Godot-version matrix; an end-to-end leg additionally boots real headless Godot, installs the core
addon **and the pinned Dialogic addon**, then drives each tool and asserts the presence-gated results.

## Docs

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece).
- `docs/ci.md` — workflows, the version gate, the multi-Godot matrix, required secrets.
- `CLAUDE.md` — maintainer notes (incl. the addon-dependent / presence-gate model).

## Publish

Source-only, version-gated release (see `docs/ci.md`): configure NuGet Trusted Publishing (OIDC) + the
`NUGET_USER` variable, bump `<Version>` (`commands/bump-version.ps1 -NewVersion x.y.z`), merge to `main`;
`release.yml` runs the full matrix, publishes the package to NuGet, and cuts an atomic GitHub Release.

License: **Apache-2.0** (this extension). The Dialogic addon it drives is MIT, © Emilio Coppola and
contributors — install it yourself (see the prerequisite above); it is never bundled here.
