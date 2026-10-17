# Blender MCP Character Rigging Skill

A skill for Claude and other agents rigging characters through Blender MCP. It covers skin weights, deformation diagnosis, safe edits, and visual validation.

**[Read the skill](SKILL.md)** · **[Download the skill ZIP](downloads/blender-mcp-character-rigging.zip?raw=true)**

## Use it with your agent

Download and extract the ZIP, or clone this repository. Give your agent access to `SKILL.md` and the `references` folder together, then use this prompt:

> Read SKILL.md and its linked references. Use this skill to inspect and rig my character through Blender MCP. Diagnose the actual failure, preserve my source, and validate the resulting deformation. Inspect screenshots from front, side, and back, and replay the failing motion before accepting a repair.

The agent needs a working connection to your Blender MCP server to inspect or edit Blender. The skill tells it to discover the available tools rather than assume connector-specific commands.

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
| [references/lessons.md](references/lessons.md) | Common failures, repair methods, and their limits |
| [references/mcp-and-python.md](references/mcp-and-python.md) | MCP operation guidance and adaptable Python examples |
| [downloads/blender-mcp-character-rigging.zip](downloads/blender-mcp-character-rigging.zip?raw=true) | Complete packaged skill |

## Visual inspection

The skill instructs the agent to inspect front, side, and back views, compare body and clothing separately, and replay failing poses and full motion. Camera placement and screenshot capture use the connected Blender MCP server. A successful autorig response or normalized weights alone do not establish a good rig.
