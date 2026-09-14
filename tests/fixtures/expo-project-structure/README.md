# Project structure regression cases

These advice cases cover ENG-26768. They use the case schema from
`.claude/skills/expo-skill-eval/SKILL.md` and stay outside the distributed plugin.

Run each `prompt` in a fresh agent context with the local
`plugins/expo/skills/expo-project-structure/SKILL.md`. For case 3, also supply
`.claude/skills/expo-skill-eval/references/runtime-matrix.md`. Give the executor
only the prompt and those inputs, then grade its saved response against every
`expectations` item. Do not give the executor the expected answers.

All cases request advice, so `platforms` is empty and no app, simulator, or
screenshot is needed. These are behavioral regressions for an agent to run,
not automated Bun tests. Record the results in the pull request.
