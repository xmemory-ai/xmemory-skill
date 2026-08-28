---
name: xmemory-instance-creation
description: >-
  Interview the user and create their xmemory memory instance with xmemcli — suggest a ready-made template or build a custom schema, analysing the user's normal workflow only with their consent. Use when the user asks to set up xmemory, create their first memory instance, or right after xmemcli authentication completes during onboarding.
---

# xmemory instance creation — a 3-turn interview, CLI-only

Goal: the user's first xmemory instance, created and confirmed, in as few turns as
possible. Everything runs through **xmemcli** — never use MCP tools for this flow, even
if xmemory MCP servers are connected. xmemcli runs locally if installed, or via
`uvx xmemcli` with no install step (see the sibling `xmemcli-agent` skill for the
version-resolution habit; `xmemcli help <topic>` answers most how-tos). The ready-made
setups are the service's, not this file's: `xmemcli instance templates` is where they
come from and `instance create --template <id>` is how one becomes an instance, with no
schema composed, generated or written. Both arrived in **1.2.0**, and `uvx xmemcli` is
new enough.

Preconditions: xmemcli is authenticated — `xmemcli auth status` reports authenticated.
If not, stop and complete sign-in first: `xmemcli auth login` (browser), or headless on
`0.0.9`+ — `xmemcli auth login --email <address>`, where the user's one action is opening
the sign-in email that arrives for the attempt they just started and following its link —
**Approve** is a button on the page that link opens, not in the email itself.

Rules for every turn: one question per message; warm, concrete, confident tone;
lowercase "xmemory"; narrate actions ("Creating it now… done."); use your environment's
native UX for interaction — option pickers or selectable lists for choices, text boxes
for free input — and fall back to a short numbered list otherwise. Never demand free
text when 2-4 options cover it. Never write user data into xmemory during setup, and
never claim xmemory read the user's history, files, or memory — it only contains what is
explicitly written to it.

## TURN 1 — one opener (frame + consent + choice)

