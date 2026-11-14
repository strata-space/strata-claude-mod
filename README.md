# StrataSpace mod for Claude Code

A terminal document view for [StrataSpace](https://strata.space) in Claude Code.
It turns the terminal into a document surface: a pane beside the chat shows the
document Claude is reading, and a band above the prompt tells you when it
changes.

This is a Claude Code hooks module (a "mod"). It runs in Claude Code builds that
have hooks modules enabled; they are rolling out. On a build without them the
plugin installs and stays idle.

## Install

Works on its own: it registers the StrataSpace MCP server. Install the `strata`
plugin beside it for the skills; Claude Code then runs one server for both.

```
/plugin marketplace add strata-space/marketplace
/plugin install strata-mod@strata-space
```

The `strata` CLI is optional. Only the live tier needs it.

## What it does

- **A pane opens when Claude reads a document.** `read_document` draws the
  document, or the sections Claude read, in a pane beside the chat: an outline
  strip (`0` to `9` jump to headings, `r` re-reads the whole), the revision, and
  html embeds folded to a one-line placeholder (`show`/`fold` toggles the
  source; embeds run in the web app, not the terminal). The pane docks on its
  own in a wide terminal (144 columns); `/strata doc` opens it anywhere.
- **A band above the prompt tracks what Claude has read.** It checks the
  revision every ten seconds and shows `v<n> stale` when someone edits a
  document after Claude read it; **Refresh stale** re-reads it into Claude's
  context. **Hide** puts the band away for the session; `/strata band` brings it
  back.
- **Read with a cursor.** With the pane focused (`ctrl+x tab` cycles focus):
  `j`/`k` move a cursor over paragraphs, `v` extends the selection, `c` posts a
  comment anchored to the paragraph, `q` quotes it into your prompt, `s`
  proposes a replacement as a suggestion, `a` asks Claude about it, `x` clears.
  Comment threads draw under the paragraph they quote, with reply and resolve.
  Writes from the pane always follow your keystroke; the module never writes on
  its own.
- **`/strata`** opens an explorer: search your Spaces, open a result in the pane
  or read it into Claude's context, and a review queue of the suggestions and
  comments made this session, with accept and reject.
- **`@strata:`** in the prompt completes to a document from your Spaces. On
  submit, the document's text reaches Claude as context and the mention reads as
  the document's title and id.
- **Live, with the CLI.** When a `strata` CLI with `doc watch` is installed and
  signed in (`strata login`) to the deployment the document lives on, the pane
  follows the document live: edits appear as they land, collaborators show in
  the header, and the section your cursor is in shows as your presence in the
  web editor. `/strata live` reports or switches it (`/strata live off`).
  Without the CLI the pane polls revisions over MCP instead.

## Options

Under `/config`, plugin options: **strata CLI** (`cliPath`, default `strata` on
`PATH`), **Live watch** (`liveWatch`), **Open the pane on read** (`autoOpen`).

## Layout

```
.claude-plugin/plugin.json   # manifest: state contract, user options
.mcp.json                    # the StrataSpace MCP server
hooks/hooks.json             # names the module
hooks/register.tsx           # the module; parts and tests beside it
types/index.d.ts             # the module's state contract (what it keeps in $.state)
```

See [CONTRIBUTING.md](CONTRIBUTING.md) to work on it. License: MIT.
