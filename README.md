# team-report

Multi-agent team analysis producing an interactive paginated HTML report with inline Accept/Defer/Reject decision cards and a feedback loop.

Dynamically spawns 3–10 expert subagents based on scope, runs them in parallel, and generates a self-contained HTML report. Decisions persist in `localStorage` and accumulate in `docs/reports/feedback.yaml` for progressive improvement.

## Install (Claude Code)

```
/plugin marketplace add DevP0tion/DevP0tion
/plugin install team-report@devp0tion
```

## Usage

```
/team-report color theme analysis
/team-report lobby component architecture review
/team-report performance optimization report
```

The skill and its reference docs live under [`skills/team-report/`](skills/team-report/).
