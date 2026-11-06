# Tavily workflows repo

Source of truth for Tavily workflow skills. Each skill under `skills/` produces one
specific deliverable, such as a literature review or a meeting brief, and builds on the
core and task skills in `tavily-ai/skills` (`tavily-search`, `tavily-research`,
`tavily-list`, `tavily-entity`, `tavily-monitor`).

## Authoring rules

- Follow `docs/workflow-authoring.md`.
  `skills/tavily-literature-review/SKILL.md` is the reference implementation.
- Layout is `skills/<skill-name>/SKILL.md`, with longer material in
  `skills/<skill-name>/references/`, one level deep.
- When adding a skill, add it to the table in `README.md` and to `skills` in
  `.claude-plugin/plugin.json`. The Codex and Cursor manifests load all of `skills/`.
- There is no router skill. Agents pick a workflow by its description, so each
  description names its deliverable and the neighbouring skill to use instead.
- Search mechanics belong in `tavily-ai/skills`, not here. A workflow states only what
  is specific to its job.
