# Pitbull Skills

Claude Code skills for the Pitbull DNA loyalty platform.

## Plugins

### `pitbull-dna-loyalty`

Domain context for the Pitbull DNA loyalty program. Covers:

- Tier system (Rookie / Alpha / Apex / Legend, PLN thresholds)
- Core models: User, Activity, LoyaltyAccount, PointTransaction, Store
- Services: TierService, LoyaltyService, MemberSnapshotService, OtpService
- Auth: JWT via `tymon/jwt-auth`, OTP SMS for Polish numbers
- API conventions: snake_case, no wrapper envelope
- V1 endpoints and test patterns
- Feature roadmap

---

## Installation

### Step 1 — Register the marketplace

Open `~/.claude/settings.json` and add `pitbull-skills` to `extraKnownMarketplaces`:

```json
{
  "extraKnownMarketplaces": {
    "pitbull-skills": {
      "source": {
        "source": "github",
        "repo": "dedicapps/pitbull-skills"
      }
    }
  }
}
```

> If you already have `extraKnownMarketplaces` (e.g. `pm-skills`), just add the `pitbull-skills` key alongside the existing ones.

### Step 2 — Enable the plugin

In the same `settings.json`, add the plugin to `enabledPlugins`:

```json
{
  "enabledPlugins": {
    "pitbull-dna-loyalty@pitbull-skills": true
  }
}
```

> Again, if you already have other plugins enabled, just add this key to the existing object.

### Full example `settings.json` snippet

```json
{
  "extraKnownMarketplaces": {
    "pitbull-skills": {
      "source": {
        "source": "github",
        "repo": "dedicapps/pitbull-skills"
      }
    }
  },
  "enabledPlugins": {
    "pitbull-dna-loyalty@pitbull-skills": true
  }
}
```

### Step 3 — Activate in your project's CLAUDE.md

Add the following line under the **Skills Activation** section in the project's `CLAUDE.md`:

```md
- `pitbull-dna-loyalty` — ALWAYS activate at the start of every session in this project.
  Contains full domain context for the Pitbull DNA loyalty platform.
```

### Step 4 — Restart Claude Code

Restart Claude Code (or open a new session) so it picks up the updated settings and fetches the plugin from GitHub.

---

## Updating the skill

1. Edit `pitbull-dna-loyalty/skills/pitbull-dna-loyalty/SKILL.md` in this repo
2. Commit and push to `main`
3. All team members get the update on their next Claude Code session

---

## Repos using this plugin

- [dedicapps/loyalty-api](https://github.com/dedicapps/loyalty-api) — Laravel backend
- [dedicapps/loyalty-app](https://github.com/dedicapps/loyalty-app) — React Native frontend
