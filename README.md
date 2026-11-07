# wizard-guide

A Claude Code skill for work only you can do: sign-ins, portal clicks, creating credentials, commands in
your own terminal and decisions. Claude shows one step at a time on a local HTML page, so the chat window
can stay small. You read the page, do the step, reply with a letter and refresh.

## When it applies

Claude reaches for it when a task needs two or more manual steps from you, including prerequisites and
credential setup you didn't ask to be guided through. It stays out of the way when Claude can do all the
work itself or you have only one thing to do. You can also ask directly:

> walk me through creating a GitHub fine-grained token, my chat window is tiny

## How it works

- **Claude does what it can first.** It checks whether a step is already done, and only leaves you the
  work it can't do itself.
- **The page carries the instructions.** Each step has its purpose, what to do, what to expect and lettered
  replies, with copy buttons for commands and values. Chat stays to a line or two per step.
- **Claude checks your reply.** Where it can, it verifies the result before marking a step as passed, and
  the final summary says what's still unproven.
- **Secrets stay out of the transcript.** You store them from your own terminal through a masked prompt.
  They shouldn't appear in chat or in the wizard's files.

The page has no server: it opens from disk and loads `wizard-data.js` on refresh. It can't send anything
back, so your replies still go through chat. The data format and step-writing rules are in
[SKILL.md](skills/wizard-guide/SKILL.md).

## Layout

```
skills/wizard-guide/SKILL.md            the instructions Claude follows
skills/wizard-guide/assets/wizard.html  the page, copied once per wizard
skills/wizard-guide/evals/              test prompts and what a good result looks like
```

Claude rewrites a small `wizard-data.js` beside the copied page for each step, so the page's markup and
styling never need regenerating.

## Install

Clone this repo, then link the skill into your Claude profile from the repo's root. On Windows this needs
Developer Mode or an elevated shell:

```powershell
New-Item -ItemType SymbolicLink -Path "$HOME\.claude\skills\wizard-guide" -Target "$PWD\skills\wizard-guide"
```

On macOS or Linux:

```bash
ln -s "$PWD/skills/wizard-guide" ~/.claude/skills/wizard-guide
```

Start a new Claude Code session to pick it up.

## Evals

Each folder under `evals/` is one case: a prompt and a set of graders. Run them from the repo's root:

```powershell
claude plugin eval skills/wizard-guide --no-publish --allow-tools "Write(//c/Users/<you>/AppData/Local/Temp/**)" "Edit(//c/Users/<you>/AppData/Local/Temp/**)"
```

The grants let the agents write the page to the system temp folder, which is where the skill puts it when
there's no scratchpad. Replace `<you>` with your Windows user name. Don't grant `Bash` on Windows: with no
sandbox available, the run is refused. That also means the step that opens the browser isn't tested.

This runs real agents on your credential, three runs per case by default plus a no-skill baseline, so use
`--case <name>` or `--runs 1` for a cheaper pass. `--no-publish` keeps the report local. Results go to
`skills/wizard-guide/evals/results/`, which is ignored by git.
