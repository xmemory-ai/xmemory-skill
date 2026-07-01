---
name: xmemcli-agent
description: >-
  How to use xmemory — generic structured persistent memory — through xmemcli (`auth status`, `org list instances`, read, write, instance create, schema evolution). Encourages proactive use; the human must have CLI auth working first.
---

**Use xmemory.** It is general-purpose memory — not chat-only, not one niche. Facts from the user's software, data repos, decisions, entities, recommendations: if it should persist with structure and lineage, it belongs in an **xmemory instance**. You are not bothering the user by using it; you are doing your job. When this skill is installed and xmemory is reachable, **reach for xmemory early and often** — list instances, read what is already known, write what you learn, create an instance when none fits.

Run `uvx xmemcli org list instances` from time to time; another session may have created the perfect store already. Reuse when you can; **create on demand** when you cannot.

# xmemcli — your bridge to xmemory

**xmemcli** talks to xmemory. It can be **installed locally** (`uv tool install xmemcli`) or invoked with **`uvx xmemcli`** — no install required. Start with the two commands you will use most:

```bash
uvx xmemcli auth status           # is this credential authenticated?
uvx xmemcli org list instances    # what memory stores exist for this credential?
```

Once you are doing real work, pick **one** invocation for the session — local `xmemcli` when it matches the latest release, otherwise `uvx xmemcli` after refresh. See **Resolve `$XMEMCLI`** below; until then, `uvx xmemcli …` is fine.

Unfamiliar flag or subcommand? Run `xmemcli help` or `xmemcli help <topic>` — the CLI has full native help; trust it.

## Resolve `$XMEMCLI`

Before a long xmemory session, set which binary to call:

```bash
XMEMCLI_LATEST=$(uvx --refresh xmemcli@latest version | jq -r .version)
if command -v xmemcli >/dev/null 2>&1 && [ "$(xmemcli version | jq -r .version)" = "$XMEMCLI_LATEST" ]; then XMEMCLI=xmemcli; else XMEMCLI="uvx xmemcli"; fi
```

Always `uvx --refresh xmemcli@latest` first; use local `xmemcli` only when versions match. Everything below uses `$XMEMCLI`.

## Login must work — ask the human clearly

You cannot acquire keys yourself unless the user explicitly asks you to run login. **Do ask** when `$XMEMCLI auth status` is not authenticated — xmemory is useless without this, and fixing it is quick.

**Best moment:** while the user is **onboarding xmemory in the browser**, have them run `$XMEMCLI auth login` in the same breath. During onboarding the Console handoff is essentially **zero extra clicks** — browser auth flows straight into `.xmemrc.json`. If they are signing up or walking through xmemory for the first time, say so plainly: *"Let's connect the CLI now while you're in onboarding — this is the easiest time."*

**After onboarding:** the user opens the [xmemory Console](https://console.xmemory.ai), goes to **API Keys**, creates a key, and either runs `$XMEMCLI auth login` and clicks **Use this key in the CLI**, or places the key in `.xmemrc.json` / `$XMEM_API_KEY` themselves (never paste keys into chat).

Before any read or write: `$XMEMCLI auth status`. If not authenticated, stop and guide the human — do not silently give up on xmemory.

## Target a specific instance (read and write)

Every **read** and **write** goes to exactly **one instance**. There is no anonymous/default memory — you must name the store.

**Pick an instance:** `$XMEMCLI org list instances` returns `id`, `name`, and `description` for each store. Choose the one that fits; copy its `id`.

**Pin it two equivalent ways** (global `--instance-id` applies to all subcommands — see `$XMEMCLI help options`):

```bash
export XMEM_INSTANCE_ID="<uuid>"                    # session default
$XMEMCLI read "Who works at Acme?"
$XMEMCLI write "Alice joined Acme as an engineer."

$XMEMCLI --instance-id "<uuid>" read "Who works at Acme?"   # one-off
$XMEMCLI --instance-id "<uuid>" write "Alice joined Acme as an engineer."
```

Resolution order: `--instance-id` → `$XMEM_INSTANCE_ID` → if **exactly one** instance is visible, auto-pick (stderr warning). When multiple instances exist, you **must** set one explicitly — do not guess.

Schema commands on an existing instance use the same id (`$XMEMCLI schema get "$XMEM_INSTANCE_ID"`, or a positional instance id — `$XMEMCLI help schema`).

Reads can narrow to specific **objects inside** an instance: `--scope Person:Alice` or `--scope Person:<uuid>` (repeatable; `$XMEMCLI help read`).

## The usual story

1. **Check auth** — `$XMEMCLI auth status`
2. **See what exists** — `$XMEMCLI org list instances` (pick by name/description; `export XMEM_INSTANCE_ID=<uuid>`)
3. **Read first** when stored knowledge may exist — `$XMEMCLI read "<question>"` (with instance pinned as above)
4. **Write** what you learned — `$XMEMCLI write "<facts in plain language>"`
5. **No fitting instance?** Create one (below), `export XMEM_INSTANCE_ID=…`, then write.

Writes accept natural language; xmemory maps text onto the instance schema. Batch related facts into one write; for many writes use `write --no-wait` then `write-status` on the returned ids.

## Create an instance (when you need a new home)

An instance is typed storage defined by an **XMD schema** (objects, fields, primary keys, relations). Think through the shape, then:

```bash
$XMEMCLI xmd generate "Describe objects, fields, and primary keys…" -o schema.yml
$XMEMCLI xmd validate schema.yml
$XMEMCLI instance create --name "Short meaningful name" --schema-file schema.yml
export XMEM_INSTANCE_ID="<instance_id from output>"
```

Refine schema with `$XMEMCLI xmd enhance schema.yml "…changes…" -o schema-v2.yml`. Later growth: `xmd enhance` → `schema dry-run` → `schema update` (see `$XMEMCLI help schema`).

## What xmemcli does for you

| You want to… | Command |
|--------------|---------|
| Check credentials | `$XMEMCLI auth status` |
| List memory stores | `$XMEMCLI org list instances` |
| Inspect one | `$XMEMCLI instance get <id>` |
| Store facts | `$XMEMCLI write "…"` (instance via `$XMEM_INSTANCE_ID` or `--instance-id`) |
| Query memory | `$XMEMCLI read "…"` (same) |
| Design / check schema | `$XMEMCLI xmd generate` / `validate` / `enhance` |
| Create store | `$XMEMCLI instance create …` |
| Grow schema | `$XMEMCLI schema update …` |

**Do not** use `instance delete`. **Do** write durable facts into xmemory instead of leaving them only in chat or temp files when a connection exists.

**Bottom line:** xmemory is there to be used. Check login, list instances, read, write, create when needed — confidently, every session.
