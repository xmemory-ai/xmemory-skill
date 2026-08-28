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
version-resolution habit; `xmemcli help <topic>` answers most how-tos).

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

(A) Context granted: analyse the user's normal workflow from what your environment
actually has. Paraphrase — never quote private material back verbatim — and write
nothing anywhere. Lead with ONE suggestion tied to a specific observation: "You've been
working on <X> a lot — I'd set up **<Label>**: you'd save things like '<one example
write, adapted to their words>' and later ask '<one example read>'. I'll call it
**<Name>**" (pick the name yourself: 2-4 words, Title case). State it confidently even
at ~80% sure — corrections are welcome signal. Options: "Create <Name>" / "Adjust it" /
"Something else". Too little context to rank? Ask one question — "What should it
remember across sessions: people, deals or projects, technical decisions, competitors,
or your agent's own tasks?" — then send the same one-suggestion message.

(B) Options requested: list every template from the catalogue below as label + one-line
description, plus "Something custom". After the pick, send the rich message from (A) for
that template, ending in "Create <Name>" / "Adjust it".

(C) Custom (chosen at any point): compress what you know into a free-text description of
the entities, their 2-3 key fields, and the links between them. Only promise supported
shapes: text, whole numbers, decimals, yes/no (a field may have fixed allowed values);
dates are text in ISO format; repeated values are a separate linked record, not a list
field. Present a 3-6 line plain-English summary (entities, key fields, links — never raw
YAML), ending in "Create <Name>" / "Change something".

## TURN 3 — create, set up, close

Choosing "Create <Name>" was the user's approval — do not re-confirm. Then:

1. Compose the schema description: for a template, write a thorough free-text
   description from its catalogue entry below (what it is, its object types, the example
   writes and reads); for custom, use what the user told you.
2. Get the schema: `xmemcli xmd generate "<description>" -o schema.yml`, or author the
   XMD yourself when you already know the shape — generation is a convenience, not a
   requirement. If generation fails, retry once with a restated description; if it fails
   again, write the XMD yourself or suggest the console's guided setup.
3. Create: `xmemcli instance create --name "<Name>" --schema-file schema.yml` (add
   `--description` when the user gave one). The CLI resolves the cluster automatically
   when there is exactly one.
4. Set up this environment: the create output includes the connect instructions for
   agent surfaces — follow the steps for this environment now, with the user's
   confirmation for anything that writes files. For the committable, instance-specific
   install (the files a project commits so the whole team gets this instance), run
   `xmemcli instance setup <instance-id> --format project` and apply the printed
   fragments exactly as their merge notes say — merge, never overwrite existing settings
   files.
5. Close in one message: what exists now (name and id); what was installed; the hook:
   "It gets more accurate the more you use it — just talk naturally and ask it to
   remember things; you can reshape the structure any time." Optionally suggest 2-3
   first things to say, adapted from the template's example writes — suggestions only,
   you write nothing. Never suggest bulk-importing.

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
- Instance-limit error on create: say plainly that the plan's limit is reached, point at
  the console for upgrading, stop. Never delete anything to make room unless explicitly
  told to.
