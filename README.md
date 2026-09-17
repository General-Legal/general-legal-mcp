# General Legal MCP

[General Legal](https://general.legal) lets you open legal matters, submit contracts, exchange messages with attorneys, and download reviewed documents through an AI assistant or autonomous agent.

Choose the matters connection that matches your authentication method:

| Service | Endpoint | Authentication | Guide |
| --- | --- | --- | --- |
| General Legal | `https://mcp.general.legal/mcp` | Clerk OAuth; sign in and select an organization | [OAuth guide](docs/oauth.md) |
| General Legal for Agents | `https://agents-mcp.general.legal/mcp` | API key; anonymous signup/recovery when enabled | [Agents guide](docs/agents.md) |

Both expose the same eight matter/document tools. The agents service also supports six signup/recovery tools and their one-time API-key results. The [public REST API](https://api.general.legal/api/v1/docs) remains available for direct integrations.

## Install the plugin

```bash
npx plugins add General-Legal/general-legal-mcp
```

The plugin connects the OAuth matters service and the separate incorporation service, with their usage skills. For an API-key connection, follow the agents guide and configure `general-legal-agents` explicitly. Install the matters connection you intend to use to avoid duplicate tools.

| Path | Purpose |
| --- | --- |
| `mcp.json` | OAuth matters and incorporation connections |
| `docs/oauth.md` | OAuth connection and legal-matter workflows |
| `docs/agents.md` | API-key access, signup/recovery, and legal-matter workflows |
| `skills/legal-matters/` | Instructions for the installed OAuth matters connection |
| `skills/company-formation/` | Incorporation instructions |

## Support

- Email: support@general.legal
- Website: [general.legal](https://general.legal)
