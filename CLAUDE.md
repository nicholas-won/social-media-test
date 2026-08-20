# Fastlane accounts in this project

This project's Fastlane MCP server (`.mcp.json`) authenticates with `Bearer ${FASTLANE_API_KEY}`. That single env var has pointed to **two different Fastlane workspaces** across sessions — never assume which one is currently loaded:

1. **`wonrepmax`** — personal fitness/travel/lifestyle brand. TikTok `@wonrepmax`, Instagram `won.repmax`. Angles: bench press, after-work training, travel, food, "off the clock" personality content.
2. **`pawlohq`** — the Pawlo app account (shared/household pet care coordination app). TikTok `@pawlohq`, Instagram `pawlohq`. Angles: fair-share pet care, invisible pet parent, who-fed-the-dog, medication tracking. See `pawlo-social-strategy.md` for the full strategy and workspace audit.

## Rule: verify before acting, don't assume

Before running any Fastlane write action (posting, scheduling, angle/preference changes, Blitz), confirm which workspace is actually live:

```bash
curl -sS "https://api.usefastlane.ai/api/v1/connections" \
  -H "Authorization: Bearer $FASTLANE_API_KEY" \
  -H "User-Agent: usefastlane-ai-agent/1.0"
```

- `platformUsername` containing `wonrepmax` → the fitness/travel workspace.
- `platformUsername` containing `pawlohq` → the Pawlo app workspace.

If the resolved workspace doesn't match what the task calls for, **stop and ask** rather than guessing or proceeding — do not silently act on the wrong account. If a key is pasted directly in conversation for a specific account, prefer passing it explicitly in that request rather than relying on `FASTLANE_API_KEY`, since that env var's value has changed between sessions before.
