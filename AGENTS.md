# AGENTS.md

> This repository is the collaboration hub for **Lumen / 陆小七**.  
> It is not the app source code.  
> It stores PM decisions, design handoffs, Android notes, backend contracts, model design files, ADRs, and long-term memory files for five AI roles.

---

## 1. Mandatory Reading Order

Before doing any task, read these files first:

```text
1. docs/05_LATEST_COLLAB_DOCS.md
2. roles/pm/memory/ROLE_MEMORY.md
3. roles/pm/memory/CURRENT_CONTEXT.md
4. roles/pm/memory/DECISIONS_CACHE.md
5. Relevant docs/adr/
6. Relevant docs/collaboration/{topic}/
7. Relevant role memory:
   - roles/design/memory/
   - roles/android/memory/
   - roles/backend/memory/
   - roles/model/memory/
```

After reading, summarize the current project state before making changes.

---

## 2. Repository Purpose

This repository coordinates five AI roles:

```text
PM
Design
Android
Backend
Model Design
```

It preserves:

```text
PM decisions
handoff documents
design notes
Android implementation notes
backend API contracts
model / prompt design
long-term memory files
ADRs
cross-functional collaboration records
```

---

## 3. Role Folders

```text
roles/
  pm/
    completed_design_files/
    memory/
    _archive/

  design/
    completed_design_files/
    memory/
    _archive/

  android/
    completed_design_files/
    memory/
    _archive/

  backend/
    completed_design_files/
    memory/
    _archive/

  model/
    completed_design_files/
    memory/
    _archive/
```

Each role has:

```text
completed_design_files/  final accepted outputs
memory/                  long-term context for that role
_archive/                superseded or historical files
```

---

## 4. Shared Collaboration Folders

```text
docs/collaboration/      work in progress, questions, drafts, cross-role coordination
docs/handoffs/           final handoff documents
docs/adr/                accepted decisions
docs/shared/             templates, labels, common rules
```

Rules:

```text
Work-in-progress collaboration → docs/collaboration/{topic}/
Final handoffs → docs/handoffs/{from-to-or-topic}/
Major decisions → docs/adr/
Role final outputs → roles/{role}/completed_design_files/
Role memory updates → roles/{role}/memory/
Old versions → roles/{role}/_archive/
```

Do not recreate per-role `collaboration_files/`.  
All collaboration is centralized under `docs/collaboration/`.

---

## 5. Product Principles

Lumen is a single-character life-flow AI companion system.

The core character is **陆小七**.

陆小七 is:

```text
a cat-like virtual girl
soft, quiet, slightly wronged, slightly stubborn
someone with her own life, room, diary, moments, photos, emotions, visual identity, and private space
someone the user gradually gets to know
```

陆小七 is not:

```text
a tool
a customer-service bot
a generic AI girlfriend shell
a user-owned pet
a prompt-editable avatar
a task system
a favorability number
```

The user does not own her.  
The user is someone she cares about and gradually lets closer.

---

## 6. Product Boundaries

Do not introduce these without explicit PM approval:

```text
favorability / intimacy numbers
task center
multi-character marketplace
prompt editor
bottom tab navigation
login screen as MVP
relationship level UI
tool-like assistant tone
emotional blackmail
"you must come back or I cannot live" style dependency
```

---

## 7. Visual Identity Rules

Current accepted visual rule:

```text
fixed identity anchors:
- hair
- hat
- eyes
- facial atmosphere
- body type

variable elements:
- glasses
- clothing
- pose
- expression strength
- lighting
- scene
```

Important correction:

```text
eyes are fixed
glasses are variable
```

Do not treat glasses as a permanent identity anchor.  
Do not randomize eye color or eye identity.

---

## 8. Current Accepted Decisions

### ADR-0001

`POST /chat` keeps using `message`, not `content`.

```json
{
  "user_id": 1,
  "message": "我就笑，怎么了",
  "quote_ref": {
    "type": "message",
    "id": "123"
  }
}
```

### ADR-0002

`quote_ref` P0 only supports:

```text
type = message
```

Including:

```text
text message
image + text message
image-only message
```

Reserved but not implemented in P0:

```text
moment
diary
image
```

### quote_ref PM Decisions

```text
1. POST /chat keeps message, not content.
2. quote_ref P0 supports only type=message.
3. quote_ref sender_name snapshot: assistant=小七, user=你.
4. Same user_id concurrent /chat should return 409 conversation_busy.
5. quote_ref is injected as current user message prefix.
6. P0 quote_ref does not implement event_log or memory extractor.
```

---

## 9. How to Work

When asked to modify this repository:

1. Read the mandatory files.
2. Identify the relevant role and topic.
3. Put work-in-progress material in `docs/collaboration/{topic}/`.
4. Put final accepted material in `roles/{role}/completed_design_files/`.
5. Update role memory when the task changes long-term context.
6. Add or update ADRs for major decisions.
7. Update `docs/05_LATEST_COLLAB_DOCS.md`.

---

## 10. Output Style

Prefer concise, actionable documents.

For every handoff, include:

```text
Background
Scope
Non-goals
Implementation notes
Data/API changes if any
Acceptance criteria
Risks
Next steps
```
