# session-replay

A Claude Code skill that turns a conversation or work session into a short animated page. Agents
show up as avatars, each hand-off is a labelled pill that flies from one agent to another, and
counters and a short list of key moments sit alongside. It is meant for long or multi-agent runs
where you want to see who led, who did the work, what was checked and what failed, without reading
the transcript. The output is a single self-contained HTML file.

## Install

In Claude Code:

```
/plugin marketplace add M0hammedHaris/session-replay-plugin
/plugin install session-replay@session-replay
```

Or from a shell:

```bash
claude plugin marketplace add M0hammedHaris/session-replay-plugin
claude plugin install session-replay@session-replay
```

A local clone works too: `claude plugin marketplace add ./session-replay-plugin`.

## Use

Run `/session-replay` (shown as `/session-replay:session-replay` when another skill has the same
name), or just ask: "show me visually what happened in this session", "replay last night's run".

Claude collects what happened from the conversation and whatever evidence survives (git history,
logs, subagent results), fills in the data object, and gives you the page.

## The data object

The page is a fixed template, `skills/session-replay/assets/template.html`, driven by one object
between the markers `/*REPLAY_DATA_START*/` and `/*REPLAY_DATA_END*/`.

| field | what it is |
|---|---|
| `title`, `when`, `lede` | heading, eyebrow line (date or time range), one-sentence summary |
| `helpersLabel`, `outputLabel`, `momentsTitle`, `openTitle` | section labels |
| `agents` | `{id, name, role, tier, avatar}`; `tier` is `lead`, `worker` or `helper`; `avatar` is `spark`, `ghost`, or omitted for initials |
| `steps` | ordered events, each with an optional time `t` and a short `label`; `kind` is `send` (`from`, `to`), `check` (`who`), `spawn` (`by`, `who`) or `output` (`who`) |
| `moments` | up to four `{when, title, text, tone}` highlights |
| `open` | `{title, text}` items: what is unfinished, unverified or waiting on you |

`send` steps also take `starts`, `ends`, `verdict` (`"ok"` or `"bad"`) and `tone`. The full
reference is in [SKILL.md](skills/session-replay/SKILL.md).

## Network note

The page loads the [Motion](https://motion.dev) library from the jsDelivr CDN. If that script is
blocked or you are offline, it falls back to the browser's built-in Web Animations API, so the
replay still plays.

## License

MIT
