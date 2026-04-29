---
name: super-ai-ml-ops
description: "AI/ML operations: evaluation, monitoring, cost control, and reliability. Use for productionizing AI systems."
---

# Super AI/ML Ops

## Overview
Make AI systems stable and measurable in production.


## User Intent Examples
- "Need help with LLM Evaluation for my product/site."
- "Create a plan for LLM Ops."
- "Not sure where to start, need a quick assessment."

## Workflow
1. Define evaluation metrics, datasets, and acceptance thresholds.
2. Set up observability for quality, latency, and errors.
3. Implement caching and cost controls.
4. Create monitoring and alerting for regressions.
5. Establish release and rollback procedures for prompts/models.
6. Document runbooks and ongoing QA cadence.

## Minimal Intake Questions
- Primary goal or outcome
- Scope (pages, systems, teams, or timeframe)
- Constraints (tools, budget, timeline)

## Output Format
- Eval plan and scoring rubric
- Monitoring and alerting checklist
- Cost and caching strategy
- Release and rollback plan
- Runbook and QA cadence

## Routing Map (Modules)
- **LLM Evaluation** -> `references/modules/llm-evaluation.md`
- **LLM Ops** -> `references/modules/llm-ops.md`

## Bundled References
- `references/modules/`
- `references/toolkit/`
- `scripts/`
- `assets/`
- `agents/`

## Compatibility Notes
- If any module references slash commands or tool-specific legacy paths, translate them into plain-language steps.
- Keep outputs platform-agnostic unless the user specifies a specific tool, stack, or agent.

## Guardrails
- Do not rely on single metrics; include qualitative checks.
- Track cost per request and cap budgets.
- Treat prompt/model updates as production changes.
