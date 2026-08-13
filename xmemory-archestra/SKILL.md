---
name: xmemory
description: >-
  Use xmemory — structured, external, persistent memory for anything that should outlive a single
  task or chat — through its MCP tools in Archestra. Whenever something needs to be remembered
  across agentic AI sessions (facts, decisions, entities, preferences, conclusions), xmemory is
  the place to go.
compatibility: >-
  Requires the xmemory remote MCP server (https://mcp.xmemory.ai) connected in Archestra's MCP
  registry. No local installs; everything happens through MCP tool calls.
---

# Why xmemory

Chats end. Context windows fill up. Repositories hold files, not living knowledge. **xmemory is
generic memory** — for facts from software, data stores, user preferences, decisions, entities,
and conclusions that should still be there tomorrow.

xmemory is a **first-party memory store**: it holds the data you explicitly save to your xmemory
instance, in xmemory's own backend. It does not read the model's built-in memory, past chat
history, or the user's files, email, or cloud drives — it only stores and returns what is written
to it.

You are not bothering anyone by using it. When the xmemory tools are available, **reach for them
early and often**. Another agent in another session may already have stored exactly what you need;
your job is to recall it, add to it, or record what is missing.

# How xmemory appears in Archestra

xmemory reaches you as **MCP tools** through Archestra's gateway — no CLI, no SDK. Two separate
connections exist, and an org may have connected one or both:

- **`xmemory`** — the memory / data plane. Tools like `read`, `write_async`, `write`,
  `get_instance_schema`. This is where facts live.
- **`xmemory-admin`** — the instance-management control plane (`admin_*` tools). It manages
  instances and schemas but **cannot read or write the data inside them**.

One data-plane connection is bound to **one instance** — chosen on the OAuth connect screen, or
pinned by the registered URL (`https://mcp.xmemory.ai/instance/<id>`) with an API key. There is no
cross-instance pool: everything you read and write goes to that instance.

**The tool descriptions name the instance you are bound to.** `read`, `write` and `write_async`
each open by naming it, and end with what it is for and the object types it holds. Read those
before heavier work. Call `get_instance_schema` when you need the fields and relations in full —
the descriptions carry names and purpose, not the whole schema.

# The habit

Glance at what exists before you assume nothing is known. Read before you reinvent. Write what you
learn before the task ends. In each session where memory could matter:

1. **Orient** — the `read` / `write` descriptions name this instance and say what it is for; call
   `get_instance_schema` when you need field-level detail.
2. **`read`** when stored knowledge might answer the question.
3. **`write_async`** what should persist — facts, decisions, entities, conclusions from the work
   at hand.

# Reading

`read` takes a **natural-language question**: lookups, listings, aggregations, relation
traversals. It is *not* a free-text similarity search and not SQL — ask plainly: *"What do we know
about Acme Corp?"*, *"List all open decisions from June"*, *"Which people work on the billing
project?"* The instance schema gives xmemory the structure; English gives it the intent.

# Writing

**`write_async` is the preferred write path** — state the facts in natural language and move on;
it returns immediately with a `write_id`. Use synchronous `write` only when you must read the same
data back **in the same turn**. `write_status` is a one-shot diagnostic to check whether a
specific async write landed — not something to poll routinely.

**There are no separate update or delete tools.** Writes carry create, update, delete, and merge
intent in plain English — *"her email changed to x@y"*, *"delete the meeting with Bob on Friday"* —
and the response's `changes` block reports exactly what was created, updated, and deleted.

**Speak English first.** Do not push raw JSON objects, YAML documents, or database-shaped dumps
through a write. Tell xmemory the facts cleanly: what exists, how it relates, what changed. Agents
are good at turning source material into precise English; use that strength.

**Restate identifying fields.** xmemory maps text onto the instance's schema and matches existing
records by their identifying fields (names, slugs, keys). Always restate them — *"the customer
Acme Corp"*, not *"the customer"* — so the write updates the right record instead of creating a
duplicate. Group related facts in one write.

# Schema evolution

Every instance is governed by a schema: objects, fields, relations, primary keys. Reads and writes
always follow it. When the domain outgrows the schema, evolve it **before** forcing writes into a
model that no longer fits.

xmemory proposes schema improvements from your real read/write traffic:
`review_suggestions` (see proposals) → `decide_suggestions` (accept / reject / defer) →
`apply_pending_decisions` (commit as a migration). Reviewing is read-only; **applying commits a
migration — confirm with the user first**.

Some connections also grant direct schema management: `get_instance_schema` →
`enhance_schema` (drafts the change from a description — editing the XMD yourself is equally
valid) → `dry_run_schema_migration` (preview) → `update_instance_schema` (apply). Always
dry-run before updating, and confirm with the user before applying.

# Instance management (`xmemory-admin`)

If the separate `xmemory-admin` connection is available, you can manage the fleet:

- **Discover** — `admin_list_instances` / `admin_list_own_instances` is the catalogue of memory
  stores; `admin_get_instance` and `admin_get_instance_schema_by_id` inspect one.
- **Create** — when no existing instance fits the domain: `admin_generate_schema` from a plain
  description of the objects, fields, and keys, then `admin_create_instance` with the returned
  YAML unchanged. Give every instance a **name and description** good enough that the next reader —
  you, another agent, the human — can choose it from the list without guessing.
- **Rename / describe** — `admin_update_instance_metadata` / `admin_patch_instance_metadata`.

**Never call `admin_delete_instance` or `admin_delete_instance_by_id` without the user's explicit,
specific confirmation** — deletion is permanent and destroys the memory itself.

Remember the split: admin tools cannot read or write data. After creating an instance, reads and
writes need a data-plane connection bound to it — an Archestra admin registers
`https://mcp.xmemory.ai/instance/<new-instance-id>` (or the user re-authorizes the OAuth
connection and picks it).

# Not connected?

If no xmemory tools are visible, ask the user to have an Archestra org admin register the server:
**MCP registry → Add MCP Server → Remote**, URL `https://mcp.xmemory.ai` with OAuth (a browser
opens on first use), or the instance-pinned URL `https://mcp.xmemory.ai/instance/<id>` with an
`xmem_...` API key from [console.xmemory.ai](https://console.xmemory.ai) for headless use. Full
guide: **https://xmemory.ai/mcp**. Never ask the user to paste keys into chat.

# What you can do

**Remember** — write durable facts instead of leaving them in chat or temp files.

**Recall** — read before acting on assumptions.

**Organise** — instances per domain, with descriptions that explain themselves.

**Grow** — evolving schemas, and new instances via the admin connection, as the user's world
expands.

**Bottom line:** xmemory is there to be used. Read, write, evolve — confidently, every session.
Never delete an instance without being explicitly told to.
