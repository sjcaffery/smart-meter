# Smart Meter

A Claude Code skill that runs quietly in the background and does three things on every response:

1. **Flags risk before it happens.** Before Claude sends anything, deletes records, touches a live sheet/CRM, or deploys, it rates how confident it actually is against a higher bar for anything that can't be undone. If it's not sure enough, it says so and stops rather than guessing - built and refined off real incidents.

2. **Hands bulk work down to cheaper models, and pulls the strongest model in to review.** When a task is bulk or mechanical grind (searching lots of files, batch edits, first drafts), Claude delegates it to a pinned cheaper-model subagent (`grind`/`worker`) to save budget - the model is fixed by the agent definition, so the grind can't silently run on the expensive tier. And before anything risky ships, it brings in the `reviewer` agent (pinned to the strongest model) to check it first, even if the grind ran cheap. This delegation runs in subagents; it does not change the main model you are driving - that stays your lever (see point 3).

3. **Tells you the cheapest model that still does the job properly, for the parts only you control.** Every reply ends with a one-line recommendation for which tier fits what you just asked - so you're not paying top-tier prices for routine work, or under-powering something that needed more. If the job's difficulty changes partway through a task, it'll stop and flag that too.

Net effect: fewer costly mistakes, real token savings happening automatically in the background, and a live recommendation for the one lever only you control - the main model you're running.

## Install (Claude Code, Windows/PowerShell)

1. Copy `skills/smart-meter/SKILL.md` to `%USERPROFILE%\.claude\skills\smart-meter\SKILL.md` (create the folders if needed).
2. Copy `hooks/smart-meter-reminder.ps1` to `%USERPROFILE%\.claude\hooks\smart-meter-reminder.ps1`.
3. Open `%USERPROFILE%\.claude\settings.json` (create it if it doesn't exist) and add this to the `UserPromptSubmit` hooks array:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe",
            "args": [
              "-NoProfile",
              "-ExecutionPolicy",
              "Bypass",
              "-File",
              "C:\\Users\\<you>\\.claude\\hooks\\smart-meter-reminder.ps1"
            ]
          }
        ]
      }
    ]
  }
}
```

If you already have other hooks in `settings.json`, add this as a new entry in the `UserPromptSubmit` array rather than replacing what's there, and swap `<you>` for your actual Windows username.

4. Restart Claude Code (or start a new session) so it picks up the new skill and hook.

That's it - every response from then on carries the Smart Meter read-out.
