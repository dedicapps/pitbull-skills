---
name: pitbull-dna-loyalty
description: Use when working on the Pitbull DNA loyalty API (loyalty-api project). Covers program rules, tier system, models, services, auth, API conventions, and planned features.
---

# Pitbull DNA Loyalty Program

## What This Is

**Pitbull DNA** is a loyalty program for a Polish clothing brand (Pitbull). Members earn tier status based on annual spending in PLN across four purchase channels. The backend is a Laravel 13 REST API consumed by a React Native (Expo) mobile app.

---

## Tier System

4 tiers based on **annual PLN value** (sum of `contribution_pln` from activities with `status = 'progress'`):

| Level | Name    | Threshold   |
|-------|---------|-------------|
| 1     | Rookie  | 0 PLN       |
| 2     | Alpha   | 1 200 PLN   |
| 3     | Apex    | 3 000 PLN   |
| 4     | Legend  | 6 000 PLN   |

Config key: `loyalty.tier_thresholds`, `loyalty.tier_names`

**Programme year type** (`loyalty.programme_year_type`): `calendar` (Jan–Dec) or `rolling` (last 12 months). Default: `calendar`.

---

## Authentication

- **Method:** JWT via `tymon/jwt-auth`. All member routes use `auth:api` middleware.
- **OTP Flow:** Polish phone numbers (+48, normalized to digits only e.g. `48123456789`)
  1. `POST /v1/auth/otp/request` → sends SMS, returns `request_id` + `expires_at` (2 min)
  2. `POST /v1/auth/otp/verify` → validates code + request_id → returns JWT token
  3. `POST /v1/auth/otp/resend` → resends OTP
  4. `POST /v1/auth/signout` → invalidates token
- **SMS Provider:** Custom `SmsApi` channel (`app/Channels/SmsApi/`), not Twilio/Vonage.
- **Dev mode:** OTP response includes `dev_code` when `APP_ENV !== 'production'`.

---

## Core Models

| Model               | Key Fields                                                              |
|---------------------|-------------------------------------------------------------------------|
| `User`              | `phone` (unique, normalized), `member_code` (XXXX-XXXX-PIT-DNA), `first_name`, `last_name`, `home_hq`, `joined_at` |
| `OtpRequest`        | `phone`, `code` (6 digits), `expires_at`, `verified_at`               |
| `Activity`          | `user_id`, `channel`, `title`, `date`, `contribution_pln` (nullable), `meta`, `status` |
| `LoyaltyAccount`    | `user_id`, `balance` (points)                                           |
| `PointTransaction`  | `loyalty_account_id`, `type` (earn/redeem), `points`, `description`, `meta` |
| `Store`             | `name`, `address`, `latitude`, `longitude`, `is_active`               |
| `MemberPreference`  | `user_id`, `notifications_enabled`, `regional_hq`                     |

**Activity channels:** `Retail` | `E-commerce` | `Event` | `Omnichannel`
**Activity statuses:** `progress` (monetary, counts toward tier) | `recognition` (events/non-monetary) | `access`

---

## Core Services

| Service                  | Responsibility                                                           |
|--------------------------|--------------------------------------------------------------------------|
| `TierService`            | `levelFor(float)`, `nameFor(int)`, `labelFor(int)`, `progressToNextLevel()`, `currentThreshold()`, `nextThreshold()` |
| `LoyaltyService`         | `earn(User, int, ?string)`, `redeem(User, int, ?string)`, `getOrCreateAccount(User)` — throws `InsufficientPointsException` |
| `MemberSnapshotService`  | `annualValue(User)`, `buildProgress(User)`, `buildContextualBenefits(int)`, `buildLifecycleMessages(float, int)` |
| `OtpService`             | OTP generation, SMS dispatch, verification                              |

---

## API Conventions

- All routes: `routes/api.php` under `Route::prefix('v1')`
- All responses: snake_case JSON (frontend converts to camelCase)
- **No wrapper envelope** on success responses EXCEPT `GET /member/activity` uses `{ "data": [...] }`
- Controllers: `app/Http/Controllers/Api/V1/`
- Resources: `app/Http/Resources/`
- Error format (standard Laravel validation):
```json
{ "message": "...", "errors": { "field": ["..."] } }
```
- OpenAPI annotations via `Dedaco/Scramble` — use `#[Group(...)]` on controllers

---

## V1 Endpoints

**Public:**
- `POST /v1/auth/otp/request`
- `POST /v1/auth/otp/verify`
- `POST /v1/auth/otp/resend`
- `GET /v1/guest/info` → tier thresholds, programme name

**Authenticated (`auth:api`):**
- `POST /v1/auth/signout`
- `GET /v1/member/snapshot` → full dashboard state (profile, progress, benefits, lifecycle messages)
- `GET /v1/member/activity` → transaction/activity history
- `GET /v1/member/identity-card` → QR card data (member_code, qr_value, tier)
- `GET /v1/member/nearby-hq` → nearest HQ location
- `PUT /v1/member/preferences` → update notification/regional prefs

---

## Test Patterns

```php
// Auth helper
$user = User::factory()->create();
$token = JWTAuth::fromUser($user);
$this->withToken($token)->getJson('/api/v1/member/snapshot');

// Database refresh
uses(LazilyRefreshDatabase::class);

// Run tests
php artisan test --compact --filter=ClassName
```

---

## Feature Roadmap (in priority order)

| # | Feature                   | Status  |
|---|---------------------------|---------|
| 1 | Points API (balance, earn, redeem) | Planned — plan: `docs/superpowers/plans/2026-04-03-01-points-api.md` |
| 2 | Stores Endpoint           | Planned — plan: `docs/superpowers/plans/2026-04-03-02-stores-endpoint.md` |
| 3 | Tier Protection & Rollover (30-day grace, Jan 1 via `tier:evaluate` command) | Planned — plan: `docs/superpowers/plans/2026-04-03-03-tier-protection.md` |
| 4 | Referral Program          | Medium priority |
| 5 | Birthday Credits          | Medium priority |
| 6 | Challenges / Missions     | Low priority |
| 7 | POS Integration           | Low priority |

---

## HQ Locations

Initial support: Warsaw HQ (`Chmielna 19, Warsaw`). `GET /member/nearby-hq` accepts optional `?lat=&lng=` for future geo distance.

---

## Key Business Rules

- Phone stored as digits only (no `+`, no spaces): `48123456789`
- `member_code` format: `XXXX-XXXX-PIT-DNA`
- QR value format: `pitbull-dna://member/{member_code}`
- `tier_protected_until` / `protected_tier_level` (planned): 30-day grace on loyalty_accounts after year-end evaluation
- `wallet_pass_available`: hardcoded `false` until Apple/Google Wallet is implemented
- Tier level is **computed** from Activity, not stored
