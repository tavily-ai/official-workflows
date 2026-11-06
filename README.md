# Tavily workflows

Agent skills that each produce one finished deliverable from the web, using Tavily.
They build on the core and task skills in [`tavily-ai/skills`](https://github.com/tavily-ai/skills)
and add the job-specific parts: what to ask, which sources to search, the output
template, and the checks the output must pass.

| Skill | Produces |
|---|---|
| `tavily-literature-review` | A literature review from papers and preprints |
| `tavily-market-research` | A market report with sourced, dated size estimates and players |
| `tavily-competitor-analysis` | A competitor comparison plus a pricing and changelog monitor |
| `tavily-due-diligence` | A diligence memo on one company, with red flags |
| `tavily-lead-gen` | A prospect list qualified against an ideal customer profile |
| `tavily-meeting-prep` | A one-page brief on a company and the people in a meeting |

## Install

```bash
npx skills add tavily-ai/official-workflows
```

Install one workflow:

```bash
npx skills add tavily-ai/official-workflows --skill tavily-literature-review
```

The workflows call Tavily through the `tvly` CLI or the Tavily MCP server. Sign in once
with `tvly login`, or set `TAVILY_API_KEY`. Each one installs the `tavily-ai/skills` skill it depends on the first
time it runs, if that skill is missing.