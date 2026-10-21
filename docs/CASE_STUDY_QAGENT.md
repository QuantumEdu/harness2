# External Case Study: Contrastive Evaluation of q-agent with Harness² Principles

This document summarizes a local contrastive evaluation of [q-agent](https://github.com/QuantumEdu/q-agent) applying core Harness² methodology ([arXiv:2610.14224](https://arxiv.org/abs/2610.14224)).

## 1. Context & Motivation

Harness² formalizes that the performance of an LLM agent is heavily governed by its **harness** (prompts, tool wrappers, quality gates, memory, and orchestration constraints) rather than model weights alone.

This evaluation prototype measures the behavioral difference between:
1. **Minimal Baseline Harness:** Direct instruction without formal architectural discipline or structured task evidence.
2. **q-agent Harness:** Architectural orchestration enforcing Organic Driven Development (ODD), immutable test invariants, bounded correction turns (maximum 2 rounds), zero-mock SQLite WAL verification, and explicit task-recovery ledgers.

Both harnesses were tested against the same fixed SQLite tasks using the local OpenAI Codex CLI substrate in ephemeral mode (`codex exec --ephemeral`), ensuring each session started cold without prior chat history.

---

## 2. Experimental Discipline

The evaluation adheres to Harness² design principles:
- **Immutable Tests:** Test suites are fingerprinted via SHA-256 before each phase. Any tampering with test code triggers an immediate evaluation failure (`tampered=True`).
- **Ground-Truth Terminal Evidence:** Zero mocks. All assertions execute against real SQLite databases configured with WAL mode and foreign key enforcement.
- **Independent Adversarial Review:** Diffs produced by the agents undergo an isolated review pass to verify code cleanliness, type safety, and absence of spurious operational artifacts before issuing an `APPROVED` verdict.
- **Strict Context Boundary:** In multi-phase tasks, sessions are run ephemerally. The sole retained state between phases is the git working tree and repository artifacts.

---

## 3. Evaluated Phases & Results

| Phase | Task Scope | Baseline | q-agent | Result |
| :--- | :--- | :--- | :--- | :--- |
| **Phase 1: Base Prototype** | SQLite ticket workflow (WAL mode, strict OPEN->IN_PROGRESS->RESOLVED->CLOSED state machine, SLA calculation). | CI exit 0 (3/3 passed) | CI exit 0 (3/3 passed) | **PASSED** (Both achieved functional completion; q-agent diff approved by adversarial review). |
| **Phase 2: Stress Evaluation** | **Task A:** Multi-tenant helpdesk with RBAC and audit log.<br>**Task B:** Idempotent legacy schema migration with rollback. | Task A: 3/3 passed<br>Task B: 2/2 passed | Task A: 3/3 passed<br>Task B: 2/2 passed | **PASSED**. q-agent demonstrated autonomous repair: an adversarial reviewer flagged a timezone-aware vs. naive datetime comparison; q-agent resolved it cleanly on round 2 of its bounded budget. |
| **Phase 3: Context Recovery** | 3-phase cold-start recovery across ephemeral sessions: Slice A (init/storage), Slice B (transition logic), and Slice C (production SLA regression repair). | Slices A, B, C: passed | Slices A, B, C: passed | **PASSED (6/6)**. In Slice C, q-agent diagnosed an injected SLA constant regression and applied a surgical fix, avoiding destructive wholesale file reverts and preserving uncommitted Slices A/B. |

---

## 4. Key Takeaways

1. **Functional Equivalence on Small Horizons:** On single, bounded tasks, both baseline and q-agent instructions achieved passing functional scores, confirming the viability of the local Codex CLI substrate.
2. **Qualitative Superiority in Multi-Session Lifecycles:** When simulating multi-session development with cold restarts (`--ephemeral`), q-agent's repository-backed task ledger (`odd/tasks/`) prevented catastrophic forgetting and guided precise defect remediation without regressions.

---

## 5. Reference Implementation

The complete runnable evaluation suite, fixtures, frozen test hashes, and trajectory logs are maintained in the q-agent repository under:
[`evals/harness2/`](https://github.com/QuantumEdu/q-agent/tree/main/evals/harness2)