- More than one cluster visible: ask which one, by name, before creating (set
  `XMEM_CLUSTER_ID` or pass the CLI's cluster flag).
- The user rejects the custom summary twice: ask what's wrong in one question, then
  regenerate from a description amended with their words.
- The user is in a hurry: accept a one-line description, pick the best single template
  or generate directly, and go straight to the "Create <Name>" message.
- The context analysis was declined: never mention it again.

## Template catalogue

### Personal CRM (id: personal_crm)
- What it is: Remembers the people in your life — friends, family, and professional contacts — and everything you learn about them over time.
- Who it's for: Anyone who wants to stay close to friends, family, and a professional network without dropping threads.
- Example things you'd save: Met Anna Petrova at the fintech meetup — she just moved to Lisbon and started a new job at a payments startup. | Marco's birthday is June 12; he's into trail running and doesn't drink alcohol. | Had a call with John from the gym — he offered to intro me to his sister who runs a design studio; follow up next week.
- Example questions you'd ask: Who do I know in Lisbon? | Whose birthdays are coming up, and what gift ideas have I noted? | What did Marco and I talk about last time, and what did I promise to follow up on?
- Object types: Affiliation, Connection, ContactMethod, ImportantDate, Interaction, LifeEvent, Note, Organization, Person, Preference, Topic

### Sales pipeline (id: sales_pipeline)
- What it is: A B2B sales CRM — accounts and the people at them, deals and their buying committees, activities, quotes, and tasks.
- Who it's for: Founders, account executives, and small sales teams tracking B2B deals without a heavyweight CRM.
- Example things you'd save: Logged a discovery call with Acme Corp — their VP of Ops Sarah is the economic buyer; budget around $50k, decision by end of Q3. | Moved the Globex deal to negotiation and sent a quote for 200 seats at $12 per seat per month. | Task: send the security questionnaire back to Initech by Friday.
- Example questions you'd ask: Which deals are closing this quarter, and what's blocking each one? | Who's our champion at Acme, and when did we last talk to them? | What open tasks are overdue across the pipeline?
- Object types: Account, Activity, Campaign, Contact, Deal, DealRole, Event, LineItem, Product, Quote, Task, Territory

### Investor pipeline (id: investor_pipeline)
- What it is: Venture deal flow and portfolio memory — funds and LPs, companies, people, deals with diligence signals, meetings, and reference checks.
- Who it's for: Venture investors and angels managing deal flow, diligence, and portfolio updates.
- Example things you'd save: Met with Nebula AI's founders — impressive traction, $80k MRR growing 15% month over month; raising a $4M seed at a $20M cap. | Reference check on their CTO came back strong — two former colleagues independently called her a top-1% operator. | Passed on Quantify — market too small; keep warm for their next round.
- Example questions you'd ask: Which active deals are in diligence, and what signals are still missing? | What did we hear in reference checks for Nebula AI? | Which portfolio companies haven't sent an update in over a quarter?
- Object types: Coinvestment, Company, Connection, Deal, Firm, Fund, Meeting, Person, PersonRole, ReferenceCheck, SignalScore, Task, Update

### Engineering memory (id: engineering_memory)
- What it is: Durable codebase knowledge — components and architecture, tech stack, ownership, conventions, decisions, known issues, and runbook commands.
- Who it's for: Engineering teams, and developers working with AI-assisted tools, who keep re-explaining the same codebase context.
- Example things you'd save: Decision: we use dbmate for migrations, not alembic — versions are date-prefixed and CI applies them on merge. | The payments service is owned by the platform team; restart it with `make payments-restart`. | Known issue: staging drops websocket connections after 60 seconds idle — workaround is a client-side ping every 30 seconds.
- Example questions you'd ask: Who owns the payments service, and how do I restart it? | Why did we pick Postgres over a key-value store for sessions? | What known issues do we have around websockets?
- Object types: Command, Component, Convention, Decision, GlossaryTerm, Interface, KeyFile, KnownIssue, Person, Technology

### Competitor tracking (id: competitor_intel)
- What it is: Competitive intelligence — competitors, their products and pricing, a feature-by-feature capability matrix, win/loss notes, news signals, people, and funding.
- Who it's for: Product managers, founders, and marketers who track a competitive landscape.
- Example things you'd save: Rivalsoft launched usage-based pricing today — starts at $99 a month plus $0.02 per API call. | We lost the Acme deal to Rivalsoft on SSO and audit logs; price wasn't the issue. | Rivalsoft raised a $30M Series B led by a top-tier fund.
- Example questions you'd ask: How does our pricing compare to Rivalsoft's latest plans? | What were the last three win/loss reasons against Rivalsoft? | What changed in the competitive landscape this month?
- Object types: Benchmark, CapabilityAssessment, Competitor, Feature, Fund, Insight, InvestmentRound, KeyPerson, NewsEvent, PricingPlan, Product, Resource, WinLoss

### Agent task memory (id: agent_task_memory)
- What it is: Working memory for AI coding agents, keyed by a stable task id — plans and steps, requirements, decisions, review pushbacks, issues, findings, and verification checks.
- Who it's for: Developers running AI coding agents that need continuity across sessions, compactions, and restarts.
- Example things you'd save: Task AUTH-42: plan step 3 done — refresh-token rotation implemented; next up is revoking sessions on password change. | Decision on AUTH-42: store sessions in Postgres, not an in-memory cache — we need transactional revocation. | Reviewer pushback on AUTH-42: don't log token prefixes; agreed and removed.
- Example questions you'd ask: Where did task AUTH-42 leave off, and what's the next action? | What decisions and review pushbacks are recorded for AUTH-42? | Which verification checks are still pending on this task?
- Object types: Check, Decision, Finding, Issue, Plan, Pushback, Requirement, Session, Step, Task, Touchpoint
