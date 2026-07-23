# Crypto Watchlist Alerts — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for tracking crypto prices with customizable alerts (price thresholds and percent moves), on-demand price checks, optional morning summaries, and quiet hours. The bot admin can view usage stats and top alerts while respecting user privacy.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual crypto investors
- Telegram power users
- price alert enthusiasts

## Success criteria

- Users can manage watchlists with 50+ tickers
- Alerts trigger reliably with cooldowns
- Morning summaries deliver at user-selected times
- Admin can view real-time metrics dashboard

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with watchlist management and settings
- **Add to watchlist** (button, actor: user, callback: watchlist:add) — Open ticker selection dialog with inline buttons and freeform input
  - inputs: ticker symbol, display name
  - outputs: confirmation message, updated watchlist
- **Set alert** (button, actor: user, callback: alert:configure) — Trigger conversational flow for price threshold or percent move alerts
  - inputs: alert type, value
  - outputs: confirmation message, alert rules
- **/price** (command, actor: user, command: /price) — Request current price of specified ticker or full watchlist
  - inputs: ticker symbol, all
  - outputs: price data with 24h change
- **Configure quiet hours** (button, actor: user, callback: settings:quiet_hours) — Open time selection interface
  - inputs: start time, end time
  - outputs: updated quiet hours
- **/admin** (command, actor: admin, command: /admin) — Show owner-only metrics dashboard
  - inputs: ADMIN_CHAT_ID
  - outputs: user count, top alerts

## Flows

### Onboarding
_Trigger:_ /start

1. Explain core features
2. Set default quiet hours (23:00-07:00)
3. Disable morning summary by default

_Data touched:_ User

### Alert triggering
_Trigger:_ Price update event

1. Check all user watchlists
2. Apply quiet hours filter
3. Queue alerts if suppressed
4. Send triggered alerts
5. Apply cooldown periods

_Data touched:_ WatchlistEntry, AlertEvent

### Morning summary
_Trigger:_ User-selected local time

1. Generate price summary
2. Include recent percent moves
3. Add cooldown status notes
4. Send only if enabled

_Data touched:_ User, WatchlistEntry

### Admin metrics
_Trigger:_ /admin

1. Verify admin ID
2. Aggregate user count
3. Calculate top 10 alerting tickers
4. Format dashboard

_Data touched:_ User, AlertEvent

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — User preferences and state
  - fields: telegram_id, watchlist, alert_rules, quiet_hours, morning_summary_time, last_known_prices, cooldown_state
- **WatchlistEntry** _(retention: persistent)_ — Tracked cryptocurrency
  - fields: ticker, display_name, enabled, price_threshold, percent_move_alerts
- **AlertEvent** _(retention: persistent)_ — Triggered alert record
  - fields: user_id, coin, old_price, new_price, percent_change, timestamp
- **OwnerMetrics** _(retention: persistent)_ — Administrative analytics
  - fields: total_users, alert_counts_by_ticker

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View user count and top alerts via /admin command
- Configure admin chat ID in settings

## Notifications

- Price alert messages with coin details
- Morning summary with watchlist prices
- Quiet hour alert summaries
- Admin metrics dashboard updates

## Permissions & privacy

- All user data is private and not shared
- Admin view only accessible to configured owner
- Watchlist contents remain confidential

## Edge cases

- Unknown tickers handled with suggestions
- Price feed failures retried up to 3 times
- Queued alerts during quiet hours summarized later
- Watchlist size capped at 50 tickers

## Required tests

- End-to-end alert triggering with cooldown
- Morning summary delivery at user-selected time
- Quiet hours suppression and summary
- Admin metrics dashboard accuracy

## Assumptions

- Crypto price API integration will be implemented
- Users will manage watchlists through buttons and commands
- Admin will configure their own chat ID
