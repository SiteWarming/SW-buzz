# Buzz Assessment

- **Assessment date:** July 22, 2026
- **Assessment scope:** Whether Buzz is suitable as an internal replacement for Slack
- **Repository snapshot:** Buzz Desktop 0.4.23 source at commit `acfbb1b`

## Executive Summary

Buzz is a real, substantial open-source collaboration system, not a mock-up or a
thin Slack clone. Its strongest idea is that people and AI agents use the same
identity, channel, event, and audit primitives. This gives agents much better
access to organizational context and makes their Buzz-native actions easier to
attribute than actions performed through conventional shared bot accounts.

Buzz is not currently a suitable organization-wide replacement for Slack. It is
pre-1.0, operationally young, and still has important gaps around identity
recovery, mobile maturity, migration, integrations, administration, permission
granularity, workflow approvals, and agent security. The source is moving
quickly enough that some status documentation already trails implementation,
which further argues against treating the present release as stable
infrastructure.

The recommended course is to keep Slack as the organization's primary
communication system and run a limited Buzz pilot for one internal project.
Buzz should only replace Slack after the pilot passes explicit reliability,
security, recovery, administration, and adoption gates.

## Recommendation

**Current decision: Pilot; do not migrate the organization.**

Buzz is worth piloting when all of the following are true:

- AI agents are expected to participate in daily project work, not merely answer
  occasional questions.
- Pilot participants are technically comfortable with early-stage software.
- The organization can operate and secure Postgres, Redis, object storage, TLS,
  backups, and the Buzz relay.
- Agents can be isolated from production credentials and sensitive systems.
- Slack can remain available as the communication fallback during evaluation.

Buzz is not yet a reasonable primary workspace when the organization depends
heavily on mature mobile notifications, a large Slack application ecosystem,
centralized identity lifecycle management, nontechnical guest onboarding,
formal retention controls, or compliance tooling.

## What Buzz Actually Is

Buzz is a self-hostable workspace built around a Nostr event relay. Messages,
reactions, workflow activity, profiles, Git events, and many other platform
operations are represented as cryptographically signed events. The shipped
system includes:

- A Rust relay backed by Postgres, Redis, and S3-compatible object storage
- A Tauri and React desktop application
- A Flutter mobile application under active development
- Channels, threads, direct messages, forums, canvases, search, media, and voice
- A workflow engine with message, reaction, schedule, and webhook triggers
- An ACP harness for Goose, Codex, Claude Code, and other compatible agents
- A minimal model-agnostic Buzz agent
- Agent-facing CLI and MCP tools
- Git smart HTTP hosting and signed Git-related events
- A hash-chain audit log

The project is intentionally broader than Slack. Its stated goal is to combine
communication, agents, automation, project memory, and parts of the software
forge into one workspace.

Relevant project documentation:

- [README](README.md)
- [Architecture](ARCHITECTURE.md)
- [Security policy](SECURITY.md)
- [Product vision](VISION.md)
- [Sovereignty vision](VISION_SOVEREIGN.md)
- [Agent vision](VISION_AGENT.md)
- [Project and Git vision](VISION_PROJECTS.md)

## Claim Assessment

### Buzz is a working product

**Assessment: True.**

The code supports the launch announcement's central feature claims. The relay,
desktop client, search, media handling, workflow engine, agent harness, agent
CLI, audit implementation, and Git components are present as production code
with unit and integration test infrastructure. Buzz should not be dismissed as
vaporware.

This does not establish production maturity. Commit volume, lines of code, and
test-file counts demonstrate activity and scope, not reliability. The project
was built unusually quickly and with extensive agent assistance. That velocity
is impressive, but it also increases the importance of integration testing,
operational testing, and cautious rollout.

### Buzz can replace Slack and GitHub today

**Assessment: Not for a typical organization.**

Buzz can reproduce important portions of team chat and Git workflows, but its
own documentation calls it unfinished and pre-1.0. Known limitations include
incomplete workflow approval gates and partially stubbed workflow actions. The
mobile clients and their notification delivery paths are much younger than the
desktop and relay surfaces.

Buzz should be evaluated as a new integrated workspace, not as a feature-for-
feature Slack replacement. A migration decision must account for every Slack
workflow, application, guest process, notification path, retention requirement,
and administrative procedure currently in use.

### Agents are first-class members

**Assessment: Technically meaningful, with marketing simplification.**

Humans and agents both receive Nostr keypairs, profiles, channel memberships,
and signed event histories. An agent is not forced through a separate shared
bot identity or a narrow slash-command interface. This is a genuine
architectural difference from many conventional chat integrations.

