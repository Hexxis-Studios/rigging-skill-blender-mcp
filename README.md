# Blender MCP Character Rigging Skill

A portable skill for Claude and other agents working with Blender MCP. It collects practical character-rigging methods, verified repairs, mistakes, and rejected experiments from real character-authoring work.

**[Read the skill](SKILL.md)** · **[Download the skill ZIP](downloads/blender-mcp-character-rigging.zip?raw=true)**

## Use it with your agent

Download and extract the ZIP, or clone this repository. Give your agent access to `SKILL.md` and the `references` folder together, then use this prompt:

> Read SKILL.md and its linked references. Use this skill to inspect and rig my character through Blender MCP. Diagnose the actual failure, preserve my source, and validate the resulting deformation. Inspect screenshots from front, side, and back, and replay the failing motion before accepting a repair.

The agent needs a working connection to your Blender MCP server to inspect or edit Blender. The skill tells it to discover the available tools rather than assume connector-specific commands. No HexiVerse project files, assets, paid services, or particular game engine are required.

## What it covers

- Existing armatures, authored skin weights, and cross-limb contamination.
- Local weight repairs and nearest-triangle/barycentric transfers.
- Separate binding regions for layered clothes, skirts, robes, sleeves, hair, and headwear.
- Rest poses, bind data, imported joint origins, and retargeting diagnosis.
- Safe MCP edits and Blender Python inspection/assignment patterns.
- Visual inspection of evaluated deformation, matched failing poses, motion playback, and export/reimport checks.
- Rejected approaches and limits, so an agent does not repeat failed experiments as proven recipes.

## Files

| File | Purpose |
| --- | --- |
| [SKILL.md](SKILL.md) | Main workflow and acceptance checks |
| [references/lessons.md](references/lessons.md) | Recorded working methods, failures, and evidence limits |
| [references/mcp-and-python.md](references/mcp-and-python.md) | MCP operation guidance and adaptable Python examples |
| [downloads/blender-mcp-character-rigging.zip](downloads/blender-mcp-character-rigging.zip?raw=true) | Complete packaged skill |

## Validation and limits

The packaged skill passed skill-format validation, local-reference checks, Python example syntax checks, and ZIP integrity checks. The strongest recorded rigging evidence concerns repairs to existing imported humanoids and garments. This is not a universal automatic rig generator, and the example snippets have not been tested on your character.

Visual inspection instructions are included. Camera placement and screenshot capture use the capabilities of the connected Blender MCP server; there is no bundled universal camera/capture script. A successful autorig response or normalized weights alone do not establish a good rig.
