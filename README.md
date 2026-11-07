# wizard-guide

A Claude Code skill for work only you can do: sign-ins, portal clicks, creating credentials, commands in
your own terminal and decisions. Claude shows one step at a time on a local HTML page, so the chat window
can stay small. You read the page, do the step, reply with a letter and refresh.

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
