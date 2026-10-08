# Claude marketplace entrypoint

This directory contains the Claude plugin marketplace catalog for this
repository.

- `marketplace.json` exposes the YouCam AI Agent plugin package at
  `./plugins/youcam-ai-agent` from the repository root.
- The installable package contains its Claude plugin manifest at
  `plugins/youcam-ai-agent/.claude-plugin/plugin.json`.
- The plugin manifest registers the skills under
  `plugins/youcam-ai-agent/skills/`.

Install from GitHub with:

```text
/plugin marketplace add perfect-corp/youcam-ai-agent
/plugin install youcam-ai-agent@youcam-ai-agent
```

If the marketplace is already installed, refresh it with:

```text
/plugin marketplace update youcam-ai-agent
```
