# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. It is for contributors: Claude Code does not load it for people who install the plugin.

## What this repo is

The `strata-mod` plugin: a Claude Code hooks module (TypeScript the engine runs directly) that shows StrataSpace documents in the terminal. There is no build step and no package manager.

- `hooks/register.tsx` (named by `hooks/hooks.json`): the module. A document pane on `read_document` (`hooks/markdown.ts` cuts sections, folds html embeds, makes the blocks the cursor walks), a band above the prompt with revision polling over `find` info, the explorer and review queue, `@strata:` mentions, and the live tier over `strata doc watch` (`hooks/watch.ts` parses its JSON lines).
- `hooks/pane.ts`: the pure side (views, budgets, selections, anchors). `hooks/strata.ts`: reading the MCP server's answers.
- `types/index.d.ts`: the `$.state` contract, named by `plugin.json` `types`. `userConfig` in `plugin.json` declares the options.
- `.claude-plugin/plugin.json`: the manifest. It lists the `strata` plugin under `dependencies`; that plugin provides the MCP server and the skills. This repo bundles no `.mcp.json`.

## Commands

```bash
# Validate (manifest + what the module hooks and calls) and test (hooks/*.test.ts)
claude plugin validate .
claude plugin test .
# Type-check after one load (`claude --plugin-dir .` lays .claude-plugin/types/, gitignored)
npx --package typescript tsc -p .
```

## Hooks module rules

- Every `$.state` atom, `e.plugin` comparison and contract is keyed under the plugin's own name, `strata-mod`.
- The MCP server belongs to the `strata` plugin. Calls name it in the tool-name spelling, `plugin_strata_strata` (`plugin:strata:strata` in `/mcp`), until the module observes an `mcp__<server>__read_document` call and uses that server. Do not bundle a server here.
- Every write to Strata from the pane follows a press or a keystroke; the module never writes on its own, and it never holds, rewrites or denies the model's own document writes (`rewrite_document`, `replace_document_text`, `insert_document_section`): review-before-write, if ever wanted, belongs on the edit tools server-side, not in a client-side gate. `$.mcp.call` raises no permission prompt.
- `$.state` values are declared in `types/index.d.ts` with a `shape` tag per atom; change the tag when a value's fields change, so a reload declines what an older version stored.
- No new hardcoded URLs: document links come from `find` info (`url`) or the workspace-chooser deep link in `hooks/strata.ts`.
- The live tier spawns the CLI the `cliPath` option names, `strata` by default; it must degrade to revision polling when the CLI is missing, signed out, too old, or not saying which deployment it is signed into (`probeTier`), never fail the pane. A watch follows a document only when the document's link in the web app (from `find` info) names the deployment the CLI's `status --json` `apiUrl` names (`deploymentOf`): the CLI watches by id on its own deployment, and another deployment's MCP server may hold a document of the same id. A document is the pair of its MCP server and its id everywhere the module tells documents apart (`DocumentRef`, `isViewOf`, `isRowOf`).
- Tests live beside the code (`hooks/*.test.ts`) and run with `claude plugin test .`; the kit's stand-ins for `$` calls answer `{ value }`, plain events answer their result, and a `process.spawn` stand-in is an async generator.
- `hooks/hooks.json` keeps both shapes: an empty classic `hooks` record, so a Claude Code build without hooks modules (they are rolling out) validates and loads the plugin without the module, and `modules` naming `register.tsx` for a build with them. Keep both. CI pins a Claude Code build that has `claude plugin test` and fails when the tests fail.
- The module changes its state through one queue of steps (`step` in `hooks/register.tsx`): a step reads the atoms, decides, writes and returns, and waits on nothing else (no `$.mcp.call`, no process, no question). Work that waits runs outside the queue and brings its answer in as a step judged against the state then: a fetch by the view's `nav` (one more on every navigation), the watch's events by its identity (`watching === watch`), a post by the draft's id and the field's generation. Do not add module-level sequence counters or "judged inside the update and again after" checks; add a step, or extend the runner that owns the concern (`settlePane`, `syncWatch`, `requestThreads`, `probe`, `runSearch`, `checkRevisions`). `hooks/pane.ts` is the pure side (views, budgets, selections, anchors): keep `$` out of it. `hooks/interleave.test.ts` drives the module through every ordering of held fetches, a read of the model's and a close; add to its events when adding a path that can run beside another.
