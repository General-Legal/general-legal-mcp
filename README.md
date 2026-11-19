# General Legal MCP

[General Legal](https://general.legal) lets you open legal matters, submit contracts, exchange messages with attorneys, and download reviewed documents through an AI assistant or autonomous agent.

Choose the matters connection that matches your authentication method:

| Service | Endpoint | Authentication | Guide |
| --- | --- | --- | --- |
| General Legal | `https://mcp.general.legal/mcp` | Clerk OAuth; sign in or sign up during connection, then select an organization | [OAuth guide](docs/oauth.md) |
| General Legal for Agents | `https://agents-mcp.general.legal/mcp` | API key; anonymous signup/recovery when enabled | [Agents guide](docs/agents.md) |

New clients can complete signup and accept the engagement agreement during connection, then return to their MCP client. Existing clients use their current account and organization. See the [OAuth guide](docs/oauth.md) for the signup flow and recovery steps for a previously stuck connection.

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
| `skills/legal-ops-in-a-box/` | Map legal intake and approvals, draft automations, and propose matter handoffs |
| `skills/company-formation/` | Incorporation instructions |

## Legal Ops in a Box

With the updated plugin loaded, select `legal-ops-in-a-box` if your host lists skills, or ask it to use Legal Ops in a Box to design your intake workflow. It uses the host's available capabilities and refers to `legal-matters` for matter handling. Existing installs need to refresh from a version that includes the skill. Connecting just the MCP endpoint does not install plugin skills.

Try the [synthetic demo](docs/legal-ops-demo.md) from a checkout. It includes a chat-only path with no credentials or live routing.

## Support

- Email: support@general.legal
- Website: [general.legal](https://general.legal)
