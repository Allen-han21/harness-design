# harness-design

Harness design patterns, experiments, and implementation notes for long-running AI systems and agentic development.

## Overview

This repository is a working space for designing harnesses that help AI agents operate reliably over long-running tasks.

The focus is not only on prompting, but on the full execution environment around the model:

- context shaping
- task planning
- tool orchestration
- evaluator loops
- guardrails and policy enforcement
- observability and feedback
- human escalation points

## Why This Repo

As models become more capable, the main engineering problem shifts from "what should we ask?" to "what system should we build around the model?"

This repository exists to explore that system design layer in a practical way.

## Core Questions

- How should long-running agent sessions store and hand off state?
- When should a task be handled by a single agent vs multiple roles?
- Which rules should remain as prompts, and which should be enforced mechanically?
- How should evaluation, retry, rollback, and escalation be designed?
- What metrics best reflect harness quality in production?

## Initial Scope

- planner / generator / evaluator patterns
- structured handoff artifacts
- deterministic validation layers
- agent-readable repository design
- cost and latency tradeoff experiments
- safety boundaries for state-changing actions

## Suggested Structure

```text
docs/
  principles/
  patterns/
  plans/
  evaluations/
examples/
scripts/
```

## Roadmap

1. Define the core design principles for long-running AI harnesses.
2. Document reusable harness patterns and failure modes.
3. Build small prototypes to validate orchestration choices.
4. Add evaluation rubrics and measurable success criteria.
5. Evolve the repository into a reusable reference project.

## Status

Bootstrap phase. The repository currently starts with the project definition and will grow around practical experiments.
