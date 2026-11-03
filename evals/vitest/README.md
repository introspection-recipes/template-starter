# Evals

This recipe's evals live under `evals/`, which `.recipeignore` keeps out of the
deployed recipe. The layout follows the
[Recipe format](https://github.com/introspection-org/recipes/blob/main/docs/recipe-format.md#evaluations):

```text
evals/
  vitest/              vitest-evals suites
  harbor/<task>/       one Harbor task per folder
  lib/                 code shared by several tasks or suites
  sources/             pinned generators of derived eval data
  fakes/<server>/      MCP fakes and their scenarios
```

Start with a vitest-evals suite in this folder. Add a Harbor task under
`evals/harbor/<task>/` only when a case needs its own environment; in a Harbor
trial, only the task's `environment/` reaches the agent. Every eval path must
stay excluded by `.recipeignore`; the single `evals/` line does that.

To write the first suite, ask your coding agent to run
`introspection skills create/vitest`, or `introspection skills improve/vitest`
for a recipe already in use, and follow it.
