# PRODUCT_REQUIREMENTS.md

**120 Army — Global Christian Prayer & Discipleship Platform**
Version 1.0 | Last updated: 2026-06-08

---

## Table of Contents

1. [Authentication](#1-authentication)
2. [User Profiles](#2-user-profiles)
3. [Prayer Wall](#3-prayer-wall)
4. [Prayer Requests](#4-prayer-requests)
5. [Prayer Tracking](#5-prayer-tracking)
6. [Global Prayer Globe](#6-global-prayer-globe)
7. [Friend System](#7-friend-system)
8. [Messaging](#8-messaging)
9. [Notifications](#9-notifications)
10. [Righteous Radio](#10-righteous-radio)
11. [Kingdom Partner Subscription](#11-kingdom-partner-subscription)
12. [Donations](#12-donations)
13. [Christian Short Videos](#13-christian-short-videos)
14. [Events](#14-events)
15. [Bible Study Groups](#15-bible-study-groups)
16. [Testimonies](#16-testimonies)
17. [Admin Dashboard](#17-admin-dashboard)
18. [Analytics](#18-analytics)

---

## 1. Authentication

### Purpose
Secure, low-friction identity management that supports both spiritual anonymity (for sensitive prayer requests) and verified community membership.

### User Stories
- As a new visitor, I can sign up with email/password or via Google/Apple SSO so I can join quickly.
- As a returning user, I can log in with biometric authentication (Face ID / fingerprint) on mobile so I don't have to type a password.
- As a user who forgot my password, I can reset it via email link within 5 minutes.
- As a user, I can enable 2FA via authenticator app or SMS so my account stays secure.
- As an admin, I can suspend or delete any user account and see an audit log of the action.
- As a user, I can log out of all devices simultaneously for security.

### Features
- Email + password registration with email verification
- Google OAuth 2.0 and Apple Sign-In
- Biometric authentication (WebAuthn / passkeys)
- Two-factor authentication (TOTP + SMS fallback)
- Password reset via time-limited signed email link
- Session management: view and revoke active sessions
- Account deletion (GDPR-compliant, 30-day grace period)
- Rate limiting on login attempts (5 attempts → 15-minute lockout)
- JWT access tokens (15-min expiry) + rotating refresh tokens (30-day expiry)
- Role system: `user`, `moderator`, `admin`, `super_admin`

### Database Requirements

```sql
-- users
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
email           TEXT UNIQUE NOT NULL
email_verified  BOOLEAN DEFAULT false
password_hash   TEXT
display_name    TEXT NOT NULL
avatar_url      TEXT
role            TEXT DEFAULT 'user'   -- user | moderator | admin | super_admin
is_active       BOOLEAN DEFAULT true
is_anonymous    BOOLEAN DEFAULT false -- for anonymous prayer posters
created_at      TIMESTAMPTZ DEFAULT now()
updated_at      TIMESTAMPTZ DEFAULT now()

-- sessions
id              UUID PRIMARY KEY
user_id         UUID REFERENCES users(id) ON DELETE CASCADE
refresh_token   TEXT UNIQUE NOT NULL
device_info     JSONB
ip_address      INET
expires_at      TIMESTAMPTZ NOT NULL
created_at      TIMESTAMPTZ DEFAULT now()

-- oauth_accounts
id              UUID PRIMARY KEY
user_id         UUID REFERENCES users(id) ON DELETE CASCADE
provider        TEXT NOT NULL   -- google | apple
provider_uid    TEXT NOT NULL
UNIQUE(provider, provider_uid)

-- two_factor_auth
id              UUID PRIMARY KEY
user_id         UUID REFERENCES users(id) ON DELETE CASCADE
secret          TEXT NOT NULL   -- encrypted TOTP secret
backup_codes    TEXT[]          -- hashed backup codes
enabled_at      TIMESTAMPTZ
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/auth/register` | None | Create account |
| POST | `/auth/login` | None | Email/password login |
| POST | `/auth/oauth/:provider` | None | OAuth callback |
| POST | `/auth/refresh` | Refresh token | Rotate access token |
| POST | `/auth/logout` | Bearer | Revoke current session |
| POST | `/auth/logout-all` | Bearer | Revoke all sessions |
| POST | `/auth/password-reset/request` | None | Send reset email |
| POST | `/auth/password-reset/confirm` | None | Apply new password |
| POST | `/auth/2fa/setup` | Bearer | Generate TOTP secret |
| POST | `/auth/2fa/verify` | Bearer | Confirm TOTP code |
| DELETE | `/auth/2fa` | Bearer | Disable 2FA |
| GET | `/auth/sessions` | Bearer | List active sessions |
| DELETE | `/auth/sessions/:id` | Bearer | Revoke session |

---

## 2. User Profiles

### Purpose
Rich, spiritually-contextualised identities that help believers connect authentically — expressing faith journeys, gifts, denominations, and prayer focus areas.

### User Stories
- As a user, I can set my display name, avatar, bio, denomination, and spiritual gifts so others understand my background.
- As a user, I can specify my country and city so I appear on the global prayer map.
- As a user, I can list my prayer focus areas (e.g., missions, healing, government) so like-minded intercessors can find me.
- As a user, I can view another user's public profile, see their prayer activity stats, and send a friend request.
- As a user, I can control which profile fields are public, friends-only, or private.
- As a user, I can see my own prayer statistics (prayers submitted, prayers prayed, testimonies posted).

### Features
- Avatar upload with automatic crop/resize (stored in object storage)
- Cover photo / banner image
- Bio (500 character max)
- Denomination selector (searchable list + "Other" free text)
- Spiritual gifts multi-select (1 Corinthians 12 list)
- Prayer focus areas multi-select (up to 10)
- Location (country required, city optional)
- Website / social links (up to 5)
- Privacy controls per field (public / friends / private)
- Profile badges (e.g., Kingdom Partner, Intercessor, Group Leader)
- Prayer stats widget on profile (total prayers, prayer streak, testimonies)
- "Praying for" widget showing active prayer requests
- Follow / friend button
- Block user action

### Database Requirements

```sql
-- profiles (1-to-1 with users)
id                  UUID PRIMARY KEY REFERENCES users(id)
bio                 TEXT
cover_url           TEXT
denomination        TEXT
spiritual_gifts     TEXT[]
prayer_focus_areas  TEXT[]
country_code        CHAR(2)
city                TEXT
lat                 NUMERIC(9,6)
lng                 NUMERIC(9,6)
website_url         TEXT
social_links        JSONB   -- { platform: url }
privacy_settings    JSONB   -- { field: 'public' | 'friends' | 'private' }
prayer_streak       INT DEFAULT 0
total_prayers_sent  INT DEFAULT 0
total_prayers_prayed INT DEFAULT 0
last_active_at      TIMESTAMPTZ

-- badges
id          UUID PRIMARY KEY
slug        TEXT UNIQUE NOT NULL
label       TEXT NOT NULL
icon_url    TEXT
description TEXT

-- user_badges
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
badge_id    UUID REFERENCES badges(id)
awarded_at  TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (user_id, badge_id)
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/users/:id` | Optional | Get public profile |
| GET | `/users/me` | Bearer | Get own profile |
| PATCH | `/users/me` | Bearer | Update profile fields |
| POST | `/users/me/avatar` | Bearer | Upload avatar |
| POST | `/users/me/cover` | Bearer | Upload cover photo |
| GET | `/users/:id/stats` | Optional | Prayer stats |
| GET | `/users/:id/badges` | Optional | User badges |
| POST | `/users/:id/block` | Bearer | Block user |
| DELETE | `/users/:id/block` | Bearer | Unblock user |
| GET | `/users/search` | Bearer | Search by name/location/gift |

---

## 3. Prayer Wall

### Purpose
A real-time, community-wide feed of prayer requests — the spiritual heart of the platform, designed to feel like entering a sacred communal prayer space rather than a social media feed.

### User Stories
- As a user, I can see a live feed of prayer requests from the global community and from my friends.
- As a user, I can filter the wall by category, location, language, or recency.
- As a user, I can tap "I'm Praying" on any request to encourage the poster and log my prayer.
- As a user, I can leave a short encouragement comment on a prayer request.
- As a user, I can share a prayer request to my friends or copy a link.
- As a user, I want urgent/critical requests (e.g., medical emergencies) to be visually distinguished.

### Features
- Infinite scroll feed with real-time WebSocket updates
- Feed tabs: Global / Friends / Local (by country) / Trending
- Filter bar: category, language, urgency, recency
- "I'm Praying" button with animated count increment
- Encouragement comments (280 chars max, threaded one level deep)
- Request cards show: poster avatar/name (or "Anonymous"), category badge, prayer count, time ago, urgency indicator
- Pinned requests (admin/moderator curated)
- Trending algorithm: weighted score of prayer count + recency + comment activity
- Content moderation: report button, auto-flag via keyword filter, moderator review queue
- Anonymous posting option (name hidden, avatar replaced with generic icon)

### Database Requirements

```sql
-- prayer_requests (see Module 4 for full schema)
-- prayer_wall_pins
id          UUID PRIMARY KEY
request_id  UUID REFERENCES prayer_requests(id) ON DELETE CASCADE
pinned_by   UUID REFERENCES users(id)
position    INT DEFAULT 0
expires_at  TIMESTAMPTZ
created_at  TIMESTAMPTZ DEFAULT now()

-- prayer_wall_reports
id          UUID PRIMARY KEY
request_id  UUID REFERENCES prayer_requests(id) ON DELETE CASCADE
reporter_id UUID REFERENCES users(id)
reason      TEXT NOT NULL   -- spam | offensive | misinformation | other
notes       TEXT
status      TEXT DEFAULT 'pending'  -- pending | reviewed | dismissed | actioned
created_at  TIMESTAMPTZ DEFAULT now()
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/wall` | Optional | Paginated wall feed |
| GET | `/wall/trending` | Optional | Trending requests |
| GET | `/wall/friends` | Bearer | Friends' requests |
| GET | `/wall/local` | Optional | Country-filtered feed |
| POST | `/wall/:requestId/pray` | Bearer | Log "I'm Praying" |
| DELETE | `/wall/:requestId/pray` | Bearer | Undo prayer |
| POST | `/wall/:requestId/report` | Bearer | Report content |
| GET | `/wall/pins` | None | Get pinned requests |

---

## 4. Prayer Requests

### Purpose
The atomic unit of the platform — structured, categorised prayer needs submitted by users, designed to be actionable for intercessors and trackable over time.

### User Stories
- As a user, I can submit a prayer request with a title, description, category, urgency level, and optional image.
- As a user, I can post anonymously so sensitive requests (health, marriage, addiction) feel safe to share.
- As a user, I can set an expiry date or mark my request as answered.
- As a user, I can edit or delete my own requests.
- As a user, I can see how many people have prayed for my request and read their encouragements.
- As a user, I can re-open a prayer request if the need continues.

### Features
- Rich text description (1,000 chars max)
- Category taxonomy: Health, Family, Finance, Career, Relationships, Nations, Salvation, Healing, Protection, Guidance, Praise, Other
- Urgency levels: Normal, Urgent, Critical
- Optional image attachment
- Anonymous mode toggle
- Expiry date (default 30 days, renewable)
- "Mark as Answered" action → triggers testimony prompt
- Prayer count, comment count, share count
- Edit history (last 3 versions stored)
- Request visibility: Public / Friends only / Group only (when posted in a study group)
- Tags (freeform, up to 5)
- Language auto-detection + manual override

### Database Requirements

```sql
-- prayer_requests
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id         UUID REFERENCES users(id) ON DELETE SET NULL
is_anonymous    BOOLEAN DEFAULT false
title           TEXT NOT NULL
body            TEXT NOT NULL
category        TEXT NOT NULL
urgency         TEXT DEFAULT 'normal'  -- normal | urgent | critical
image_url       TEXT
tags            TEXT[]
language        CHAR(5) DEFAULT 'en'
visibility      TEXT DEFAULT 'public'  -- public | friends | group
group_id        UUID REFERENCES study_groups(id)
status          TEXT DEFAULT 'active'  -- active | answered | expired | removed
prayer_count    INT DEFAULT 0
comment_count   INT DEFAULT 0
share_count     INT DEFAULT 0
expires_at      TIMESTAMPTZ
answered_at     TIMESTAMPTZ
created_at      TIMESTAMPTZ DEFAULT now()
updated_at      TIMESTAMPTZ DEFAULT now()

-- prayer_request_versions
id          UUID PRIMARY KEY
request_id  UUID REFERENCES prayer_requests(id) ON DELETE CASCADE
title       TEXT
body        TEXT
edited_at   TIMESTAMPTZ DEFAULT now()
edited_by   UUID REFERENCES users(id)

-- prayer_comments
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
request_id  UUID REFERENCES prayer_requests(id) ON DELETE CASCADE
user_id     UUID REFERENCES users(id) ON DELETE SET NULL
parent_id   UUID REFERENCES prayer_comments(id)  -- one level threading
body        TEXT NOT NULL   -- 280 char max enforced in app
is_anonymous BOOLEAN DEFAULT false
created_at  TIMESTAMPTZ DEFAULT now()
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/requests` | Bearer | Create prayer request |
| GET | `/requests/:id` | Optional | Get single request |
| PATCH | `/requests/:id` | Bearer (owner) | Edit request |
| DELETE | `/requests/:id` | Bearer (owner) | Delete request |
| POST | `/requests/:id/answer` | Bearer (owner) | Mark as answered |
| POST | `/requests/:id/renew` | Bearer (owner) | Extend expiry |
| GET | `/requests/:id/comments` | Optional | Paginated comments |
| POST | `/requests/:id/comments` | Bearer | Post comment |
| DELETE | `/requests/:id/comments/:cid` | Bearer | Delete comment |
| GET | `/users/:id/requests` | Optional | User's requests |

---

## 5. Prayer Tracking

### Purpose
Personal intercessory discipline — helping users build consistent prayer habits, track prayers prayed for others, and see answered prayers over time.

### User Stories
- As a user, I can maintain a personal prayer list of requests I'm committed to praying for regularly.
- As a user, I can log each time I pray for a specific request so I can track my faithfulness.
- As a user, I can see my current prayer streak and receive encouragement to maintain it.
- As a user, I can set daily prayer reminders at a time I choose.
- As a user, I can view a calendar heat-map of my prayer activity over the past year.
- As a user, I can see which of my saved prayers have been answered.

### Features
- Personal prayer list (save any public request + add private items)
- Private prayer items (not shared publicly)
- "Log Prayer" action per item with optional note
- Daily prayer streak counter (resets at midnight user-local time)
- Streak milestones: 7, 30, 100, 365 days → badge awards
- Calendar heat-map (GitHub-style) of prayer activity
- Daily prayer reminder (push notification + email, user-configurable time)
- Weekly summary notification: "You prayed X times this week"
- Answered prayers archive
- Prayer journal: attach private notes to any prayer session

### Database Requirements

```sql
-- prayer_list_items
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id         UUID REFERENCES users(id) ON DELETE CASCADE
request_id      UUID REFERENCES prayer_requests(id) ON DELETE SET NULL  -- NULL = private item
title           TEXT    -- for private items
notes           TEXT
reminder_time   TIME    -- local time for daily reminder
is_private      BOOLEAN DEFAULT false
position        INT DEFAULT 0
created_at      TIMESTAMPTZ DEFAULT now()

-- prayer_logs
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id         UUID REFERENCES users(id) ON DELETE CASCADE
item_id         UUID REFERENCES prayer_list_items(id) ON DELETE CASCADE
journal_note    TEXT
prayed_at       TIMESTAMPTZ DEFAULT now()

-- prayer_streaks
user_id         UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE
current_streak  INT DEFAULT 0
longest_streak  INT DEFAULT 0
last_prayed_at  DATE
total_days      INT DEFAULT 0
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/tracking/list` | Bearer | Get prayer list |
| POST | `/tracking/list` | Bearer | Add item to list |
| PATCH | `/tracking/list/:id` | Bearer | Update item |
| DELETE | `/tracking/list/:id` | Bearer | Remove from list |
| POST | `/tracking/list/:id/log` | Bearer | Log a prayer |
| GET | `/tracking/logs` | Bearer | Prayer log history |
| GET | `/tracking/streak` | Bearer | Current streak info |
| GET | `/tracking/heatmap` | Bearer | Year calendar data |
| PATCH | `/tracking/reminder` | Bearer | Set reminder time |

---

## 6. Global Prayer Globe

### Purpose
A real-time 3D visualisation of prayer activity worldwide — communicating the global scale and unity of the 120 Army community in a visually stunning, emotionally resonant way.

### User Stories
- As a visitor, I can see a live 3D globe showing where prayers are being sent and received right now.
- As a user, I can click on a country to see the number of active prayer requests and intercessors.
- As a user, I can see animated arcs tracing prayers being prayed across countries in real-time.
- As a user, I can toggle between "Prayer Sent" and "Intercessors Active" view modes.
- As a user on mobile, I can interact with the globe by rotating and pinching to zoom.

### Features
- Globe.gl powered 3D interactive globe
- Real-time prayer arcs: animated lines from intercessor → request origin country
- Country heat-map overlay: colour intensity based on prayer activity
- Live counter: total prayers in last 24h globally
- Country drill-down panel: top requests, active intercessors count
- Pulse animations at active prayer hotspots
- Day/night terminator line overlay
- Toggle: prayer activity / intercessor density / answered prayers
- Auto-rotate when idle; disable on user interaction
- WebSocket feed for live arc updates (throttled to max 20 arcs/sec for performance)

### Database Requirements

```sql
-- globe_activity (time-series, short TTL — purge > 48h)
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
event_type      TEXT NOT NULL    -- prayer_sent | prayer_prayed | request_posted
from_country    CHAR(2)
to_country      CHAR(2)
lat             NUMERIC(9,6)
lng             NUMERIC(9,6)
created_at      TIMESTAMPTZ DEFAULT now()

-- country_stats (materialised, refreshed every 5 min)
country_code    CHAR(2) PRIMARY KEY
active_requests INT DEFAULT 0
active_users    INT DEFAULT 0
prayers_24h     INT DEFAULT 0
updated_at      TIMESTAMPTZ DEFAULT now()
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/globe/stats` | None | Global summary stats |
| GET | `/globe/countries` | None | Per-country activity data |
| GET | `/globe/country/:code` | None | Single country drill-down |
| GET | `/globe/live` | None | SSE stream of live arcs |
| WS | `/globe/stream` | None | WebSocket arc feed |

---

## 7. Friend System

### Purpose
Cultivate genuine spiritual community by connecting believers in mutual, consent-based relationships — enabling friends to see each other's prayer lives and pray together.

### User Stories
- As a user, I can send a friend request to anyone whose profile I can view.
- As a user, I can accept, decline, or ignore incoming friend requests.
- As a user, I can see a list of my friends and browse their recent prayer activity.
- As a user, I can remove a friend or block a user at any time.
- As a user, I can see mutual friends when viewing someone's profile.
- As a user, I receive a notification when someone accepts my friend request.

### Features
- Bidirectional friend requests (both parties must accept)
- Friend request with optional note (140 chars)
- Friend list with search
- Suggested friends: mutual friends, same denomination, same location, same prayer focus areas
- Mutual friends count shown on profiles
- Activity feed filtered to friends only
- Unfriend action (silent — no notification to removed party)
- Block: prevents all interaction, hides profiles from each other
- Friend count shown on profile (capped display at 500+)

### Database Requirements

```sql
-- friendships
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
requester_id    UUID REFERENCES users(id) ON DELETE CASCADE
addressee_id    UUID REFERENCES users(id) ON DELETE CASCADE
status          TEXT DEFAULT 'pending'  -- pending | accepted | declined
note            TEXT
created_at      TIMESTAMPTZ DEFAULT now()
updated_at      TIMESTAMPTZ DEFAULT now()
UNIQUE(requester_id, addressee_id)

-- blocks
blocker_id      UUID REFERENCES users(id) ON DELETE CASCADE
blocked_id      UUID REFERENCES users(id) ON DELETE CASCADE
created_at      TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (blocker_id, blocked_id)
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/friends/request/:userId` | Bearer | Send friend request |
| POST | `/friends/accept/:requestId` | Bearer | Accept request |
| POST | `/friends/decline/:requestId` | Bearer | Decline request |
| DELETE | `/friends/:userId` | Bearer | Unfriend |
| GET | `/friends` | Bearer | My friends list |
| GET | `/friends/requests` | Bearer | Pending requests |
| GET | `/friends/suggestions` | Bearer | Suggested friends |
| GET | `/users/:id/mutual-friends` | Bearer | Mutual friends |

---

## 8. Messaging

### Purpose
Private, Spirit-led communication between believers — enabling prayer partnerships, discipleship conversations, and community coordination without leaving the platform.

### User Stories
- As a user, I can send a direct message to any friend.
- As a user, I can create group chats with up to 50 participants for prayer circles.
- As a user, I can send text, images, prayer request links, and Bible verse snippets.
- As a user, I can see read receipts and typing indicators.
- As a user, I can delete my messages or leave a group conversation.
- As a user, I can mute or archive conversations.
- As a user, I cannot receive messages from users who are not my friends (DM request flow for non-friends).

### Features
- Direct messages (friends only; non-friends send a message request)
- Group chats (max 50 members, admin-managed)
- Message types: text, image, prayer_request card, bible_verse card, voice note (30s max)
- Real-time delivery via WebSocket
- Read receipts (single tick sent, double tick read)
- Typing indicators
- Message reactions (emoji, max 6 distinct per message)
- Message reply (thread reference)
- Edit message (up to 5 minutes after send)
- Delete for me / delete for everyone
- Mute conversation (1h, 8h, always)
- Archive conversation
- Message request inbox for non-friend DMs
- End-to-end encryption for DMs (Signal Protocol)
- Media stored in object storage, purged after 90 days if conversation inactive

### Database Requirements

```sql
-- conversations
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
type            TEXT NOT NULL     -- direct | group
name            TEXT              -- group name
avatar_url      TEXT
created_by      UUID REFERENCES users(id)
created_at      TIMESTAMPTZ DEFAULT now()

-- conversation_members
conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE
user_id         UUID REFERENCES users(id) ON DELETE CASCADE
role            TEXT DEFAULT 'member'  -- member | admin
last_read_at    TIMESTAMPTZ
is_muted        BOOLEAN DEFAULT false
mute_until      TIMESTAMPTZ
is_archived     BOOLEAN DEFAULT false
joined_at       TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (conversation_id, user_id)

-- messages
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE
sender_id       UUID REFERENCES users(id) ON DELETE SET NULL
type            TEXT DEFAULT 'text'  -- text | image | prayer_card | verse_card | voice
body            TEXT
metadata        JSONB     -- { url, request_id, verse_ref, duration }
reply_to_id     UUID REFERENCES messages(id)
is_edited       BOOLEAN DEFAULT false
edited_at       TIMESTAMPTZ
deleted_at      TIMESTAMPTZ
created_at      TIMESTAMPTZ DEFAULT now()

-- message_reactions
message_id  UUID REFERENCES messages(id) ON DELETE CASCADE
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
emoji       TEXT NOT NULL
created_at  TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (message_id, user_id, emoji)
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/conversations` | Bearer | List conversations |
| POST | `/conversations` | Bearer | Create DM or group |
| GET | `/conversations/:id/messages` | Bearer | Paginated messages |
| POST | `/conversations/:id/messages` | Bearer | Send message |
| PATCH | `/messages/:id` | Bearer | Edit message |
| DELETE | `/messages/:id` | Bearer | Delete message |
| POST | `/messages/:id/reactions` | Bearer | Add reaction |
| DELETE | `/messages/:id/reactions/:emoji` | Bearer | Remove reaction |
| POST | `/conversations/:id/mute` | Bearer | Mute conversation |
| PATCH | `/conversations/:id/archive` | Bearer | Archive |
| GET | `/conversations/requests` | Bearer | Message requests |
| POST | `/conversations/requests/:id/accept` | Bearer | Accept message request |
| WS | `/ws/messages` | Bearer | Real-time message stream |

---

## 9. Notifications

### Purpose
Timely, contextually relevant alerts that deepen engagement without creating noise — ensuring users never miss prayers answered, friend activity, or community moments they care about.

### User Stories
- As a user, I receive a push notification when someone prays for my request.
- As a user, I receive a notification when I get a new message or friend request.
- As a user, I can configure exactly which events trigger push, email, or in-app notifications.
- As a user, I can view all my notifications in a feed and mark them as read.
- As a user, I can mute all notifications on a schedule (e.g., midnight–6am).
- As a user, I never receive more than one notification email per hour (digest grouping).

### Features
- In-app notification centre (bell icon, unread badge count)
- Push notifications (FCM for Android, APNs for iOS, Web Push for PWA)
- Email notifications with digest batching (immediate / hourly / daily)
- Notification categories and per-category user controls:
  - Prayer activity (someone prayed for you, prayer answered)
  - Social (friend request, friend accepted, someone commented)
  - Messages (new DM, group message)
  - Groups (new post, upcoming event)
  - Platform (announcements, subscription expiry)
- Do Not Disturb schedule
- Notification feed with read/unread state
- Bulk mark-all-read action
- Deep-link routing: each notification links directly to the relevant content

### Database Requirements

```sql
-- notifications
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id         UUID REFERENCES users(id) ON DELETE CASCADE
type            TEXT NOT NULL    -- prayer_prayed | friend_request | message | etc.
title           TEXT NOT NULL
body            TEXT
image_url       TEXT
deep_link       TEXT
metadata        JSONB
is_read         BOOLEAN DEFAULT false
read_at         TIMESTAMPTZ
created_at      TIMESTAMPTZ DEFAULT now()

-- notification_preferences
user_id             UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE
prayer_push         BOOLEAN DEFAULT true
prayer_email        BOOLEAN DEFAULT true
social_push         BOOLEAN DEFAULT true
social_email        BOOLEAN DEFAULT false
messages_push       BOOLEAN DEFAULT true
messages_email      BOOLEAN DEFAULT false
groups_push         BOOLEAN DEFAULT true
groups_email        BOOLEAN DEFAULT true
platform_push       BOOLEAN DEFAULT true
platform_email      BOOLEAN DEFAULT true
dnd_enabled         BOOLEAN DEFAULT false
dnd_start_time      TIME DEFAULT '22:00'
dnd_end_time        TIME DEFAULT '07:00'
email_digest_freq   TEXT DEFAULT 'hourly'  -- immediate | hourly | daily

-- push_tokens
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
token       TEXT UNIQUE NOT NULL
platform    TEXT NOT NULL   -- ios | android | web
created_at  TIMESTAMPTZ DEFAULT now()
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/notifications` | Bearer | Paginated notification feed |
| PATCH | `/notifications/:id/read` | Bearer | Mark one as read |
| POST | `/notifications/read-all` | Bearer | Mark all as read |
| DELETE | `/notifications/:id` | Bearer | Delete notification |
| GET | `/notifications/preferences` | Bearer | Get preferences |
| PATCH | `/notifications/preferences` | Bearer | Update preferences |
| POST | `/notifications/push-token` | Bearer | Register device token |
| DELETE | `/notifications/push-token/:id` | Bearer | Deregister device |

---

## 10. Righteous Radio

### Purpose
A curated, Spirit-filled audio experience — worship music, prayer streams, sermons, and prophetic content that creates ambient spiritual atmosphere within the platform.

### User Stories
- As a user, I can open a radio player and choose from themed stations (worship, prayer, sermons, soaking music).
- As a user, the radio continues playing in the background while I browse the platform.
- As a user, I can see the currently playing track with artist info and artwork.
- As a user, I can like a track, skip to the next, and adjust volume.
- As a user (Kingdom Partner), I can access exclusive stations with premium content.
- As a user, I can see a schedule of live worship and prayer streams.

### Features
- Persistent audio player (bottom bar, does not interrupt on navigation)
- Stations: Worship, Soaking / Instrumental, Prayer, Sermons, Gospel, Prophetic
- Premium stations for Kingdom Partners
- Now Playing: track title, artist, album art, progress bar
- Controls: play/pause, skip, volume, like/unlike
- Track info card with artist bio and related content
- Live stream indicator and schedule
- Playback history (last 50 tracks)
- Offline listening: Kingdom Partners can cache up to 20 tracks
- Background audio (Service Worker / media session API)
- Mini-player that expands to full-screen player

### Database Requirements

```sql
-- radio_stations
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
name            TEXT NOT NULL
slug            TEXT UNIQUE NOT NULL
description     TEXT
artwork_url     TEXT
stream_url      TEXT NOT NULL
is_live         BOOLEAN DEFAULT false
is_premium      BOOLEAN DEFAULT false
sort_order      INT DEFAULT 0
is_active       BOOLEAN DEFAULT true

-- radio_tracks
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
station_id      UUID REFERENCES radio_stations(id) ON DELETE CASCADE
title           TEXT NOT NULL
artist          TEXT NOT NULL
album           TEXT
artwork_url     TEXT
audio_url       TEXT NOT NULL
duration_sec    INT
is_premium      BOOLEAN DEFAULT false
played_at       TIMESTAMPTZ   -- for live/scheduled tracks
created_at      TIMESTAMPTZ DEFAULT now()

-- radio_likes
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
track_id    UUID REFERENCES radio_tracks(id) ON DELETE CASCADE
created_at  TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (user_id, track_id)

-- radio_play_history
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
track_id    UUID REFERENCES radio_tracks(id) ON DELETE CASCADE
played_at   TIMESTAMPTZ DEFAULT now()
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/radio/stations` | Optional | List all stations |
| GET | `/radio/stations/:id` | Optional | Station details |
| GET | `/radio/stations/:id/tracks` | Optional | Station tracklist |
| GET | `/radio/now-playing` | Optional | Currently playing per station |
| GET | `/radio/schedule` | None | Upcoming live streams |
| POST | `/radio/tracks/:id/like` | Bearer | Like track |
| DELETE | `/radio/tracks/:id/like` | Bearer | Unlike track |
| GET | `/radio/history` | Bearer | Playback history |

---

## 11. Kingdom Partner Subscription

### Purpose
A premium membership tier that deepens commitment to the mission, unlocks exclusive features, and funds the platform's global ministry operations.

### User Stories
- As a user, I can view what Kingdom Partner benefits I'd receive before subscribing.
- As a user, I can subscribe monthly or annually via credit card or Apple/Google Pay.
- As a user, I receive immediate access to premium features upon subscription confirmation.
- As a user, I can manage, pause, or cancel my subscription at any time.
- As a user, my subscription auto-renews and I receive a 7-day renewal reminder email.
- As an admin, I can grant complimentary Kingdom Partner status to ministry leaders.

### Features

**Tiers**

| Tier | Monthly | Annual | Highlights |
|---|---|---|---|
| Intercessor | $4.99 | $47.99 | Ad-free, premium radio stations, offline caching |
| Warrior | $9.99 | $95.99 | All Intercessor + exclusive study content, priority prayer matching |
| General | $24.99 | $239.99 | All Warrior + personal discipleship coach matching, early feature access |

- Stripe Billing integration (subscriptions, trials, proration)
- Apple In-App Purchase + Google Play Billing for mobile
- 7-day free trial for first-time subscribers
- Annual plan shows savings vs monthly
- Subscription management portal (via Stripe Customer Portal)
- Grace period: 3 days after failed payment before feature loss
- Complimentary access grants (admin-issued, with optional expiry)
- Subscription receipt emails
- Kingdom Partner badge on profile

### Database Requirements

```sql
-- subscription_plans
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
name            TEXT NOT NULL    -- intercessor | warrior | general
stripe_price_id TEXT UNIQUE
monthly_price   NUMERIC(8,2)
annual_price    NUMERIC(8,2)
features        JSONB    -- array of feature slugs
is_active       BOOLEAN DEFAULT true

-- subscriptions
id                  UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id             UUID REFERENCES users(id) ON DELETE CASCADE
plan_id             UUID REFERENCES subscription_plans(id)
stripe_sub_id       TEXT UNIQUE
stripe_customer_id  TEXT
status              TEXT NOT NULL   -- trialing | active | past_due | canceled | paused | complimentary
current_period_start TIMESTAMPTZ
current_period_end   TIMESTAMPTZ
cancel_at_period_end BOOLEAN DEFAULT false
trial_end           TIMESTAMPTZ
created_at          TIMESTAMPTZ DEFAULT now()
updated_at          TIMESTAMPTZ DEFAULT now()

-- subscription_events (immutable log)
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
subscription_id UUID REFERENCES subscriptions(id)
event_type      TEXT NOT NULL   -- created | renewed | upgraded | canceled | payment_failed | etc.
metadata        JSONB
created_at      TIMESTAMPTZ DEFAULT now()
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/subscription/plans` | None | List plans |
| GET | `/subscription/me` | Bearer | Current subscription |
| POST | `/subscription/checkout` | Bearer | Create Stripe checkout session |
| POST | `/subscription/portal` | Bearer | Stripe billing portal URL |
| POST | `/subscription/cancel` | Bearer | Cancel at period end |
| POST | `/subscription/resume` | Bearer | Resume pending cancellation |
| POST | `/webhooks/stripe` | Stripe sig | Handle Stripe events |
| POST | `/admin/subscription/grant` | Admin | Grant complimentary access |

---

## 12. Donations

### Purpose
Enable believers to financially partner with the 120 Army mission — supporting specific prayer initiatives, global outreach projects, and platform operations with transparency and accountability.

### User Stories
- As a user, I can make a one-time donation or set up a recurring giving commitment.
- As a user, I can choose which campaign or fund to direct my gift toward.
- As a user, I receive an immediate email receipt that is tax-receipt eligible.
- As a user, I can see the impact of donations (e.g., "147 people funded prayer coverage in Nigeria").
- As a user, I can view my full giving history.
- As an admin, I can create and manage donation campaigns with targets and progress bars.

### Features
- One-time and recurring donations (weekly / monthly / annually)
- Campaign-based giving (with goal amount, progress bar, end date)
- General fund giving
- Payment methods: card (Stripe), Apple Pay, Google Pay
- Anonymous giving option
- Automated PDF tax receipts (annual summary + per-transaction)
- Giving history dashboard
- Campaign updates and impact reports sent to donors
- Donor recognition wall (opt-in, shows first name + amount range)
- Admin campaign management: create, pause, close, view donor list

### Database Requirements

```sql
-- donation_campaigns
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
title           TEXT NOT NULL
description     TEXT
image_url       TEXT
goal_amount     NUMERIC(12,2)
raised_amount   NUMERIC(12,2) DEFAULT 0
currency        CHAR(3) DEFAULT 'USD'
status          TEXT DEFAULT 'active'  -- active | paused | completed | archived
starts_at       TIMESTAMPTZ
ends_at         TIMESTAMPTZ
created_by      UUID REFERENCES users(id)
created_at      TIMESTAMPTZ DEFAULT now()

-- donations
id                  UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id             UUID REFERENCES users(id) ON DELETE SET NULL
campaign_id         UUID REFERENCES donation_campaigns(id) ON DELETE SET NULL
stripe_payment_id   TEXT UNIQUE
amount              NUMERIC(12,2) NOT NULL
currency            CHAR(3) DEFAULT 'USD'
is_recurring        BOOLEAN DEFAULT false
recurrence          TEXT    -- weekly | monthly | annually
is_anonymous        BOOLEAN DEFAULT false
status              TEXT DEFAULT 'pending'  -- pending | succeeded | failed | refunded
receipt_url         TEXT
created_at          TIMESTAMPTZ DEFAULT now()
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/donations/campaigns` | None | List active campaigns |
| GET | `/donations/campaigns/:id` | None | Campaign details |
| POST | `/donations/checkout` | Bearer | Create donation session |
| GET | `/donations/history` | Bearer | User giving history |
| GET | `/donations/receipt/:id` | Bearer | Download receipt PDF |
| POST | `/webhooks/stripe-donations` | Stripe sig | Handle donation events |
| POST | `/admin/campaigns` | Admin | Create campaign |
| PATCH | `/admin/campaigns/:id` | Admin | Update campaign |

---

## 13. Christian Short Videos

### Purpose
Bite-sized (60–180 second) Spirit-filled video content — testimonies, prayer prompts, scripture meditations, and ministry clips — designed for daily spiritual nourishment and viral sharing.

### User Stories
- As a user, I can scroll through a vertical video feed similar to a spiritually curated experience.
- As a user, I can like, comment, share, and save videos.
- As a user, I can follow content creators and have their videos prioritised in my feed.
- As a creator (Kingdom Partner), I can upload videos, add captions, choose a category, and publish.
- As a user, I can report inappropriate videos for moderator review.
- As a user, I can view a creator's full video profile.

### Features
- Vertical autoplay feed (TikTok-style swipe)
- Feed algorithm: personalisation by category engagement, followed creators, trending
- Categories: Testimony, Prayer, Scripture, Worship, Discipleship, Prophetic, Ministry
- Video length: 15 seconds – 3 minutes
- Upload (Kingdom Partners + verified creators): MP4, MOV, max 500MB
- Auto-generated captions (speech-to-text)
- Thumbnail auto-extraction or custom upload
- Engagement: like, comment (threaded one level), share (in-app link), save to collection
- Creator follow system (separate from friend system)
- Creator profile: bio, video grid, follower count, total likes
- Video collections (user-curated saved video playlists)
- Duet / response video (reply with a video)
- Content moderation: auto-flag keywords, moderator review queue

### Database Requirements

```sql
-- videos
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
creator_id      UUID REFERENCES users(id) ON DELETE SET NULL
title           TEXT NOT NULL
description     TEXT
category        TEXT NOT NULL
video_url       TEXT NOT NULL
thumbnail_url   TEXT
caption_url     TEXT      -- VTT file
duration_sec    INT
view_count      INT DEFAULT 0
like_count      INT DEFAULT 0
comment_count   INT DEFAULT 0
share_count     INT DEFAULT 0
save_count      INT DEFAULT 0
status          TEXT DEFAULT 'processing'  -- processing | published | flagged | removed
published_at    TIMESTAMPTZ
created_at      TIMESTAMPTZ DEFAULT now()

-- video_likes
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
video_id    UUID REFERENCES videos(id) ON DELETE CASCADE
created_at  TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (user_id, video_id)

-- video_saves
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
video_id    UUID REFERENCES videos(id) ON DELETE CASCADE
collection_id UUID REFERENCES video_collections(id)
created_at  TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (user_id, video_id)

-- video_collections
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
name        TEXT NOT NULL
is_private  BOOLEAN DEFAULT false
created_at  TIMESTAMPTZ DEFAULT now()

-- creator_follows
follower_id UUID REFERENCES users(id) ON DELETE CASCADE
creator_id  UUID REFERENCES users(id) ON DELETE CASCADE
created_at  TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (follower_id, creator_id)
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/videos/feed` | Optional | Personalised feed |
| GET | `/videos/trending` | None | Trending videos |
| GET | `/videos/:id` | Optional | Single video |
| POST | `/videos` | Bearer (creator) | Upload video |
| DELETE | `/videos/:id` | Bearer (owner) | Delete video |
| POST | `/videos/:id/like` | Bearer | Like video |
| GET | `/videos/:id/comments` | Optional | Comments |
| POST | `/videos/:id/comments` | Bearer | Post comment |
| POST | `/videos/:id/report` | Bearer | Report video |
| GET | `/creators/:id` | Optional | Creator profile + videos |
| POST | `/creators/:id/follow` | Bearer | Follow creator |
| GET | `/collections` | Bearer | User's collections |
| POST | `/collections` | Bearer | Create collection |
| POST | `/videos/:id/save` | Bearer | Save to collection |

---

## 14. Events

### Purpose
Connect the 120 Army community through virtual and in-person gatherings — prayer summits, worship nights, Bible studies, and city-level meetups organised and promoted within the platform.

### User Stories
- As a user, I can browse upcoming events by date, location, and type.
- As a user, I can register for an event and add it to my calendar.
- As an event organiser (verified), I can create events with full details, images, and registration settings.
- As a user, I receive a reminder 24 hours and 1 hour before a registered event.
- As a user, I can see which friends are attending an event.
- As an admin, I can feature events on the homepage and send platform-wide announcements.

### Features
- Event types: Virtual (Zoom/livestream link), In-person, Hybrid
- Event categories: Prayer Summit, Worship Night, Bible Study, Discipleship, Conference, Outreach, Other
- Rich event detail page: banner image, description, schedule, speakers, location (map embed for in-person), registration capacity
- RSVP: Going / Interested / Can't Go
- Capacity limits and waitlist
- Calendar export: .ics / Google Calendar / Apple Calendar
- Friends attending widget
- Event reminders (push + email)
- Recurring events support
- Organiser dashboard: attendee list, export CSV, send message to all attendees
- Featured events (admin-curated)
- Events discoverable on the globe for in-person events

### Database Requirements

```sql
-- events
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
organiser_id    UUID REFERENCES users(id) ON DELETE SET NULL
title           TEXT NOT NULL
description     TEXT
category        TEXT NOT NULL
event_type      TEXT NOT NULL    -- virtual | in_person | hybrid
banner_url      TEXT
start_at        TIMESTAMPTZ NOT NULL
end_at          TIMESTAMPTZ NOT NULL
timezone        TEXT NOT NULL
location_name   TEXT
address         TEXT
lat             NUMERIC(9,6)
lng             NUMERIC(9,6)
virtual_url     TEXT
capacity        INT
waitlist_enabled BOOLEAN DEFAULT false
is_featured     BOOLEAN DEFAULT false
is_recurring    BOOLEAN DEFAULT false
recurrence_rule TEXT             -- iCal RRULE
status          TEXT DEFAULT 'published'  -- draft | published | canceled | completed
rsvp_count      INT DEFAULT 0
created_at      TIMESTAMPTZ DEFAULT now()

-- event_rsvps
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
event_id    UUID REFERENCES events(id) ON DELETE CASCADE
status      TEXT NOT NULL   -- going | interested | not_going
is_waitlist BOOLEAN DEFAULT false
created_at  TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (user_id, event_id)
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/events` | Optional | Browse events (filter/search) |
| GET | `/events/:id` | Optional | Event detail |
| POST | `/events` | Bearer (organiser) | Create event |
| PATCH | `/events/:id` | Bearer (organiser) | Update event |
| DELETE | `/events/:id` | Bearer (organiser) | Cancel event |
| POST | `/events/:id/rsvp` | Bearer | RSVP to event |
| GET | `/events/:id/attendees` | Bearer | Attendees list |
| GET | `/events/:id/calendar` | Bearer | Download .ics |
| GET | `/events/my` | Bearer | My registered events |
| GET | `/events/friends` | Bearer | Events friends are attending |

---

## 15. Bible Study Groups

### Purpose
Structured, leader-facilitated communities for ongoing Scripture engagement, corporate prayer, and discipleship — the primary vehicle for deep spiritual formation on the platform.

### User Stories
- As a user, I can browse and join public Bible study groups by topic, book of the Bible, or denomination.
- As a user, I can create a private group for my church small group or family.
- As a group leader, I can post study sessions with content, discussion questions, and prayer points.
- As a group member, I can post comments, share insights, and submit prayer requests visible only to the group.
- As a group leader, I can manage members (approve, remove, promote to co-leader).
- As a user, I can leave a group at any time.

### Features
- Group types: Public (open join), Private (invite/request to join), Secret (invite only, not discoverable)
- Group profile: name, description, cover image, topic tags, denomination, meeting schedule
- Study sessions: structured posts with passage reference, notes, discussion prompts, prayer points
- Session completion tracking per member
- Group prayer wall (visibility scoped to group members)
- Group events (linked to Events module)
- Member roles: Leader, Co-leader, Member
- Invitation links (shareable, optionally expiring)
- Reading plans: multi-week scripture reading schedules with daily check-ins
- Group announcements (pinned posts)
- Member count, session count shown on group card

### Database Requirements

```sql
-- study_groups
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
name            TEXT NOT NULL
description     TEXT
cover_url       TEXT
type            TEXT DEFAULT 'public'  -- public | private | secret
denomination    TEXT
topic_tags      TEXT[]
meeting_schedule TEXT
member_count    INT DEFAULT 0
created_by      UUID REFERENCES users(id)
created_at      TIMESTAMPTZ DEFAULT now()

-- group_members
group_id    UUID REFERENCES study_groups(id) ON DELETE CASCADE
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
role        TEXT DEFAULT 'member'   -- leader | co_leader | member
status      TEXT DEFAULT 'active'   -- pending | active | removed
joined_at   TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (group_id, user_id)

-- study_sessions
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
group_id        UUID REFERENCES study_groups(id) ON DELETE CASCADE
author_id       UUID REFERENCES users(id) ON DELETE SET NULL
title           TEXT NOT NULL
passage_ref     TEXT
content         TEXT NOT NULL
discussion_qs   TEXT[]
prayer_points   TEXT[]
is_pinned       BOOLEAN DEFAULT false
published_at    TIMESTAMPTZ DEFAULT now()

-- reading_plans
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
group_id        UUID REFERENCES study_groups(id) ON DELETE CASCADE
title           TEXT NOT NULL
duration_days   INT NOT NULL
days            JSONB NOT NULL   -- [{ day: 1, passage: 'John 1:1-18', notes: '' }]
created_at      TIMESTAMPTZ DEFAULT now()

-- reading_plan_progress
plan_id     UUID REFERENCES reading_plans(id) ON DELETE CASCADE
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
day         INT NOT NULL
completed   BOOLEAN DEFAULT false
completed_at TIMESTAMPTZ
PRIMARY KEY (plan_id, user_id, day)
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/groups` | Optional | Browse groups |
| POST | `/groups` | Bearer | Create group |
| GET | `/groups/:id` | Optional | Group detail |
| PATCH | `/groups/:id` | Bearer (leader) | Update group |
| POST | `/groups/:id/join` | Bearer | Join / request to join |
| POST | `/groups/:id/leave` | Bearer | Leave group |
| GET | `/groups/:id/members` | Bearer (member) | Members list |
| PATCH | `/groups/:id/members/:uid` | Bearer (leader) | Update member role |
| DELETE | `/groups/:id/members/:uid` | Bearer (leader) | Remove member |
| GET | `/groups/:id/sessions` | Bearer (member) | Study sessions |
| POST | `/groups/:id/sessions` | Bearer (leader) | Create session |
| GET | `/groups/my` | Bearer | My groups |
| POST | `/groups/:id/invite` | Bearer (leader) | Generate invite link |

---

## 16. Testimonies

### Purpose
A dedicated space for believers to share answered prayers and miraculous stories — building corporate faith, encouraging intercessors whose prayers were answered, and giving glory to God.

### User Stories
- As a user, I can share a testimony linked to a prayer request that was answered.
- As a user, I can submit a standalone testimony (not linked to a request).
- As a user, I can browse testimonies by category and give a "Glory to God" acknowledgement.
- As a user, I can comment on a testimony to express encouragement.
- As a user whose prayer was prayed for by others, I can see who prayed and notify them when it's answered.
- As an admin, I can feature powerful testimonies on the homepage.

### Features
- Testimony types: Healing, Salvation, Provision, Restoration, Protection, Breakthrough, Guidance, Other
- Link to original prayer request (optional)
- Auto-notify all intercessors who logged a prayer on the original request when testimony is posted
- Rich text body (2,000 chars)
- Optional image/media attachment
- "Glory to God" counter (equivalent of a like)
- Comments (280 chars)
- Share to external social platforms
- Featured testimonies (admin curated) — displayed on homepage and globe
- Testimony feed: All / Following / Category filter
- Testimony count shown on user profile

### Database Requirements

```sql
-- testimonies
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id         UUID REFERENCES users(id) ON DELETE SET NULL
request_id      UUID REFERENCES prayer_requests(id) ON DELETE SET NULL
title           TEXT NOT NULL
body            TEXT NOT NULL
category        TEXT NOT NULL
media_url       TEXT
glory_count     INT DEFAULT 0
comment_count   INT DEFAULT 0
is_featured     BOOLEAN DEFAULT false
is_anonymous    BOOLEAN DEFAULT false
created_at      TIMESTAMPTZ DEFAULT now()

-- testimony_glories
user_id         UUID REFERENCES users(id) ON DELETE CASCADE
testimony_id    UUID REFERENCES testimonies(id) ON DELETE CASCADE
created_at      TIMESTAMPTZ DEFAULT now()
PRIMARY KEY (user_id, testimony_id)

-- testimony_comments
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
testimony_id UUID REFERENCES testimonies(id) ON DELETE CASCADE
user_id     UUID REFERENCES users(id) ON DELETE SET NULL
body        TEXT NOT NULL
created_at  TIMESTAMPTZ DEFAULT now()
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/testimonies` | Optional | Browse feed |
| GET | `/testimonies/featured` | None | Featured testimonies |
| POST | `/testimonies` | Bearer | Submit testimony |
| GET | `/testimonies/:id` | Optional | Single testimony |
| DELETE | `/testimonies/:id` | Bearer (owner) | Delete |
| POST | `/testimonies/:id/glory` | Bearer | Give glory |
| GET | `/testimonies/:id/comments` | Optional | Comments |
| POST | `/testimonies/:id/comments` | Bearer | Post comment |
| GET | `/users/:id/testimonies` | Optional | User's testimonies |

---

## 17. Admin Dashboard

### Purpose
Centralised control plane for platform administrators — content moderation, user management, analytics, feature flags, and community health oversight.

### User Stories
- As an admin, I can view a real-time dashboard of key platform metrics.
- As a moderator, I can review flagged content (prayers, videos, comments) and take action.
- As an admin, I can manage users: view profiles, suspend, ban, or grant roles.
- As an admin, I can create and schedule platform-wide announcements.
- As an admin, I can manage donation campaigns, subscription plans, and radio stations.
- As a super admin, I can manage admin and moderator accounts.

### Features

**Overview Dashboard**
- Real-time: active users, prayers in last 24h, new signups, active streams
- Charts: DAU/MAU trend, prayer volume, new users, retention, revenue

**User Management**
- Search by name, email, role, status
- View full profile, activity log, subscription status
- Actions: suspend (with duration + reason), permanent ban, role change, password reset, grant Kingdom Partner

**Content Moderation**
- Queue: flagged prayers, videos, comments, testimonies
- View report details, take action: approve / remove / warn user / ban user
- Bulk actions
- Moderation audit log

**Announcements**
- Create platform-wide notification / banner with scheduling
- Target by: all users, Kingdom Partners, specific countries

**Feature Flags**
- Toggle features on/off globally or by user segment

**Content Management**
- Radio stations CRUD
- Donation campaigns CRUD
- Featured content curation (testimonies, events)

**Roles**: `moderator` (content queue only), `admin` (all except super-admin actions), `super_admin` (full access + admin management)

### Database Requirements

```sql
-- admin_audit_log (immutable)
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
admin_id        UUID REFERENCES users(id)
action          TEXT NOT NULL    -- user_suspended | content_removed | role_changed | etc.
target_type     TEXT             -- user | prayer | video | etc.
target_id       UUID
reason          TEXT
metadata        JSONB
created_at      TIMESTAMPTZ DEFAULT now()

-- announcements
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
title           TEXT NOT NULL
body            TEXT NOT NULL
type            TEXT DEFAULT 'info'   -- info | warning | celebration
target_segment  TEXT DEFAULT 'all'    -- all | partners | country:<code>
deep_link       TEXT
scheduled_at    TIMESTAMPTZ
sent_at         TIMESTAMPTZ
created_by      UUID REFERENCES users(id)

-- feature_flags
slug        TEXT PRIMARY KEY
enabled     BOOLEAN DEFAULT false
rollout_pct INT DEFAULT 0       -- 0-100, percentage of users
metadata    JSONB
updated_at  TIMESTAMPTZ DEFAULT now()
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/admin/overview` | Admin | Dashboard summary stats |
| GET | `/admin/users` | Admin | Paginated user list |
| GET | `/admin/users/:id` | Admin | User detail + history |
| POST | `/admin/users/:id/suspend` | Admin | Suspend user |
| POST | `/admin/users/:id/ban` | Admin | Permanently ban |
| PATCH | `/admin/users/:id/role` | Super Admin | Change role |
| GET | `/admin/moderation/queue` | Moderator | Flagged content |
| POST | `/admin/moderation/:id/action` | Moderator | Take moderation action |
| GET | `/admin/audit-log` | Admin | Audit log |
| POST | `/admin/announcements` | Admin | Create announcement |
| GET | `/admin/flags` | Admin | Feature flags |
| PATCH | `/admin/flags/:slug` | Admin | Toggle flag |

---

## 18. Analytics

### Purpose
Data-driven insight into community health, spiritual engagement, and platform growth — enabling informed decisions about features, content, and global outreach strategy.

### User Stories
- As an admin, I can view growth metrics: new users, retention, churn over time.
- As an admin, I can see engagement metrics: prayers posted, prayers prayed, videos watched, sessions attended.
- As an admin, I can see a geographic breakdown of user activity and prayer focus areas.
- As an admin, I can view revenue metrics: MRR, ARR, subscriber count by tier, donation totals.
- As a content creator, I can see analytics on my own videos (views, watch time, likes, followers gained).
- As a group leader, I can see group engagement: session completion rates, member activity.

### Features

**Platform Analytics (admin)**
- Acquisition: signups by day/channel (organic, social, referral)
- Activation: % of signups who post first prayer, join first group, log first prayer
- Retention: Day 1/7/30 retention cohorts
- Engagement: DAU, MAU, prayer events/user, session length
- Content: top prayers, top videos, top testimonies, top groups
- Geography: heatmap of user density and prayer activity by country
- Revenue: MRR, ARR, subscriber count by plan, churn rate, LTV, donation totals
- Real-time: concurrent users, active prayers, active streams

**Creator Analytics**
- Per-video: views, watch time, completion rate, likes, comments, shares, saves
- Channel: follower growth, total views over time, top videos

**Group Analytics (leaders)**
- Member growth, active member %, session completion rate, prayer request volume

**Technical Implementation**
- Event tracking: client emits structured events to `/analytics/track`
- Events stored in time-series table; aggregated into summary tables via background jobs
- Dashboards built on aggregated tables for performance
- Raw events retained for 12 months; aggregates kept indefinitely

### Database Requirements

```sql
-- analytics_events (time-series, partitioned by month)
id          UUID DEFAULT gen_random_uuid()
user_id     UUID    -- nullable for anonymous events
session_id  TEXT
event_name  TEXT NOT NULL    -- prayer_posted | video_viewed | group_joined | etc.
properties  JSONB
platform    TEXT             -- web | ios | android
country     CHAR(2)
created_at  TIMESTAMPTZ DEFAULT now()

-- analytics_daily_summary (aggregated by background job)
date            DATE NOT NULL
metric          TEXT NOT NULL    -- dau | new_users | prayers_posted | etc.
value           NUMERIC NOT NULL
dimension_key   TEXT             -- optional grouping (country, plan_type, etc.)
dimension_value TEXT
PRIMARY KEY (date, metric, COALESCE(dimension_key,''), COALESCE(dimension_value,''))

-- revenue_snapshots (daily)
date            DATE PRIMARY KEY
mrr             NUMERIC(12,2)
arr             NUMERIC(12,2)
total_subscribers INT
subscribers_by_plan JSONB
total_donations NUMERIC(12,2)
new_subscribers INT
churned_subscribers INT
```

### API Requirements

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/analytics/track` | Optional | Ingest client event |
| GET | `/analytics/overview` | Admin | Platform summary |
| GET | `/analytics/growth` | Admin | User growth data |
| GET | `/analytics/engagement` | Admin | Engagement metrics |
| GET | `/analytics/geography` | Admin | Geographic breakdown |
| GET | `/analytics/revenue` | Admin | Revenue metrics |
| GET | `/analytics/content` | Admin | Top content rankings |
| GET | `/analytics/realtime` | Admin | Live platform stats |
| GET | `/analytics/creator` | Bearer (creator) | Creator's own stats |
| GET | `/analytics/creator/videos/:id` | Bearer (creator) | Per-video analytics |
| GET | `/analytics/groups/:id` | Bearer (leader) | Group engagement |

---

## Cross-Cutting Requirements

### Performance
- API p95 response time < 200ms for read endpoints, < 500ms for writes
- Feed pagination uses cursor-based pagination (no offset)
- Database queries must have index coverage — no sequential scans on hot tables
- Media assets served via CDN (CloudFront or similar)
- WebSocket connections auto-reconnect with exponential backoff

### Security
- All endpoints require HTTPS
- CSRF protection on all state-changing requests
- Rate limiting on all public endpoints
- PII fields (email, location) encrypted at rest
- GDPR: data export and deletion available to users within 30 days of request

### Internationalisation
- All user-facing strings externalised for i18n from day one
- RTL layout support (Arabic, Hebrew)
- Language stored on user profile; API returns language-aware content where applicable

### Accessibility
- WCAG 2.1 AA compliance
- All interactive elements keyboard navigable
- Screen reader labels on icon-only buttons