"First-class member" does not mean that an agent is equivalent to a human in
judgment, accountability, or safety. It means the agent uses the same platform
identity and communication primitives.

### Everything is signed and attributable

**Assessment: Partly true.**

Buzz-native events are signed. A valid signature demonstrates that a particular
private key signed particular event bytes. It does not prove that:

- The action was correct
- A human explicitly approved it
- The model was not manipulated by prompt injection
- The private key was not stolen or misused
- A local operating-system side effect occurred exactly as reported

Signing improves provenance. It does not by itself provide authorization,
confidentiality, correctness, sandboxing, or safe autonomy.

### The audit record is permanent or immutable

**Assessment: Overstated.**

Buzz's audit log is a keyless SHA-256 hash chain. The project's security
documentation accurately describes it as tamper-evident rather than
tamper-resistant. It can reveal accidental corruption or an isolated row edit,
but an attacker with database write access can modify data and recompute the
chain. This is useful audit infrastructure, but it is not an immutable ledger.

### Buzz is decentralized and self-sovereign

**Assessment: Partly true.**

Apache 2.0 licensing, self-hosting, portable Nostr keys, and an open wire format
provide genuine exit rights. An organization can operate a relay without
placing its workspace on Block's infrastructure.

Each workspace still relies on an authoritative relay operator. Portable
identity does not automatically provide mature federation or seamless migration
of messages, media, repositories, search state, workflows, audit history, and
operational configuration. The current system provides source and identity
portability more convincingly than it provides turnkey workspace portability.

### Self-hosting means messages are private

**Assessment: Only relative to Block.**

Running the relay internally removes Block from the server path. It does not
hide content from the relay operator or database administrators. The documented
model uses TLS in transit and storage-layer encryption at rest; general end-to-
end encryption is not the current workspace model. Cloud-hosted model providers
may also receive agent prompts, retrieved channel context, and tool results.

The correct statement is: self-hosting lets the organization choose and control
the trusted server operator. It does not eliminate the server trust boundary.

### Buzz agents do not require API keys

**Assessment: Misleading.**

Buzz identity can remove the need for a conventional Slack bot token inside the
workspace, but it does not eliminate model or tool credentials. Existing Claude
Code or Codex installations may reuse their authenticated local setup. The
bundled Buzz agent requires provider credentials for Anthropic, OpenAI, or
Databricks, unless it is pointed at an appropriate locally hosted endpoint such
as Ollama, llama.cpp, or vLLM.

### Buzz can run 100% locally

**Assessment: Possible under a narrow configuration.**

A local relay, local desktop application, local agent process, local model, and
local tools can form a local deployment. This stops being fully local when it
uses a cloud model, remote relay, hosted MCP service, external search provider,
or any other network tool. A process running on a laptop is not necessarily a
local-inference or private-data system.

Claims that a local model produces "the same quality" as a cloud model require
repeatable evaluations across representative work. A successful demonstration
of one web-research task is not sufficient evidence.

## The Central Architectural Tradeoff

Buzz's primary benefit and primary risk come from the same design decision: it
removes boundaries between conversation, code, automation, agents, and project
memory.

For agent effectiveness, this is attractive. Agents lose less context between
Slack, GitHub, CI, tickets, and isolated model sessions. A channel can preserve
the discussion, implementation, review, and operational history of a project in
one searchable place.

Those former seams also acted as security and failure boundaries. Combining the
systems means an untrusted channel message, repository file, pasted webpage, or
workflow result may enter the context of an agent that has filesystem, shell,
Git, or network tools. More context can improve performance while simultaneously
increasing the prompt-injection and data-exfiltration surface.

Block's own Goose red-team exercise demonstrated that an invisible prompt
injection could lead to compromise of an employee laptop. The report concludes
that current coding agents cannot reliably separate trusted instructions from
untrusted context. Buzz does not make that underlying model limitation
disappear.

