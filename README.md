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

## Installation

Add to your `~/.claude/settings.json`:

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

## Usage

Add to your project's `CLAUDE.md` under **Skills Activation**:

```md
- `pitbull-dna-loyalty` — ALWAYS activate at the start of every session in this project.
```

## Repos using this plugin

- [dedicapps/loyalty-api](https://github.com/dedicapps/loyalty-api) — Laravel backend
- [dedicapps/loyalty-app](https://github.com/dedicapps/loyalty-app) — React Native frontend
