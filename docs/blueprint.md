# CryptoWatch — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for personalized crypto price alerts and summaries. Users track tickers via buttons or typed input, set threshold/percent alerts, manage quiet hours, and receive morning summaries. Admin view shows usage stats and alert trends.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual crypto traders
- Casual price watchers
- Telegram power users

## Success criteria

- Users can setup and manage watchlists with alerts
- Admin receives accurate usage statistics
- Alerts respect quiet hours and cooldown rules

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Initialize user profile and show onboarding flow
- **BTC** (button, actor: user, callback: add_ticker:BTC) — Quick-add Bitcoin to watchlist
  - inputs: none
  - outputs: watchlist item
- **Add custom ticker** (button, actor: user, callback: add_custom_ticker) — Prompt for arbitrary ticker symbol input
- **/price** (command, actor: user, command: /price) — Show current prices for watchlist or specific ticker
- **/admin** (command, actor: owner, command: /admin) — Show admin statistics and alert history

## Flows

### Onboarding
_Trigger:_ /start

1. Request currency preference
2. Request timezone
3. Show quick-add buttons

_Data touched:_ user profile

### Alert management
_Trigger:_ Add threshold alert

1. Request alert direction
2. Request target price
3. Confirm rule

_Data touched:_ watchlist item

### Morning summary
_Trigger:_ Scheduled time

1. Fetch prices
2. Format summary with changes
3. Send to user

_Data touched:_ price snapshot

### Admin view
_Trigger:_ /admin

1. Show active users
2. Show top alerts
3. Show recent events

_Data touched:_ owner stats

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User profile** _(retention: persistent)_ — User preferences and settings
  - fields: telegram_id, currency, timezone, quiet_hours, summary_time
- **Watchlist item** _(retention: persistent)_ — Tracked crypto ticker with alert rules
  - fields: ticker, alert_rules, last_alert_time
- **Price snapshot** _(retention: session)_ — Current price data for comparisons
  - fields: price, timestamp
- **Alert event** _(retention: persistent)_ — Triggered alert details
  - fields: user_id, ticker, alert_type, old_price, new_price
- **Owner stats** _(retention: persistent)_ — Administrative metrics
  - fields: active_users, alert_counts, recent_events

## Integrations

- **Telegram** (required) — Bot API messaging
- **Price feed API** (required) — Crypto price data
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /admin command access
- Real-time alert notifications toggle
- Quiet hour override

## Notifications

- Price alert notifications with full price context
- Morning summary with watchlist status
- Admin alert event notifications (configurable)

## Permissions & privacy

- User data stored with telegram_id as key
- No third-party data sharing
- Price data cached temporarily

## Edge cases

- Invalid ticker suggestions
- Overlapping alert cooldowns
- Quiet hours during alert window

## Required tests

- End-to-end alert trigger with price change
- Summary formatting with multiple currencies
- Admin stats accuracy validation

## Assumptions

- Price feed has 99%+ uptime
- Users understand crypto ticker conventions
- Admin will manually verify price feed reliability