See: [How We Red-Teamed Our Own AI Agent](https://engineering.block.xyz/blog/how-we-red-teamed-our-own-ai-agent-)

## Slack-Replacement Assessment

### Communication reliability

Slack is mature infrastructure with well-understood desktop, web, mobile, push,
offline, and notification behavior. Buzz's core chat implementation is real,
but it has not accumulated comparable operational history.

Before migration, the organization must validate:

- Message delivery during relay restarts and network interruptions
- Subscription recovery and unread-state correctness
- Desktop and mobile notification timeliness
- Thread and search behavior at the organization's expected data volume
- Media upload, download, and retention behavior
- Voice reliability, if huddles are required
- Upgrade and rollback behavior

### Identity lifecycle

Buzz's portable keys are powerful but place more responsibility on the user and
operator. Losing a private key can mean losing the identity, and the sovereignty
documentation explicitly notes that there is no conventional forgotten-password
recovery flow.

An organizational deployment needs documented procedures for:

- Identity creation and verification
- Secure backup and multi-device enrollment
- Lost or stolen devices
- Key rotation or replacement
- Employee offboarding
- Recovery when a user loses all key material
- Separating personal Nostr identities from organization-managed identities

These procedures must be tested, not merely written.

### Authentication and authorization

Channel membership is Buzz's principal content gate. Private channels are
hidden from nonmembers, and authenticated nonmembers cannot read their events.
That is a clear and useful boundary.

The same model is relatively coarse for powerful agents. Buzz's security policy
states that there is no separate general ACL or capability taxonomy: within a
channel, membership grants the principal read and write access. Local tool
permissions, repository permissions, cloud credentials, and operating-system
access therefore require separate controls.

An agent's Buzz membership must never be treated as the complete authorization
model for everything the agent can do.

### Migration and historical context

No documented Slack workspace-history importer was located during this
assessment. This must be confirmed again against the release selected for a
pilot because Buzz is changing quickly.

Without an importer, the organization must choose among:

- Keeping Slack available as a historical archive
- Exporting Slack data and building an access or import path
- Migrating only selected channels
- Starting Buzz without prior conversational history

Starting without history weakens one of Buzz's principal advantages: providing
agents with organizational context.

### Integrations and workflows

Buzz supports internal workflows and inbound/outbound webhooks. It does not yet
offer the breadth and maturity of the Slack application ecosystem.

Before a pilot, inventory all current Slack dependencies, including:

- Source control and pull-request notifications
- CI and deployment notifications
- Incident response and paging
- Ticketing and project management
- Calendar and meeting tools
- Monitoring and observability
- Customer support or sales systems
- Forms, polls, approvals, and custom bots
- Identity provisioning and guest management

Each dependency needs a Buzz replacement, an intentional retirement decision,
or an accepted temporary gap.

Workflow approval infrastructure exists, but the current architecture
documentation says approval suspension and resumption are not wired end to end.
Irreversible operations must not rely solely on Buzz workflow approvals until
that path is implemented and independently tested.

### Administration, compliance, and governance

The codebase includes member roles, moderation, audit infrastructure, search,
and multi-community isolation work. However, this assessment did not locate a
documented, mature equivalent for the complete Slack enterprise administration
surface, including Slack-history migration, SAML/SCIM lifecycle management,
legal holds, DLP, and configurable retention administration.

Not every organization needs those functions. The organization should decide
which are actual requirements rather than assuming that an open-source system
must reproduce every enterprise feature.

### Operating cost

Buzz is free software, not cost-free infrastructure. An internal deployment
creates responsibility for:

- Relay availability and upgrades
- Postgres operations and migrations
- Redis operations
- Object storage and media lifecycle
- TLS certificates and domain routing
- Backups and restoration
- Monitoring, alerting, and incident response
- Capacity planning
- Security patching
- Agent runtime, model, and tool costs

The relevant comparison is not Buzz's license price against Slack's subscription
price. It is the total internal operating cost against the control, agent
integration, and data ownership benefits Buzz provides.

## Agent Security Requirements

No agent should receive broad host or production access merely because it has a
separate Buzz identity.

For an organizational pilot:

- Run agents in dedicated containers or disposable virtual machines.
- Give each agent a separate Buzz key and narrowly selected channel membership.
- Use separate, least-privilege credentials for each external system.
- Do not mount general developer home directories or credential stores.
- Do not expose production SSH keys, cloud credentials, browser sessions, or
  password-manager data.
- Restrict outbound network access where practical.
- Require a human-controlled approval mechanism outside the agent for
  destructive or irreversible actions.
- Treat messages, repository contents, uploaded documents, tool output, and web
  results as untrusted input.
- Log and review agent tool activity, not only its final channel messages.
- Test prompt-injection scenarios before allowing external guests or public
  community members to interact with agents.

Opening a community to untrusted participants while agents have shell or
credential access is a high-risk configuration.

## Proposed Pilot

### Scope

- One internal project
- Three to eight technically comfortable participants
- One or two agents
- One non-sensitive repository or project data set
- No production credentials
- No external guests during the initial phase
- Slack retained as the organization-wide system and fallback
- A 30- to 60-day evaluation window

### Deployment

- Run a dedicated Buzz relay environment.
- Terminate TLS using an organization-controlled domain.
- Use isolated Postgres, Redis, and object-storage resources.
- Establish monitoring before inviting pilot users.
- Back up the database, object storage, configuration, and required key material.
- Perform a complete restoration before declaring the pilot operational.
- Pin a reviewed Buzz release rather than continuously deploying `main`.
- Document the upgrade and rollback procedure.

### Agent setup

- Start with a single narrow role, such as project-history research or test
  triage.
- Use a repository copy without production secrets.
- Restrict filesystem access to the pilot workspace.
- Disable production deployment, customer communication, and financial tools.
- Record which model provider receives which data.
- Add capabilities only after the preceding scope has operated safely.

### What to measure

- Daily and weekly active participation
- Whether users continue working in Slack instead of Buzz
- Missed or delayed notifications
- Message, thread, search, and unread-state failures
- Mobile usability and notification reliability
- Agent task completion rate and correction rate
- Time saved by shared context
- Prompt-injection or unsafe-tool incidents
- Operator time spent maintaining the system
- Backup and recovery success
- User feedback from both technical and nontechnical participants

## Go/No-Go Gates for Replacing Slack

Buzz should not become the primary workspace until every required gate passes.

### Reliability

- [ ] Message delivery and history recovery have survived controlled outages.
- [ ] Desktop and mobile notifications are reliable enough for urgent work.
- [ ] Search remains correct and acceptably fast with representative history.
- [ ] Upgrade and rollback procedures have been exercised.
- [ ] Monitoring and operational ownership are assigned.

### Identity and recovery

- [ ] Identity backup and multi-device enrollment are documented and tested.
- [ ] A simulated lost-device recovery succeeds.
- [ ] Employee offboarding and key-compromise procedures succeed.
- [ ] The organization has decided whether identities are personal or managed.

### Data protection

- [ ] Relay-operator visibility is understood and accepted.
- [ ] Database and object-storage encryption and access controls are in place.
- [ ] Backup restoration has been tested.
- [ ] Retention, deletion, export, and historical-access requirements are met.
- [ ] Model-provider data handling is documented and approved.

### Agents

- [ ] Agents run within an acceptable isolation boundary.
- [ ] Each agent has narrow credentials and channel membership.
- [ ] Prompt-injection testing has been performed.
- [ ] Irreversible actions require dependable human approval.
- [ ] Agent tool activity is observable and reviewable.

### Organizational fit

- [ ] Every critical Slack integration has a replacement or approved exception.
- [ ] Historical Slack data has an accepted migration or archive plan.
- [ ] Guest and external-collaborator workflows are adequate.
- [ ] Pilot users prefer Buzz or show a measurable operational benefit.
- [ ] The ongoing operating cost has an assigned owner and accepted budget.

If a mandatory gate fails, Slack should remain the primary workspace while Buzz
continues as a project-specific agent collaboration system.

## Likely Organizational Fit

Buzz is most likely to fit an organization that is:

- Small or medium-sized
- Engineering-heavy
- Comfortable self-hosting critical tools
- Actively restructuring work around multiple agents
- Willing to tolerate early-product friction
- More interested in control and integrated context than a broad application
  marketplace

Buzz is least likely to fit an organization that is:

- Highly dependent on mobile-first communication
- Primarily nontechnical
- Reliant on many Slack applications
- Subject to strict identity, retention, or legal-hold requirements
- Unable to operate an internal relay reliably
- Planning to expose powerful agents directly to untrusted participants

## Final Assessment

Buzz has a coherent and valuable architectural idea: give people, agents,
conversation, code, and automation a shared identity and event substrate. For
an organization in which agents perform meaningful daily work, a focused pilot
could produce benefits that are difficult to reproduce with conventional Slack
bots.

The same integration expands the security blast radius and transfers substantial
operational responsibility to the organization. Buzz's cryptographic design
improves provenance, but it does not solve prompt injection, least privilege,
safe autonomy, recovery, or infrastructure operations.

The appropriate decision today is not "replace Slack" or "reject Buzz." It is:

> Keep Slack as the dependable organizational system, pilot Buzz as an
> agent-native project workspace, and allow evidence from the pilot to determine
> whether a broader migration is justified.

## External Sources

- [Official Buzz repository](https://github.com/block/buzz)
- [Official Buzz security policy](https://github.com/block/buzz/blob/main/SECURITY.md)
- [Official Buzz architecture and known limitations](https://github.com/block/buzz/blob/main/ARCHITECTURE.md)
- [Official Buzz agent configuration](https://github.com/block/buzz/blob/main/crates/buzz-agent/README.md)
- [Block's Goose red-team report](https://engineering.block.xyz/blog/how-we-red-teamed-our-own-ai-agent-)
- [Block leadership: Jack Dorsey](https://investors.block.xyz/governance/board-of-directors/person-details/default.aspx?ItemId=ad56d8c9-6b3e-4c89-8323-d3d260a5b423)
