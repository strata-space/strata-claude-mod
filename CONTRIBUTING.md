# Contributing

Thanks for helping improve the StrataSpace mod.

## What this repo ships

A Claude Code hooks module under `hooks/`: TypeScript that Claude Code runs
directly. There is no build step and no package manager. The skills and the MCP
server live in the `strata` plugin (`strata-space/strata-claude-plugin`), not
here.

## Before you open a pull request

1. Validate and test the module:
   ```bash
   claude plugin validate .
   claude plugin test .
   ```
   `claude plugin test` exists only on a Claude Code build with hooks modules
   enabled (they are rolling out). CI pins a build that has it.
2. Type-check. Load the plugin once (`claude --plugin-dir .`) so Claude Code
   lays its API types under `.claude-plugin/types/` (gitignored), which the
   repo's `tsconfig.json` extends, then run `npx --package typescript tsc -p .`.
3. Read the rules in [CLAUDE.md](CLAUDE.md).
4. Use Conventional Commits.
