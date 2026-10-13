---
name: session-replay
description: Build an animated visual replay of what happened in a conversation or work session - which agent led, which agents did the work, what was handed over, what came back, what was checked, what failed and how it was fixed - as one HTML page with avatars and messages flying between agents. Use when the user asks to see, visualise, replay, animate or get a visual view or recap of what happened in this session, of an overnight or long-running run, of a multi-agent or subagent workflow, or says they do not want to read the whole transcript. Also use for /session-replay.
---

# Session replay

Turn the session into a short animated page: agents as avatars, each hand-off as a labelled pill that
flies from one agent to another, a one-line tagline per step, counters, and a short list of what
mattered. The reader should understand the flow without reading logs.

The page is a fixed template driven by one data object. Your job is to write that object truthfully.

## Steps

1. **Collect what happened.** Use the conversation first. If the session ran tools, also read the
   evidence that survives: git history (`git log --format='%h %ad %s' --date=format:%H:%M`), run
   logs and report files, task or subagent results, timestamps on files. If the context was
   summarised, read the transcript file the summary points to for anything you need exactly.
2. **Name the agents.** Everyone who acted gets an entry: the lead (usually you), each worker that
   produced work (a coding CLI, a subagent doing the build, a human who made a decision), and each
   helper spawned for a narrow job (reviewers, searchers, verifiers).
3. **Write the steps** in the order they happened, using the step kinds below.
4. **Pick the moments** (at most four) and the **open items** (what is unfinished, unverified or
   waiting on the user).
5. **Build the page.** Copy `assets/template.html` from this skill's own folder
   (`${CLAUDE_SKILL_DIR}/assets/template.html`) to a working file. Replace everything between
   `/*REPLAY_DATA_START*/` and `/*REPLAY_DATA_END*/` with your object. Change the `<title>` at the
   top to a short name for this replay (two to four words). Change nothing else in the template
   unless the user asks for a design change.
6. **Check it parses.** Extract the `<script>` body and run `node --check` on it if Node is there.
7. **Deliver.** If an Artifact tool exists, publish the file (icon `timeline`) and give the link.
   Otherwise save it where the user can open it and give the path. Say in one or two lines what
   the page shows, and anything you could not verify about it.

## The data object

```js
{
  title: "Night Shift Replay",              // also put a matching <title> at the top of the file
  when: "13 Oct 2026 · 00:36 to 04:14",     // eyebrow line; date, range, or "this conversation"
  lede: "One sentence: who led, who worked, who checked.",
  helpersLabel: "Reviewer agents spawned by Claude",
  outputLabel: "commits",                   // what an output dot stands for: commits, files, PRs, drafts
  momentsTitle: "Three bugs the reviewers stopped",
  openTitle: "Where it stopped",
  agents: [
    {id:"claude", name:"Claude", role:"orchestrator · edits no code", tier:"lead", avatar:"spark"},
    {id:"be", name:"Kiro · backend", role:"coder", tier:"worker", avatar:"ghost"},
    {id:"r1", name:"python-reviewer", role:"read-only", tier:"helper"}
  ],
  steps: [ /* see below */ ],
  moments: [{when:"01:30 · reviewer", title:"Short headline", text:"One or two plain sentences.", tone:"bad"}],
  open: [{title:"Nothing is pushed.", text:"19 commits sit on local branches."}]
}
```

**Agents.** `tier` is `lead` (one; shown on top), `worker` (cards in a row) or `helper` (small chips
below). `avatar` is `spark` for Claude, `ghost` for Kiro, or omit it and the agent gets its
initials. Do not draw or embed a product's real logo. An agent that is created during the session
should be the target of a `spawn` step; it stays hidden until then.

**Steps.** `t` is optional on every step (`"HH:MM"`). Each `label` is a tagline of at most seven
words.

| kind | fields | what the viewer sees |
|---|---|---|
| `send` | `from`, `to`, `label` | a pill flies from one agent to the other |
| | `starts: true` | the receiver starts working on it (spinning ring, status line) |
| | `ends: true` | the sender has finished and goes idle (use on a report coming back) |
| | `verdict: "ok"` or `"bad"` | the sender's check passed or failed; the pill is green or red, a helper chip gets a tick or cross, and the receiver shakes on a fail |
| | `tone: "ok"` or `"bad"` | colour the pill without it being a verdict (for example a list of fixes going back) |
| `check` | `who`, `label` | that agent's ring spins briefly, then its status shows the result |
| `spawn` | `by`, `who`, `label` | the spawned agent pops in and receives its first task |
| `output` | `who`, `label` | a dot drops under that agent (one commit, file or deliverable) |

A typical cycle: `send` lead to worker with `starts` (the brief), `send` worker to lead with `ends`
(the report), `check` by the lead, `spawn` or `send` to a helper, `send` helper to lead with a
`verdict`, on a fail a `send` lead to worker with `tone:"bad"` and `starts` (the fixes), then an
`output`.

## Rules

- **Only what happened.** Every agent, step, count and time must come from the conversation or from
  evidence you read. If something is your reconstruction, leave it out or mark the page (for
  example "times are approximate" in `when`). Never invent a verdict, a test count or a time.
- **No times is fine.** If the session has no reliable clock, omit `t` everywhere and set `when` to
  a plain description.
- **Keep it a flow, not a transcript.** One step per hand-off. Aim for 15 to 100 steps; merge
  routine repeats ("Tasks 3 to 4 built") and keep the turning points.
- **Taglines, not explanations.** Labels say what moved: "Day 2: seeding and the patch",
  "Fail · two saves, one lost", "129 of 129 tests pass". Detail belongs in `moments`.
- **Be honest in `open`.** List what was not verified, not pushed, not reviewed or is waiting on a
  decision. If everything is done and verified, say that in one item.
- **Leave private data out.** No secrets, tokens, credentials, customer data or personal details in
  labels. The page may be shared.
- **A single agent still works.** If only you acted, use one lead and model tools or the user as
  workers only if they really exchanged work; otherwise tell the user the session has no hand-offs
  to animate and offer a plain timeline.

## Notes on the template

- It loads the Motion library from jsDelivr and falls back to the browser's own animation API if
  that script is blocked, so the page works either way.
- It opens on the finished state and animates only when the viewer presses Replay. Speeds: 1×, 2×, 4×.
- Light and dark themes are both defined. It is responsive down to phone width.
- Worker colours are assigned in order; up to six distinct ones before they repeat.
