# Super AI/ML Ops

Operate AI systems with evaluation, monitoring, reliability, cost controls, and release discipline.

## Install

Copy this folder into your agent's skills directory, then restart or reload the agent.

```bash
cp -R super-ai-ml-ops ~/.your-agent/skills/
```

Use it by name:

```text
Use $super-ai-ml-ops to help with this request.
```

## Best For

- LLM evaluation
- AI monitoring
- cost control
- release gates
- production reliability

## Outputs

- evaluation plan
- monitoring and alerting plan
- cost and latency controls
- release checklist

## Modules

| Module | Purpose |
| --- | --- |
| `llm-evaluation.md` | Evaluation design, test sets, scoring, regressions, and quality gates |
| `llm-ops.md` | Monitoring, cost control, latency, reliability, deployments, and operational guardrails |

## Example Prompts

- `Use $super-ai-ml-ops to design evals for this chatbot.`
- `Use $super-ai-ml-ops to reduce LLM cost and latency.`
- `Use $super-ai-ml-ops to create a production monitoring plan for this agent.`

## Package Contents

- `SKILL.md` is the installable skill entry point.
- `references/modules/` contains detailed workflows loaded only when needed.
- `agents/` contains optional agent metadata where supported.
- `scripts/` and `assets/` are optional helpers when bundled.

## Compatibility

This skill is plain Markdown and is intended to be agent-agnostic. If a bundled helper mentions a specific tool path, translate that instruction to the equivalent path for your environment.

## Version

See `VERSION` and `CHANGELOG.md`.

## Licence

MIT. See the root repository `LICENSE`.
