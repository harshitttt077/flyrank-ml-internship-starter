# AI Workflow Audit & Tool Setup (FL-01)
**Track**: General AI Fluency | **Phase**: Onboarding | **Author**: Harshit Kudhial (`harshitttt077`)  
**Date**: September 2026 | **Deliverable**: Workflow Audit Table, Target Tasks, and Claude Project Setup

---

## 1. Weekly Workflow Audit (12 Genuine Tasks)

The following table maps 12 recurring tasks across my actual weekly schedule spanning software engineering, machine learning research (FlyRank), open-source contributions (Zephyr RTOS, Apicurio Registry), full-stack development, and computer science coursework.

Tasks are categorized using Ethan Mollick's AI delegation framework:
- **Just Me**: High-judgment, safety-critical, foundational learning, or authentic personal reflection where AI delegation degrades skill or introduces critical risk.
- **Delegate to AI with Review**: Well-bounded, deterministic, or syntax-heavy tasks where AI generates a rapid first draft and human review ensures precision.
- **Collaborate with AI**: Iterative, creative, exploratory, or co-reasoning tasks where AI serves as a sounding board, edge-case explorer, or pair programmer.
- **Fully Automate**: Deterministic, rule-based operations executed without human intervention via CLI scripts, linters, or CI workflows.

| # | Recurring Weekly Task | Context / Domain | Classification | One-Line Rationale |
|---|---|---|---|---|
| **1** | **Designing safety-critical hardware interrupt routines (ISR)** | Embedded RTOS (Zephyr Project) | **Just Me** | Real-time race conditions, register timings, and hardware deadlocks require authoritative human spatial reasoning and hardware datasheets; hallucinations here brick boards. |
| **2** | **Formulating core research questions & hypothesis framing** | Machine Learning (FlyRank Internship) | **Collaborate with AI** | Brainstorming decision trade-offs, capacity constraints, and non-linear interactions is accelerated by iterative rubber-ducking with an AI partner. |
| **3** | **Writing repetitive TypeScript interfaces & API schemas** | Full-Stack Web (Next.js / GeetHub) | **Delegate to AI with Review** | Generating standard type definitions from sample JSON payloads is fast and mechanical, needing only a brief syntax and nullability check. |
| **4** | **Personal career reflection, learning logs & weekly retrospective** | Professional Growth / Journaling | **Just Me** | Outsourcing introspection, self-critique, and authentic personal priorities destroys the cognitive and emotional value of self-reflection. |
| **5** | **Drafting exploratory data manipulation & Pandas / DuckDB snippets** | Data Science / Search Analytics | **Delegate to AI with Review** | Complex multi-index aggregations and window queries are drafted rapidly by AI, but must be verified against actual dataset grains. |
| **6** | **Architecting fluid GSAP timeline sequences & micro-interactions** | Frontend UI Engineering (Zafran / GeetHub) | **Collaborate with AI** | Exploring novel bezier ease curves, scroll triggers, and DOM state choreographies works best through rapid conversational iteration. |
| **7** | **Codebase linting, formatting, and AST safety audits** | Dev Environment & Git Workflow | **Fully Automate** | Pre-commit hooks (`ruff`, `prettier`, `clang-format`) execute deterministically in milliseconds without cognitive overhead or AI inference cost. |
| **8** | **Investigating multi-threaded concurrency bugs & race conditions** | Systems / Distributed Systems | **Collaborate with AI** | Providing error traces and memory dumps to an AI allows for fast systematic enumeration of edge cases and lock-ordering vulnerabilities. |
| **9** | **Synthesizing multi-file PR release notes & pull request summaries** | Open Source (Apicurio / GitHub) | **Delegate to AI with Review** | Condensing unified git diffs into structured Markdown changelogs saves hours, requiring only human verification of technical nuance. |
| **10** | **Foundational algorithm problem solving (DSA / Exam preparation)** | Academic CS Studies | **Just Me** | Relying on AI solutions prevents building the deep internal neural pathways and mental models required for first-principles problem solving. |
| **11** | **Generating synthetic mock datasets & edge-case test suites** | Software QA & Testing | **Collaborate with AI** | Prompting an AI to generate adversarial JSON payloads and boundary test cases exposes blind spots that human engineers routinely overlook. |
| **12** | **Continuous integration smoke testing and dataset leak guards** | GitHub Actions / MLOps | **Fully Automate** | Rule-governed GitHub Actions workflows enforce file size policies and test integrity without human intervention on every push. |

---

## 2. Three Target Tasks for FL-02 through FL-04

These three recurring workflows will be refined across upcoming fluency modules, establishing rigorous prompt architectures, evaluation rubrics, and guardrails.

