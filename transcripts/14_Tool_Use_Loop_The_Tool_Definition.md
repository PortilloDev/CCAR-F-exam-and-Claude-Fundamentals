# Lecture 14: Tool-Use Loop: The Tool Definition

**Section:** Tools and MCP  
**Duration:** ~5 min | **Lecture ID:** 58184787

---

## Transcript

We're looking now at the tool definition. So this is now crucial, because we need a good way to describe to Claude what a tool is, how it works, so that Claude can make a good decision on when to call this tool, and as well how to call it, so with the right parameters. Of course, we need the core of it, the function itself. So it's important that the parameter names you pick are descriptive, and as well, whenever you have the input coming in, have an input checker at the beginning, and it should throw an error that is well described, so that Claude actually understands what was wrong about the way it wanted to call this tool.

So what you need to write are three parts, so that the tool is well understood by Claude:

1. **Name** — e.g., `get_current_datetime`. This is dispatched to the real method in the background. Use descriptive, unambiguous names.

2. **Description** — Explains the purpose of the tool. E.g., "Returns the current date and time." This tells Claude WHEN to call the tool.

3. **Input schema** — A JSON-formatted schema object containing the parameters Claude needs to fill out to call the tool. This defines HOW to call the tool.

You can define the tool with the `ToolParam` object:
- `name`: the tool's identifier (e.g., `get_current_datetime`)
- `description`: "Returns the current date and time" + guidance on formatting
- `input_schema`: JSON schema object with type, properties, required fields

**Input schema structure:**
- `type`: always `"object"` at the top level
- `properties`: list of all parameters (each with a `description`, `type`, and optionally `enum` for fixed values)
- `required`: list of parameter names that must always be provided

**Tips for creating the JSON schema:**
1. Use Claude Code — ask it to turn your code description into a JSON object
2. Define sample arguments, put them through a JSON Schema converter, then remove the `$schema` key

Always review the JSON object carefully — descriptions within the schema are critical for Claude to fill parameters correctly.
