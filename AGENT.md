# AI Agent - Canvas Generation Rules

This document outlines the rules and best practices for AI agents to generate and modify Obsidian Canvas files (.canvas) effectively.

## Canvas JSON Structure

Obsidian Canvas files are JSON objects with the following structure:

```json
{
  "nodes": [/* array of node objects */],
  "edges": [/* array of edge objects */],
  "metadata": {
    "version": "1.0-1.0",
    "frontmatter": {}
  }
}
```

## Node Rules

### Node Object Format
Each node must have:
- `"id"`: Unique hexadecimal string (16 characters, 0-9a-f only)
- `"type"`: "file", "text", or other valid types
- `"x"`, `"y"`: Integer coordinates for positioning
- `"width"`, `"height"`: Integer dimensions

### File Nodes
For file nodes:
- `"file"`: Relative path to the markdown file (e.g., "Docs/Agent/AGENT.md")
- Ensure the file exists before linking

### Text Nodes
For text nodes:
- `"text"`: String content (use \n for line breaks)
- `"styleAttributes"`: Empty object {}

## Edge Rules

### Edge Object Format
Each edge must have:
- `"id"`: Unique identifier (can be descriptive string)
- `"fromNode"`, `"toNode"`: IDs of connected nodes
- `"fromSide"`, `"toSide"`: "left", "right", "top", "bottom"
- `"toFloating"`: false for connected edges
- `"styleAttributes"`: Empty object {}

## Generation Best Practices

1. **Use Valid Hexadecimal IDs**: Node IDs must be 16-character hexadecimal strings (e.g., "35f7a8b9c1d2e3f4"). Avoid non-hex characters.

2. **Unique IDs**: Ensure all node and edge IDs are unique within the canvas.

3. **Positioning**: Use reasonable integer coordinates. Negative values are allowed for off-center layouts.

4. **File Existence**: Verify linked files exist before creating file nodes.

5. **JSON Validity**: Ensure the JSON is well-formed. Use tools to validate before writing.

6. **Incremental Changes**: When adding nodes, update the entire JSON structure at once to avoid corruption.

7. **Backup**: Consider backing up the canvas before major changes.

## Common Issues and Solutions

- **Nodes Disappear on Refresh**: Likely due to invalid IDs or JSON syntax errors.
- **Unmovable Nodes**: Check for duplicate IDs or invalid coordinates.
- **File Links Broken**: Ensure paths are correct and files exist.

## AI Agent Responsibilities

When generating canvases:
- Parse existing canvas structure if modifying
- Generate unique hex IDs for new elements
- Position nodes logically (e.g., central hub with connected satellites)
- Create meaningful connections between related content
- Validate JSON before final output
- Provide clear explanations of changes made