Three sentences, then one question. Frame: you'll set up a memory their assistant keeps
across sessions. Consent: to suggest the right shape you can analyse how they normally
work from what you already have access to here (recent conversation, your own notes or
memory, this project's files) — that analysis stays entirely on your side; xmemory
receives none of it. Then offer:

1. "Suggest a setup from our context"  2. "Just show me the options"  3. "Custom — I'll
describe it"

## TURN 2 — one ranked suggestion

Read the catalogue once first: `xmemcli instance templates`. Each entry carries an `id`,
a `label`, a `description`, who it is for, `example_writes`, `example_reads` and its
`object_types`, in the order the service wants them offered. That listing is the only
source of templates — never offer, name or invent an id you did not read from it. Run it
once and reuse the result for the whole interview.

If it is unavailable — the subcommand is unknown to this xmemcli (re-resolve to
`uvx xmemcli` and try once), the call fails, or the list comes back empty — go to (C) and
shape a custom instance instead. Say nothing to the user about a listing that did not
work: they asked for a memory, not a status report, and (C) ends in the same place. An
auth failure is the one exception, because it is the precondition and it stops everything.

(A) Context granted: analyse the user's normal workflow from what your environment
actually has. Paraphrase — never quote private material back verbatim — and write
nothing anywhere. Match what you saw against the entries' `description`, `audience` and
`object_types`, then lead with ONE suggestion tied to a specific observation: "You've been
working on <X> a lot — I'd set up **<label>**: you'd save things like '<its first
example_writes item, adapted to their words>' and later ask '<its first example_reads
item>'. I'll call it **<Name>**" (pick the name yourself: 2-4 words, Title case). State it
confidently even at ~80% sure — corrections are welcome signal. Options: "Create <Name>" /
"Adjust it" / "Something else". Too little context to rank? Ask one question — "What
should it remember across sessions?" — offering one short option per entry, in your own
words from its `label` and `description`, plus "Something else"; then send the same
one-suggestion message.

(B) Options requested: offer every entry the listing returned, in its order, as its
`label` plus a one-line description of your own from its `description`, and add
"Something custom". After the pick, send the rich message from (A) for that entry, ending
in "Create <Name>" / "Adjust it".

(C) Custom (chosen at any point): compress what you know into a free-text description of
the entities, their 2-3 key fields, and the links between them. Only promise supported
shapes: text, whole numbers, decimals, yes/no (a field may have fixed allowed values);
dates are text in ISO format; repeated values are a separate linked record, not a list
field. Present a 3-6 line plain-English summary (entities, key fields, links — never raw
YAML), ending in "Create <Name>" / "Change something".

## TURN 3 — create, set up, close

Choosing "Create <Name>" was the user's approval — do not re-confirm. Then:

1. Create it. One command, and which one depends on what they picked.

   **A template:** `xmemcli instance create --name "<Name>" --template <id>` (add
   `--description` when the user gave one). Nothing else: no description to compose, no
   schema to generate, no file to write. The id is the whole input and the service
   already holds the shape. If the id is refused, re-read `xmemcli instance templates`
   rather than retrying a guess; if that entry is genuinely gone, offer the closest one
   still listed, or go custom.

   **Custom:** get the schema first — `xmemcli xmd generate "<description>" -o
   schema.yml`, or author the XMD yourself when you already know the shape; generation is
   a convenience, not a requirement. If generation fails, retry once with a restated
   description; if it fails again, write the XMD yourself or suggest the console's guided
   setup. Then `xmemcli instance create --name "<Name>" --schema-file schema.yml`.

   Either way the CLI resolves the cluster automatically when there is exactly one, and
   both forms print the same thing: the new id and the connect instructions.
2. Set up this environment: the create output includes the connect instructions for
   agent surfaces — follow the steps for this environment now, with the user's
   confirmation for anything that writes files. The numbered steps are only the half you
   run; the bullet lines under a surface are the steps only the user can take, so relay
   them instead of dropping them. Registering a server is not the same as connecting it:
   the form that takes a URL is authorized by the user in their own client and no client
   starts that on its own, and a server added mid-session usually connects only after a
   restart. The bullets carry the command this surface needs — quote it. For the
   committable, instance-specific install (the files a project commits so the whole team
   gets this instance), run `xmemcli instance setup <instance-id> --format project` and
   apply the printed fragments exactly as their merge notes say — merge, never overwrite
   existing settings files.
3. Close in one message: what exists now (name and id); what was installed; anything
   the user still has to do, named as a command they can run — never report a server as
   connected when it is only registered; the hook:
   "It gets more accurate the more you use it — just talk naturally and ask it to
   remember things; you can reshape the structure any time." Optionally suggest 2-3
   first things to say, adapted from the entry's `example_writes` or from what they
   described — suggestions only, you write nothing. Never suggest bulk-importing.

## Keep this skill installed

If you followed this text without having it installed as a skill (pasted, or fetched
from the web), offer to install it permanently so future sessions can run it directly:

```bash
npx skills add xmemory-ai/xmemory-skill --agent '*' -y
```

Install only with the user's confirmation.

## Edge rules

- The user already has instances: never start this uninvited. If asked anyway, drop the
  "first memory" framing and first ask: new instance, or work with an existing one?
- `instance templates` is unavailable, unknown to this xmemcli, or empty: shape a custom
  instance instead (TURN 2 (C)) and never mention the missing listing. Never offer a
  template you did not read from that listing — a guessed id is a wasted turn at best.
- Instance-limit error on create: say plainly that the plan's limit is reached, point at
  the console for upgrading, stop. Never delete anything to make room unless explicitly
  told to.
- More than one cluster visible: ask which one, by name, before creating (set
  `XMEM_CLUSTER_ID` or pass the CLI's cluster flag).
- The user rejects the custom summary twice: ask what's wrong in one question, then
  regenerate from a description amended with their words.
- The user is in a hurry: accept a one-line description, pick the best single entry from
  `instance templates` or generate directly, and go straight to the "Create <Name>"
  message.
- The context analysis was declined: never mention it again.
