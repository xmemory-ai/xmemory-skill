# xmemory-skill

Agent skills for using **xmemory** — through **xmemcli** (generic) or through the
**xmemory MCP server** in [Archestra](https://archestra.ai).

## xmemcli-agent

Teaches AI agents to use **xmemory** through **xmemcli** — auth, list instances, read, write, create instances, evolve schemas. See [xmemcli-agent/SKILL.md](xmemcli-agent/SKILL.md).

### Install (skills.sh / npx)

From any project (e.g. `chinook-small`):

```bash
npx skills add xmemory-ai/xmemory-skill
```

### Prerequisites (human, one-time)

Ensure xmemory CLI auth works from the project directory (`.xmemrc.json` or env):

```bash
uvx xmemcli auth status
```

If not authenticated: run `uvx xmemcli auth login` (easiest during xmemory onboarding) or create an API key in the [Console](https://console.xmemory.ai).

## xmemory-archestra

Teaches agents running on the [Archestra platform](https://archestra.ai) to use xmemory through
its **remote MCP server** (`https://mcp.xmemory.ai`) — reading, writing, schema evolution, and the
separate `xmemory-admin` instance-management connection. See
[xmemory-archestra/SKILL.md](xmemory-archestra/SKILL.md).

### Install (Archestra GitHub import)

In Archestra: **Skills → Import from GitHub** → enter `xmemory-ai/xmemory-skill`, select
`xmemory-archestra`, and optionally enable **Keep in sync**. The skill's name `xmemory` becomes
the `/xmemory` slash command when slash commands are enabled.

### Prerequisites (org admin, one-time)

An Archestra org admin registers the xmemory remote MCP server in the private MCP registry:
URL `https://mcp.xmemory.ai` with OAuth, or `https://mcp.xmemory.ai/instance/<id>` with an API
key from the [Console](https://console.xmemory.ai). Full guide: <https://xmemory.ai/mcp>.

## License

© xmemory Inc. All rights reserved. This skill is proprietary; permission is granted to install
it (e.g. via the `skills` CLI) and to redistribute it solely to distribute it through agent-skill
registries, and to use it solely to connect to the xmemory service. See [`LICENSE`](./LICENSE).
Use of xmemory is governed by the [Terms & Conditions](https://xmemory.ai/terms-and-conditions.html).