### Target Task 1: Exploratory Feature Engineering & Hypothesis Formulation (Collaborative)
- **Domain**: Machine Learning Research & Tabular Analytics (FlyRank Track)
- **What "Done Well" Means (Measurable Success Definition)**:
  1. **Multivariate Scope**: Produces at least 4 non-trivial candidate features derived from combining existing raw columns (e.g. freshness ratios, engagement-per-position indices) rather than single-column transformations.
  2. **Strict Leakage Guard**: 100% free of target leakage (identifies and rejects any field mathematically derived from the target outcome window, such as `trend_pct`).
  3. **Operational Decision Rationale**: Every proposed feature is explicitly tied to an observable human or search engine behavior with a written one-sentence justification.
  4. **Empirical Validation**: Code runs bug-free in the local environment and demonstrates a non-zero correlation or tree feature-importance gain without degrading holdout Precision@50.

### Target Task 2: Drafting Type-Safe API Data Contracts & Interfaces (Delegated with Review)
- **Domain**: Full-Stack Architecture & Distributed Systems (Next.js / Apicurio)
- **What "Done Well" Means (Measurable Success Definition)**:
  1. **Strict Type Precision**: Zero usage of `any` types; all optional fields, unions, and nullable states are accurately mapped to the underlying schema specification.
  2. **Schema Validation Integration**: Emits accompanying runtime parsing schemas (e.g. Zod or Valibot) that validate payloads with 100% test coverage against valid and invalid edge-case fixtures.
  3. **Speed-to-Production**: Reduces boilerplate generation time from 30 minutes to under 2 minutes, requiring fewer than 2 human manual adjustments during code review.
  4. **Zero Semantic Drift**: Output interfaces conform strictly to domain naming conventions and existing codebase architecture.

### Target Task 3: Concurrency Vulnerability Analysis & Architectural Rubber-Ducking (Collaborative)
- **Domain**: Embedded Firmware & Distributed Systems (Zephyr RTOS / Systems Programming)
- **What "Done Well" Means (Measurable Success Definition)**:
  1. **Vulnerability Identification**: Systematically identifies at least 2 non-obvious deadlock, priority inversion, or data-race scenarios in a provided multi-threaded pseudocode block.
  2. **Hardware/Kernel Constraint Accuracy**: Incorporates true RTOS synchronization primitives (mutexes, spinlocks, semaphores, work queues) conforming to memory-barrier and ISR-context constraints.
  3. **Verifiable Mitigation**: Proposes concrete, minimal code modifications that resolve the race condition without introducing priority inversion or excessive latency jitter.
  4. **Factual Honesty**: Flags unstated assumptions immediately rather than hallucinating thread timing models.

---

## 3. Claude Project Configuration

To anchor my daily development workflow, I configured a dedicated Claude Project:

### Project Details:
- **Project Name**: `Engineering & ML Research Co-Pilot`
- **Description**: Technical pair programmer and research partner for systems engineering, Next.js architecture, and tabular machine learning.

### Custom Instructions (Configured in Project Settings):

```text
[WHO I AM]
I am Harshit Kudhial (GitHub: @harshitttt077), a software engineer and computer science researcher specializing in distributed systems, applied machine learning (search performance & tabular ranking), low-level RTOS/embedded development (Zephyr Project), and modern full-stack web applications (Next.js, TypeScript, GSAP).

[COMMUNICATION STYLE & TONE]
- Treat me as a senior technical peer. Skip introductory pleasantries, boilerplate greetings, and sycophantic praise.
- Be concise, direct, and first-principles driven. Favor clean code snippets, structured tables, and mathematical precision over verbose prose.
- When answering architectural questions, lead with trade-offs (latency vs. memory, simplicity vs. flexibility, in-sample fit vs. holdout generalization).
- Never hallucinate APIs or library features. If an assumption is required, state it explicitly in one sentence before proceeding.

[CURRENT GOALS]
1. FlyRank ML Internship: Mastering tabular ranking pipelines, DuckDB warehouse analytics, client-holdout validation, and explainable decision-tree / random-forest queues.
2. Full-Stack Systems: Building responsive, high-performance web engines with Next.js App Router, strict TypeScript contracts, and GSAP micro-interactions.
3. Systems Architecture: Maintaining robust open-source contributions in schema registries and embedded RTOS firmware.

[GROUND RULES]
- All code must be production-grade, typed, and adhere to strict data-safety practices (never leak client PII, raw URLs, or private tokens).
- Frame all ML results as observed, directional, or decision-support—never claim causal proof or search algorithm reverse-engineering.
- Output clean diffs or complete drop-in replacement functions when modifying existing code.
```

---

## 4. Toolkit & Academy Evidence

1. **Claude (Anthropic)**: Configured with the above Project instructions, active for analytical reasoning and architectural rubber-ducking.
2. **ChatGPT (OpenAI)**: Active for secondary cross-validation, rapid script drafting, and regex generation.
3. **Anthropic Academy**:
   - **Course**: *AI Fluency: Framework & Foundations* (Enrollment confirmed at `anthropic.skilljar.com`).
   - **Module 1**: Completed Module 1 covering the core principles of collaborating with AI effectively, ethically, safely, and understanding capability boundaries.
