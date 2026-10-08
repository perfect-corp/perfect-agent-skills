# YouCam AI Agent Skills

Agent skills for beauty, skincare, makeup, hair, fashion, personal appearance,
photo editing, and AI image workflows powered by Perfect Corp.'s YouCam AI
Agent MCP server.

This repository is a marketplace-style package. Host-specific marketplace
metadata points to the reusable plugin under `plugins/youcam-ai-agent/`, where
the shared skills and MCP registration live.

## Installation

### Claude

Add this repository as a Claude plugin marketplace, then install the plugin:

```text
/plugin marketplace add perfect-corp/youcam-ai-agent
/plugin install youcam-ai-agent@youcam-ai-agent
```

Claude will prompt you to connect the bundled MCP server. Complete the OAuth
sign-in in your browser, return to Claude, and retry your request. Never paste
an access token into chat.

If you have installed the marketplace before, refresh it with:

```text
/plugin marketplace update youcam-ai-agent
```

## Capabilities

- Beauty, skincare, makeup, hair, fashion, and personal-style guidance
- Photo-based appearance analysis and personalized recommendations
- Virtual makeup, hairstyle, hair-color, nail, accessory, and fashion try-on
- Image editing, retouching, enhancement, restoration, and transformation
- New image and video generation for supported creative workflows

YouCam AI Agent does not provide clinical dermatology, diagnosis, medication,
or treatment advice.

## Repository structure

```text
.
├── LICENSE
├── .claude-plugin/
│   ├── README.md
│   └── marketplace.json
└── plugins/
    └── youcam-ai-agent/
        ├── .claude-plugin/plugin.json
        ├── .mcp.json
        ├── README.md
        ├── SETUP.md
        └── skills/
            └── youcam-ai-agent/
                ├── SKILL.md
                └── references/mcp-errors.md
```

## Update the plugin

The plugin version is declared in
`plugins/youcam-ai-agent/.claude-plugin/plugin.json`. Bump it for every public
release so existing users receive the update.

## License

Copyright 2026 Perfect Corp.

The plugin configuration, skills, and documentation in this package are
licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE).

This license does not cover the private backend implementation or grant access
to the hosted YouCam service. Service use remains subject to its terms of
service. YouCam trademarks and branding are not licensed under Apache-2.0.
