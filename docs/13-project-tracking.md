# DrawSaver — Project Tracking Document

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  

---

## 1. Sprint Planning Structure

### Sprint Configuration

| Parameter | Value |
|-----------|-------|
| Sprint duration | 2 weeks |
| Sprint planning | Monday, Week 1 (1 hour) |
| Daily standup | 15 minutes async (Slack/Discord thread) |
| Sprint review | Friday, Week 2 (30 minutes) |
| Sprint retro | Friday, Week 2 (30 minutes) |
| Velocity target | 30–40 story points per sprint |

### Story Point Scale

| Points | Effort | Example |
|--------|--------|---------|
| 1 | Trivial (< 2 hours) | Fix a typo, update a constant |
| 2 | Small (2–4 hours) | Add a new validation rule, simple UI tweak |
| 3 | Medium (4–8 hours) | Implement a new API endpoint, build a component |
| 5 | Large (1–2 days) | Build drawing tool integration, WebSocket event handler |
| 8 | Extra Large (2–3 days) | Implement full auth flow, build canvas engine |
| 13 | Epic (3–5 days) | Full real-time collaboration system, version history |

### Sprint Backlog Template

```markdown
## Sprint X: [Sprint Goal]
**Dates:** YYYY-MM-DD → YYYY-MM-DD  
**Goal:** [One sentence describing what this sprint delivers]  
**Capacity:** XX story points

### Committed Items
| ID | Title | Points | Assignee | Status |
|----|-------|--------|----------|--------|
| DS-101 | Implement user registration | 5 | @dev1 | ✅ Done |
| DS-102 | Build color picker component | 3 | @dev2 | 🔄 In Progress |
| DS-103 | Socket.IO room join flow | 8 | @dev1 | 📋 To Do |

### Sprint Metrics
- **Committed:** 35 points
- **Completed:** 22 points
- **Carried Over:** 13 points
- **Velocity:** 22 (trending ↑)
```

---

## 2. Task Management Structure

### Issue Types

| Type | Prefix | Description | Example |
|------|--------|-------------|---------|
| **Feature** | `feat:` | New user-facing functionality | `feat: Implement pencil tool` |
| **Bug** | `bug:` | Something broken | `bug: Canvas not saving on disconnect` |
| **Tech Debt** | `chore:` | Refactoring, cleanup, deps | `chore: Refactor auth middleware` |
| **Docs** | `docs:` | Documentation updates | `docs: Add API authentication guide` |
| **DevOps** | `ops:` | Infrastructure, CI/CD | `ops: Set up GitHub Actions pipeline` |
| **Design** | `design:` | UI/UX design work | `design: Drawing workspace layout` |

### Issue Template

```markdown
## Title
[Type prefix]: [Concise description]

## Description
[What needs to be done and why]

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Technical Notes
[Implementation hints, related files, dependencies]

## Labels
- Priority: P0/P1/P2/P3
- Type: feature/bug/chore/docs
- Component: frontend/backend/socket/database
- Phase: MVP/Phase2/Phase3

## Story Points: [X]
```

### Kanban Board Columns

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ Backlog  │→ │ To Do    │→ │ In       │→ │ Review   │→ │ Done     │
│          │  │ (Sprint) │  │ Progress │  │ (PR)     │  │          │
│ Groomed  │  │ Committed│  │ Actively │  │ Code     │  │ Merged   │
│ & ready  │  │ to       │  │ working  │  │ review + │  │ to main  │
│          │  │ sprint   │  │ on it    │  │ QA test  │  │          │
└──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘
```

### GitHub Project Board Setup

Use **GitHub Projects (v2)** with the following custom fields:

| Field | Type | Values |
|-------|------|--------|
| Status | Single select | Backlog, To Do, In Progress, In Review, Done |
| Priority | Single select | P0-Critical, P1-High, P2-Medium, P3-Low |
| Sprint | Iteration | 2-week sprints |
| Story Points | Number | 1, 2, 3, 5, 8, 13 |
| Component | Single select | Frontend, Backend, WebSocket, Database, DevOps |
| Phase | Single select | MVP, Phase 2, Phase 3, Phase 4, Phase 5 |

---

## 3. Milestone Tracking

### Milestones

| Milestone | Target Date | Key Deliverables | Exit Criteria |
|-----------|------------|-------------------|---------------|
| **M1: Project Setup** | Week 1 | Repo, CI/CD, Docker, DB | All devs can run locally |
| **M2: Auth Complete** | Week 1 | Login, register, OAuth, JWT | User can login and hit protected API |
| **M3: Canvas MVP** | Week 2 | All drawing tools working solo | User can draw, undo, zoom |
| **M4: Persistence** | Week 3 | Save, load, auto-save, dashboard | Drawing persists across sessions |
| **M5: MVP Launch** | Week 4 | Landing page, export, polish | End-to-end solo drawing flow |
| **M6: Real-Time Sync** | Week 6 | WebSocket, multi-user drawing | 2+ users draw on same canvas |
| **M7: Collaboration** | Week 7 | Cursors, presence, reconnection | Stable 5+ user sessions |
| **M8: Advanced Features** | Week 9 | Layers, versions, all exports | Feature-complete beta |
| **M9: Production** | Week 12 | Testing, DevOps, monitoring | Production deployed + monitored |

### Milestone Burndown Example

```
Sprint Points Remaining:

