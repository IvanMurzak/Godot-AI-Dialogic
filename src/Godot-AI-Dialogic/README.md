# Dialogic Tools

AI MCP tools for the Godot Dialogic addon.

A **source-only** MCP tool extension for [Godot-MCP / AI Game Developer](https://github.com/IvanMurzak/Godot-MCP)
that adds AI tools for the community [**Dialogic**](https://github.com/dialogic-godot/dialogic) addon
(dialogue / visual-novel authoring). The package ships C# source (no compiled DLL, no bundled Godot) that
compiles inside your Godot project against your own GodotSharp, so it never locks you to a Godot version.
It authors Dialogic's **timeline** (`.dtl`) and **character** (`.dch`) resources **by string name at
runtime** and never depends on the addon at compile time, so every tool is **presence-gated** (a missing
addon returns a clean `installed: false` result).

## Required prerequisite — Dialogic (install it yourself)

This extension **does NOT include** the Dialogic addon. Install it separately into your Godot project from
the **Godot Asset Library** or **https://github.com/dialogic-godot/dialogic** (tested against
**2.0-alpha-19**), and enable it under **Project Settings → Plugins** (this also registers the `Dialogic`
autoload). Dialogic is © Emilio Coppola and contributors under the **MIT License**; this extension is
**not affiliated with or endorsed by** it.

## Install

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon in your Godot C# project.

```bash
# via the godot-cli (resolves from the shared catalog, edits your .csproj, rebuilds)
godot-cli install-extension com.IvanMurzak.Godot.MCP.Dialogic

# …or add the reference manually and rebuild:
#   <PackageReference Include="com.IvanMurzak.Godot.MCP.Dialogic" Version="0.1.0" />
```

…or pick it from the **Extensions** dock inside the Godot editor.

After a rebuild, the extension's `[AiToolType]` tool families are auto-discovered — no registry edit.

## Tools

Every editor tool is **presence-gated**: when the Dialogic addon is not installed it returns a structured
`installed: false` result with an install hint instead of crashing. The tools work at the **resource
level** (timelines / characters), not Dialogic's game-runtime API.

| Tool | Kind | Description |
| --- | --- | --- |
| `dialogic-defaults` | pure-managed | Recommended starter timeline + character config. No addon needed. |
| `dialogic-timeline-create` | editor | Create + save a `DialogicTimeline` resource (`.dtl`) seeded with a text event. |
| `dialogic-character-create` | editor | Create + save a `DialogicCharacter` resource (`.dch`) (display name, color, description). |
| `dialogic-timeline-add-text` | editor | Append a text event (optional speaking character) to an existing timeline. |
| `dialogic-get` | editor | Read a timeline (`.dtl`) or character (`.dch`) resource's config (read-only). |

License: Apache-2.0.
