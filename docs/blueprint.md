# Restaurant Booking Bot — Bot specification

**Archetype:** booking

**Voice:** friendly and professional — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot for restaurant reservations that shows real-time available slots, confirms bookings with reference codes, sends reminders, and provides admin views for owners. Guests can reschedule/cancel via inline buttons while the bot optimizes table allocation to maximize capacity.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- restaurant staff/owner
- Telegram users seeking reservations

## Success criteria

- 100% accurate slot availability without double-booking
- 95% guest confirmation rate with reference codes
- Real-time admin visibility of bookings and capacity

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with booking options
- **Book a table** (button, actor: user, callback: booking:start) — Initiates reservation flow with calendar and time selection
- **View my booking** (button, actor: user, callback: booking:view) — Displays current booking details with reschedule/cancel buttons
- **/admin** (command, actor: admin, command: /admin) — Opens owner admin dashboard (requires admin auth)

## Flows

### Guest booking flow
_Trigger:_ /start or 'Book a table' button

1. Date selection (calendar UI)
2. Available time slots (15-min increments)
3. Party size selection
4. Name/phone input
5. Table allocation confirmation with reference code

_Data touched:_ booking, tables, opening_hours

### Rescheduling flow
_Trigger:_ Reschedule button from booking view

1. New date selection
2. Available time slots for new date
3. Confirm reschedule with updated allocation

_Data touched:_ booking, tables

### Admin dashboard
_Trigger:_ /admin command

1. Today's bookings summary
2. Remaining capacity by time block
3. Mark no-show actions

_Data touched:_ booking, tables

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **booking** _(retention: persistent)_ — Reservation records with status tracking
  - fields: id, reference_code, guest_name, phone, party_size, datetime, tables_allocated, status, created_at
- **tables** _(retention: persistent)_ — Restaurant table inventory with seat counts
  - fields: table_id, seats
- **opening_hours** _(retention: persistent)_ — Daily operating hours by weekday
  - fields: weekday, start_time, end_time
- **reminder_schedule** _(retention: persistent)_ — Reminder timing configuration
  - fields: default_offset_minutes

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Set admin chat ID for notifications
- Configure opening hours and table inventory
- Adjust sitting length and reminder timing
- Mark bookings as no-show

## Notifications

- Pre-booking reminders (2h default)
- Admin daily summary at opening time
- New booking confirmation alerts

## Permissions & privacy

- Guest data stored privately and only visible to admins
- No third-party data sharing

## Edge cases

- Partial guest input requiring clarification
- No-show marking during active bookings
- Conflicting table allocations during rescheduling

## Required tests

- End-to-end booking flow with real-time availability checks
- Admin no-show marking workflow
- Reminder message delivery timing

## Assumptions

- Default 11:00-22:00 opening hours if unconfigured
- Default 90-minute sitting length
- Default 5-table inventory if unconfigured
