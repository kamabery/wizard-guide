---
name: wizard-guide
description: Guide the user through work only they can do - sign-ins, portal clicks, creating credentials, commands in their own terminal, decisions - one step at a time, on a local HTML wizard page beside a very small chat window. Use this whenever a task needs two or more manual steps from the user, or they say "step by step", "walk me through", "guide me", "wizard", "what do I need to run", "tell me what to do with the output", or mention limited space for the chat window. Also use it for prerequisites, account setup, credential creation or environment setup that Claude cannot finish alone, even if nobody asks for a wizard. Skip it when Claude can do all the work itself or the user has only one thing to do.
---

# Wizard guide

The user keeps a narrow chat window, so long instructions in chat are hard to follow. The browser page
carries the instructions, and chat carries only their reply. Each round is: read the page, do the step,
send a letter, refresh.

## 1. Plan the steps

Split the job into what only the user can do and what you can do yourself. That usually leaves the user
with sign-ins, portal screens, creating secrets and decisions.

- Do everything you can yourself, and look before you ask. Check whether a service is already running or
  a sign-in already exists, because a step you can skip is the best kind.
- Give each step one outcome. If its reply could mean two different things, split it.
- Look for steps that could damage something, such as a new credential replacing an old one. Check those
  first, and put a read-only check before the step that changes anything.

## 2. Open the page

1. Copy `assets/wizard.html` from this skill to `<wizard-folder>/wizard.html`. The folder is new and named
   for this wizard, inside the session's scratchpad directory if there is one, otherwise the system temp
   directory.
2. Write step 1 to `<wizard-folder>/wizard-data.js` (see "The data file").
3. Open `wizard.html` in the default browser: `Start-Process` on Windows, `open` on macOS, `xdg-open` on
   Linux. Give the user the full path in case it didn't open.

## 3. Run each step

For each step, rewrite `wizard-data.js`, then send a chat message of a line or two:

> Step 1 passed: the engine is running. Step 2 of 5 is ready. Refresh the page and reply with a letter.

Leave the commands and the reply options out of chat, since repeating them is what the page exists to
avoid. Assumptions, findings and caveats go on the page too, in `summary` or `note`, where they sit beside
the step they affect. Chat gets one extra line only when the user must not miss something and might not
see it on the page, such as the plan changing or something having gone wrong.

When a reply comes, check the result yourself wherever you can before marking the step as passed. A
reported "done" means the user did it, not that it worked. In `summary`, say what proved it, and tell the
user anything still unproven. If a check comes back odd, such as a 200 with empty fields, don't count it
until you understand it.

When the plan changes, update `steps` and say so in chat.

## Writing a step

Keep each page to about one screen in a narrow window: at most 200 words apart from commands, with a
`summary` of one or two sentences. The user should be able to read it and act without scrolling.

- **Only what this step needs.** Detail about later steps waits for those steps, and an assumption goes
  on the step it affects. Moving caveats out of chat doesn't mean moving all of them onto the page, so
  drop any that doesn't change what the user does next.
- **One decision per step.** If the user has to choose something and then act on it, make those two steps.
  Keep a read-only check as its own step too, ahead of the step that changes anything.
- **Say where to run things:** "in your own PowerShell window", for example, because a command run through
  Claude's prompt prefix lands in the conversation.
- **Keep commands on one line, chained with `;`.** Some terminals feed pasted lines in one at a time, so a
  prompt in the first line would swallow the second line as its input.
- **Say what the output should look like and what to do with it.**
- **Letter every reply.** `A` is the expected result, and the last letter is always a catch-all that asks
  what's on the screen. Say which details are safe to share.
- **Order copying with care:** copying a command overwrites the clipboard, so have the user copy the
  command first and any secret second.

**For decisions**, give what's being granted, the safeguards, the risk that remains and your
recommendation, in the step itself. Conditions the user approves are binding, so keep them somewhere that
outlasts the session if later work depends on them.

## Secrets

Keep secret values out of the conversation and out of the wizard's files:

- Never ask the user to paste a secret into chat, or to run a secret-handling command through Claude's
  prompt prefix. Transcripts persist.
- Never write a secret value into `wizard-data.js`.
- Have the user store it from their own terminal through a masked prompt, and print only something
  harmless such as its length. In PowerShell:

  ```powershell
  $t = Read-Host 'Token' -AsSecureString; $v = [Net.NetworkCredential]::new('', $t).Password; [Environment]::SetEnvironmentVariable('MY_TOKEN', $v, 'User'); "Stored $($v.Length) characters"; Remove-Variable t, v
  ```

  In bash, use `read -rs` and append to a file only they can read.
- Terminals opened earlier won't have a new user-level variable. Read it from the user's profile, or use a
  new terminal, and don't take a missing variable to mean it wasn't set.
- Verify by using the secret in a command that doesn't print it. Report metadata only.
- A credential that's shown only once should stay on screen until verification passes, so a bad paste can
  be redone.

## The data file

`wizard-data.js` assigns one object, and the page renders it as plain text, so commands can hold any
characters. In text fields, `**bold**` and `` `code` `` are the only formatting. Omit any field a step
doesn't need.

```js
window.WIZARD = {
  title: "HCP token setup",
  steps: ["Check existing token", "Create token", "Store it", "Verify"],
  current: 2,                         // 1-based; steps.length + 1 means finished
  summary: "Step 1 passed: the factory's token has a description.",
  heading: "Do this",                 // defaults to "Do this"
  actions: [
    { text: "Open the token page:", url: "https://example.com/tokens" },  // a clickable link
    { text: "Run this in your own PowerShell window:", command: "az account show" },
    { text: "Choose **Create a team token** and enter:" },
    { text: "Description", copy: "my-token-name" }  // a value with its own copy button
  ],
  note: "Leave the token on screen until step 4 passes.",
  replyIntro: "Send the letter that matches.",
  replies: [
    { key: "A", text: "the token was created and is showing." },
    { key: "B", text: "an error or anything else. Tell me the message, leaving out any token value." }
  ]
};
```

## Finishing

Set `current` to `steps.length + 1`, put the outcome in `summary`, and remove `actions` and `replies`. In
chat, give a short table of what's in place and what proved each part, what's still unproven, and the next
decision. This is the one message that can run longer.
