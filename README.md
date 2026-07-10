# Smart Meter

A Claude Code skill that runs quietly in the background and does three things on every response:

1. **Flags risk before it happens.** Before Claude sends anything, deletes records, touches a live sheet/CRM, or deploys, it rates how confident it actually is against a higher bar for anything that can't be undone. If it's not sure enough, it says so and stops rather than guessing - built and refined off real incidents.

2. **Automatically hands off work to the right tier - no clicking required.** When a task involves bulk or mechanical grind (searching lots of files, batch edits, first drafts), it silently delegates that to a cheaper, faster model behind the scenes to save budget. And before anything risky ships, it automatically brings in the strongest model to review it first, even if the grind work ran cheap. This happens on its own, mid-task, without you doing anything.

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