40 │ ■
35 │ ■ ■
30 │     ■ ■         ← ideal burndown
25 │         ■
20 │     ●       ■
15 │       ●       ■     ← actual progress
10 │         ●       ■
 5 │           ● ●     ■
 0 │               ● ●   ■
   └──────────────────────
     D1  D2  D3 ... D10
```

---

## 4. Bug Tracking Workflow

### Bug Severity Levels

| Severity | Response Time | Fix Time | Example |
|----------|-------------|----------|---------|
| **S0 — Critical** | Immediate | < 4 hours | Data loss, auth bypass, app crash |
| **S1 — Major** | < 4 hours | < 24 hours | Drawing sync fails, can't save |
| **S2 — Minor** | < 24 hours | Next sprint | UI alignment, wrong color display |
| **S3 — Trivial** | Best effort | When convenient | Typo, minor visual glitch |

### Bug Report Template

```markdown
## Bug Report

### Summary
[One-line description]

### Environment
- Browser: Chrome 125
- OS: Windows 11
- Account: john@example.com
- Room ID: room-abc123

### Steps to Reproduce
1. Go to drawing workspace
2. Select rectangle tool
3. Draw a rectangle
4. Press Ctrl+Z to undo
5. Observe: rectangle is not removed

### Expected Behavior
Rectangle should be removed from canvas

### Actual Behavior
Rectangle remains on canvas. Undo counter shows -1.

### Screenshots/Video
[Attach if applicable]

### Console Errors
```
TypeError: Cannot read property 'remove' of undefined
  at UndoManager.undo (undoRedo.js:45)
```

### Severity: S1
### Component: Frontend/Canvas
```

### Bug Lifecycle

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  New     │───►│ Triaged  │───►│ In Fix   │───►│ In       │
│ (report) │    │ (assign  │    │ (dev     │    │ Review   │
│          │    │  + sev)  │    │  working)│    │ (PR)     │
└──────────┘    └──────────┘    └──────────┘    └────┬─────┘
                                                     │
                ┌──────────┐    ┌──────────┐         │
                │ Won't    │    │ Verified │◄────────┘
                │ Fix      │    │ (QA pass)│
                └──────────┘    └──────────┘
                                     │
                                ┌────▼─────┐
                                │ Closed   │
                                └──────────┘
```

---

## 5. Team Workflow

### Git Workflow (GitHub Flow)

```
main (production-ready)
 │
 ├── feature/DS-101-user-registration
 ├── feature/DS-102-color-picker
 ├── fix/DS-201-undo-crash
 └── chore/DS-301-update-deps
```

**Rules:**
1. `main` is always deployable
2. Create feature branches from `main`
3. Branch naming: `{type}/DS-{id}-{short-description}`
4. Open PR when ready for review
5. Require 1 approval + passing CI before merge
6. Squash merge to keep history clean
7. Delete branch after merge

### PR Template

```markdown
## Description
[What this PR does and why]

## Related Issue
Closes #DS-XXX

## Type of Change
- [ ] Feature
- [ ] Bug fix
- [ ] Refactor
- [ ] Docs

## Checklist
- [ ] Code follows project conventions
- [ ] Self-reviewed the code
- [ ] Added/updated tests
- [ ] Updated documentation if needed
- [ ] No console.log or debug code left
- [ ] Tested on Chrome + Firefox

## Screenshots (if UI change)
[Before/After screenshots]
```

### Commit Convention (Conventional Commits)

```
feat: add pencil drawing tool
fix: resolve canvas undo crash on empty stack
docs: update API authentication guide
chore: upgrade Socket.IO to v4.8
refactor: extract canvas event handlers to hook
test: add unit tests for auth service
style: fix toolbar alignment on mobile
perf: optimize cursor broadcast throttling
```

### Code Review Checklist

- [ ] Does the code work as described in the PR?
- [ ] Are there any obvious bugs or edge cases?
- [ ] Is the code readable and well-structured?
- [ ] Are there unnecessary comments or dead code?
- [ ] Are error cases handled?
- [ ] Are there security concerns (injection, auth bypass)?
- [ ] Do new API endpoints have validation?
- [ ] Are WebSocket events properly cleaned up on disconnect?
- [ ] Do canvas operations handle undo/redo correctly?
- [ ] Are new features responsive on mobile?

---

## 6. Communication Channels

| Channel | Purpose | Tool |
|---------|---------|------|
| Sprint planning | Bi-weekly planning sessions | Discord voice / Google Meet |
| Daily updates | Async standup (what I did, what I'll do, blockers) | Discord/Slack thread |
| Code review | PR discussions | GitHub PR comments |
| Bug reports | Issue tracking | GitHub Issues |
| Design feedback | UI/UX reviews | Discord channel with screenshots |
| Architecture decisions | ADRs (Architecture Decision Records) | GitHub Discussion or `docs/adr/` |
| General chat | Quick questions, coordination | Discord/Slack |

---

## 7. Definition of Done

A feature/task is considered **Done** when:

- [ ] Code is written and follows project conventions
- [ ] Unit tests pass (coverage ≥ 80% for new code)
- [ ] Integration/E2E tests pass if applicable
- [ ] Code reviewed and approved (1+ reviewer)
- [ ] CI pipeline passes (lint + test + build)
- [ ] Documentation updated if needed
- [ ] Merged to `main` branch
- [ ] Deployed to staging (verified)
- [ ] No known regressions
