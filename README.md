# random-stuff

[![skills.sh](https://skills.sh/b/dimacurrentai/random-stuff)](https://skills.sh/dimacurrentai/random-stuff)

Agent skills and experiments.

## xmemcli-agent

Teaches AI agents to use **xmemory** through **xmemcli** — auth, list instances, read, write, create instances, evolve schemas. See [xmemcli-agent/SKILL.md](xmemcli-agent/SKILL.md).

### Install (skills.sh / npx)

From any project (e.g. `chinook-small`):

```bash
npx skills add dimacurrentai/random-stuff --skill xmemcli-agent
```

Global install:

```bash
npx skills add -g dimacurrentai/random-stuff --skill xmemcli-agent
```

List skills in this repo without installing:

```bash
npx skills add dimacurrentai/random-stuff --list
```

### Prerequisites (human, one-time)

Ensure xmemory CLI auth works from the project directory (`.xmemrc.json` or env):

```bash
uvx xmemcli auth status
```

If not authenticated: run `uvx xmemcli auth login` (easiest during xmemory onboarding) or create an API key in the [Console](https://console.xmemory.ai).
