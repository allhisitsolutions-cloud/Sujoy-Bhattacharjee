# COMPONENT_LIBRARY.md

**120 Army — Reusable Component Specifications**

This document is the authoritative reference for every shared UI component. Each component ships with its TypeScript interface, all visual states, variants, usage examples, and accessibility requirements. Build every component to this spec — no divergence.

All components are `"use client"` unless noted. All use Framer Motion for interaction animation. All are dark-mode-first and accept a `className` prop for layout overrides.

---

## Table of Contents

1. [PrayerCard](#1-prayercard)
2. [PrayerRequestCard](#2-prayerrequestcard)
3. [UserProfileCard](#3-userprofilecard)
4. [KingdomPartnerBadge](#4-kingdompartnerbadge)
5. [DonationCard](#5-donationcard)
6. [MissionCard](#6-missioncard)
7. [RadioPlayer](#7-radioplayer)
8. [VideoCard](#8-videocard)
9. [GlobalPrayerGlobe](#9-globalprayerglobe)
10. [NotificationCard](#10-notificationcard)
11. [MessageBubble](#11-messagebubble)
12. [TestimonyCard](#12-testimonycard)

---

## 1. PrayerCard

### Purpose
The atomic display unit for a single prayer event in the Prayer Wall feed. Represents one moment of intercession — who prayed, for what, and when. Designed to feel like a living, breathing record of faith rather than a list item.

### TypeScript Interface

```typescript
interface PrayerCardProps {
  // Data
  id: string;
  requestTitle: string;
  requestBody?: string;
  prayedBy: {
    id: string;
    displayName: string;
    avatarUrl?: string;
    isAnonymous?: boolean;
  };
  prayedAt: Date | string;
  category: PrayerCategory;
  urgency?: 'normal' | 'urgent' | 'critical';
  prayerCount: number;
  hasCurrentUserPrayed?: boolean;

  // Interaction handlers
  onPray?: (id: string) => void;
  onUnpray?: (id: string) => void;
  onViewRequest?: (id: string) => void;
  onShare?: (id: string) => void;

  // Display
  variant?: 'wall' | 'compact' | 'featured';
  isLoading?: boolean;
  className?: string;
}

type PrayerCategory =
  | 'health'    | 'family'      | 'finance'     | 'career'
  | 'relationships' | 'nations' | 'salvation'   | 'healing'
  | 'protection'    | 'guidance'| 'praise'      | 'other';
```

### States

| State | Behaviour |
|---|---|
| **Default** | Glass card, avatar + name, truncated request title, pray button, category badge |
| **Hovered** | Card lifts (`-translate-y-1`), border brightens to `white/14`, glow intensifies |
| **Prayed (active)** | Pray button fills gold, counter increments with spring animation, gold ring on button |
| **Loading** | Shimmer skeleton matching card dimensions — avatar circle, two text lines, button bar |
| **Anonymous** | Avatar replaced with generic prayer-hands SVG; name shows "Anonymous Intercessor" |
| **Urgent** | Amber border (`border-amber-500/30`), amber badge, subtle amber glow on card |
| **Critical** | Red border (`border-red-500/30`), red pulsing badge, red glow on card |

### Variants

**`wall`** (default) — Full card with body preview, engagement bar, and share action. Used in Prayer Wall feed.

**`compact`** — No body text, smaller avatar, inline pray button. Used in sidebars, notification previews, and prayer list.

**`featured`** — Gold glass treatment (`from-[#D4AF37]/12`), larger typography, prominent pray count. Used for pinned/featured requests.

### Visual Specification

```tsx
// wall variant — reference implementation
<motion.article
  whileHover={{ y: -4 }}
  whileTap={{ scale: 0.99 }}
  transition={{ duration: 0.2 }}
  className="relative overflow-hidden rounded-2xl p-6
             bg-gradient-to-br from-white/8 to-white/2
             backdrop-blur-xl border border-white/8
             shadow-[0_8px_32px_rgba(0,0,0,0.4)]"
  aria-label={`Prayer from ${prayedBy.displayName}: ${requestTitle}`}
>
  {/* Decorative inner glow */}
  <div className="absolute -top-10 -right-10 w-32 h-32
                  bg-[#D4AF37]/6 rounded-full blur-2xl pointer-events-none" />

  {/* Header row */}
  <div className="relative z-10 flex items-center gap-3 mb-4">
    <Avatar src={prayedBy.avatarUrl} size="md" isAnonymous={prayedBy.isAnonymous} />
    <div className="flex-1 min-w-0">
      <p className="text-sm font-semibold text-[#F8FAFC] truncate">{prayedBy.displayName}</p>
      <p className="text-xs text-[#94A3B8]">{timeAgo(prayedAt)}</p>
    </div>
    <CategoryBadge category={category} />
  </div>

  {/* Content */}
  <h3 className="relative z-10 text-[#F8FAFC] font-semibold leading-snug mb-2 line-clamp-2">
    {requestTitle}
  </h3>
  {requestBody && (
    <p className="relative z-10 text-sm text-[#94A3B8] line-clamp-3 leading-relaxed mb-4">
      {requestBody}
    </p>
  )}

  {/* Engagement bar */}
  <div className="relative z-10 flex items-center gap-4 pt-4 border-t border-white/6">
    <PrayButton id={id} count={prayerCount} active={hasCurrentUserPrayed} onPray={onPray} onUnpray={onUnpray} />
    <button onClick={() => onShare?.(id)} aria-label="Share prayer request"
            className="ml-auto p-2 rounded-lg text-[#94A3B8] hover:text-[#F8FAFC]
                       hover:bg-white/8 transition-colors duration-150">
      <ShareIcon className="w-4 h-4" />
    </button>
  </div>
</motion.article>
```

### Skeleton

```tsx
function PrayerCardSkeleton() {
  return (
    <div className="rounded-2xl p-6 bg-white/4 border border-white/6 animate-pulse space-y-4">
      <div className="flex items-center gap-3">
        <div className="w-10 h-10 rounded-full bg-white/10 shrink-0" />
        <div className="flex-1 space-y-2">
          <div className="h-3 bg-white/10 rounded-full w-1/3" />
          <div className="h-2 bg-white/6 rounded-full w-1/4" />
        </div>
        <div className="h-5 w-16 bg-white/8 rounded-full" />
      </div>
      <div className="space-y-2">
        <div className="h-4 bg-white/10 rounded-full" />
        <div className="h-4 bg-white/10 rounded-full w-4/5" />
      </div>
      <div className="h-px bg-white/6" />
      <div className="flex gap-3">
        <div className="h-8 w-24 bg-white/8 rounded-full" />
        <div className="h-8 w-8 bg-white/6 rounded-lg ml-auto" />
      </div>
    </div>
  );
}
```

### Accessibility

- `<article>` element with descriptive `aria-label` combining poster name and request title
- Pray button: `aria-label="Pray for this request"` / `aria-pressed={hasCurrentUserPrayed}`
- Anonymous cards: `aria-label` uses "Anonymous Intercessor" — never exposes hidden identity
- Category badge: `role="img"` with `aria-label` spelling out the category name
- Urgency indicator: paired with visible text — never communicated by colour alone
- Focusable with keyboard; `Enter` / `Space` on card navigates to full request
- All icon-only buttons have `aria-label`

---

## 2. PrayerRequestCard

### Purpose
A richer, more detailed card for displaying a user's own prayer requests — in their profile, the request detail page sidebar, and the tracking module. Shows status, expiry, and management actions.

### TypeScript Interface

```typescript
interface PrayerRequestCardProps {
  // Data
  id: string;
  title: string;
  body: string;
  category: PrayerCategory;
  urgency: 'normal' | 'urgent' | 'critical';
  status: 'active' | 'answered' | 'expired' | 'removed';
  isAnonymous: boolean;
  prayerCount: number;
  commentCount: number;
  tags?: string[];
  imageUrl?: string;
  createdAt: Date | string;
  expiresAt?: Date | string;
  answeredAt?: Date | string;

  // Ownership
  isOwner?: boolean;

  // Interaction handlers
  onEdit?: (id: string) => void;
  onDelete?: (id: string) => void;
  onMarkAnswered?: (id: string) => void;
  onRenew?: (id: string) => void;
  onPray?: (id: string) => void;
  onComment?: (id: string) => void;

  // Display
  variant?: 'full' | 'summary' | 'answered';
  isLoading?: boolean;
  className?: string;
}
```

### States

| State | Behaviour |
|---|---|
| **Active** | Standard glass card; expiry countdown if < 7 days remaining |
| **Answered** | Gold-tinted glass (`from-[#D4AF37]/10`); gold "Answered" badge top-right; answered date shown |
| **Expiring soon** | Amber tint; amber countdown badge ("Expires in 3 days"); "Renew" action visible |
| **Expired** | Muted opacity (`opacity-60`); greyed border; "Expired" badge; "Re-open" action |
| **Removed** | Not rendered — handled at list level |
| **Owner view** | Three-dot menu revealed on hover: Edit / Mark Answered / Delete |
| **Loading** | Shimmer skeleton with image placeholder, two text lines, tag row, stat bar |

### Variants

**`full`** — Complete card with image (if present), full body text (clamped to 4 lines), tags, stat bar, and owner action menu. Used on profile and feed.

**`summary`** — Title only, status badge, prayer count. Used in prayer list sidebar and search results.

**`answered`** — Full card with gold glass treatment, answered date, and link to associated testimony if one exists.

### Accessibility

- `<article>` with `aria-label` including title and current status
- Status badges use `role="status"` for dynamic updates
- Owner menu trigger: `aria-label="Request options"`, `aria-haspopup="true"`, `aria-expanded`
- Expiry countdown: `aria-live="polite"` for near-expiry updates
- "Mark as Answered" button: `aria-label="Mark this prayer as answered"` — confirm dialog before action
- Delete action: confirmation modal required; `aria-label="Delete prayer request"`
- Image: `alt` text auto-generated from title ("Prayer request image for: {title}")

---

## 3. UserProfileCard

### Purpose
A compact, scannable summary of a user's identity and spiritual profile — used in search results, friend suggestions, group member lists, prayer request author panels, and hovercards.

### TypeScript Interface

```typescript
interface UserProfileCardProps {
  // Data
  user: {
    id: string;
    displayName: string;
    avatarUrl?: string;
    bio?: string;
    denomination?: string;
    countryCode?: string;
    city?: string;
    spiritualGifts?: string[];
    prayerFocusAreas?: string[];
    prayerStreak?: number;
    totalPrayersPrayed?: number;
    isKingdomPartner?: boolean;
    isOnline?: boolean;
  };

  // Relationship context
  friendStatus?: 'none' | 'pending_sent' | 'pending_received' | 'friends';
  mutualFriendCount?: number;

  // Interaction handlers
  onViewProfile?: (id: string) => void;
  onSendFriendRequest?: (id: string) => void;
  onAcceptFriendRequest?: (id: string) => void;
  onMessage?: (id: string) => void;
  onPrayTogether?: (id: string) => void;

  // Display
  variant?: 'card' | 'row' | 'hovercard' | 'mini';
  showActions?: boolean;
  isLoading?: boolean;
  className?: string;
}
```

### States

| State | Behaviour |
|---|---|
| **Default** | Avatar, name, denomination, location, gifts preview, friend/message actions |
| **Online** | Green pulse dot overlaid on avatar bottom-right |
| **Kingdom Partner** | `KingdomPartnerBadge` inline with name; subtle gold ring on avatar |
| **Friend request sent** | Button changes to "Pending" (muted, non-interactive) |
| **Friend request received** | Accept / Decline button pair instead of add-friend |
| **Already friends** | "Message" and "Pray Together" actions replace friend request |
| **Loading** | Circular avatar skeleton, two text line skeletons, button bar skeleton |

### Variants

**`card`** — Full card with bio excerpt, gifts, location, stats, and action buttons. Used in search results and friend suggestions.

**`row`** — Horizontal compact: avatar + name + denomination + single primary action. Used in group member lists and leaderboards.

**`hovercard`** — Floating popover (Radix UI Popover base) triggered on name hover in feed/comments. Shows avatar, name, online status, prayer streak, mutual friends, quick actions. Max width `320px`.

**`mini`** — Avatar + name only, no actions. Used in "prayed by" stacks and mention chips.

### Accessibility

- Interactive card: `role="article"`, clickable to full profile
- Online indicator: `aria-label="Online"` on the dot; screen readers don't see it by colour alone
- Kingdom Partner indicator: text equivalent "Kingdom Partner" visually hidden alongside badge
- Hovercard: triggered by both hover and focus; `role="tooltip"` with `id` linked to triggering element via `aria-describedby`; dismissible with `Escape`
- Action buttons: `aria-label="Send friend request to {displayName}"`, `aria-label="Message {displayName}"`
- Mutual friends: `aria-label="{n} mutual friends"` on the count

---

## 4. KingdomPartnerBadge

### Purpose
A prestigious, attention-worthy indicator of paid Kingdom Partner membership status. Appears inline with usernames, on profile headers, and on content created by members. Must feel earned and special — not like a generic checkmark.

### TypeScript Interface

```typescript
interface KingdomPartnerBadgeProps {
  // Data
  tier: 'intercessor' | 'warrior' | 'general';
  showLabel?: boolean;
  showTooltip?: boolean;

  // Display
  size?: 'xs' | 'sm' | 'md' | 'lg';
  variant?: 'inline' | 'pill' | 'card';
  animated?: boolean;
  className?: string;
}
```

### States

| State | Behaviour |
|---|---|
| **Default** | Gold crown/shield icon with tier-appropriate colour |
| **Animated** | Subtle gold shimmer sweep (`background-position` animation) on the icon |
| **Tooltip visible** | Radix Tooltip showing tier name, membership benefits summary, "Upgrade" CTA for non-members |
| **Hover** | Icon brightens; tooltip appears after 300ms delay |

### Variants

**`inline`** — Icon only (`xs` or `sm`). Placed directly after username in feeds, comments, message headers. No background.

**`pill`** — Icon + label ("Kingdom Partner · Warrior") in a gold-bordered pill. Used on profile headers and cards.

**`card`** — Full benefit summary card. Used in settings, subscription upsell, and admin user detail views.

### Tier Styling

| Tier | Icon | Colour | Label |
|---|---|---|---|
| `intercessor` | Shield | `#D4AF37` (gold) | Kingdom Partner |
| `warrior` | Sword + Shield | `#E8CC6A` (bright gold) | Kingdom Warrior |
| `general` | Crown | Gradient `#E8CC6A → #D4AF37` + shimmer | Kingdom General |

### Visual Specification

```tsx
// inline variant
<span
  className="inline-flex items-center gap-1"
  aria-label={`Kingdom Partner — ${tier} tier`}
>
  <motion.span
    animate={animated ? { opacity: [0.8, 1, 0.8] } : {}}
    transition={{ duration: 2.5, repeat: Infinity, ease: 'easeInOut' }}
    className="text-[#D4AF37]"
    aria-hidden="true"
  >
    <TierIcon tier={tier} className={sizeClasses[size]} />
  </motion.span>

  {showLabel && (
    <span className="text-xs font-semibold text-[#D4AF37] tracking-wide">
      {tierLabels[tier]}
    </span>
  )}
</span>
```

### Accessibility

- Always carries `aria-label` with tier name — never communicated by icon or colour alone
- Tooltip trigger: `aria-describedby` pointing to tooltip content `id`
- Non-member tooltip "Upgrade" CTA: full link text "Upgrade to Kingdom Partner"
- Badge never uses `role="img"` alone — pair with text alternative or `aria-label`
- `general` tier shimmer animation: honours `prefers-reduced-motion` (static gold if reduced)

---

## 5. DonationCard

### Purpose
Presents a donation campaign with emotional impact — progress toward goal, story, and a clear giving CTA. Must feel generous and trustworthy, not transactional.

### TypeScript Interface

```typescript
interface DonationCardProps {
  // Data
  campaign: {
    id: string;
    title: string;
    description: string;
    imageUrl?: string;
    goalAmount: number;
    raisedAmount: number;
    currency: string;
    donorCount: number;
    endsAt?: Date | string;
    status: 'active' | 'completed' | 'paused';
    organiserName?: string;
  };

  // Interaction handlers
  onDonate?: (campaignId: string) => void;
  onViewDetails?: (campaignId: string) => void;

  // Display
  variant?: 'featured' | 'standard' | 'compact' | 'completed';
  showProgress?: boolean;
  isLoading?: boolean;
  className?: string;
}
```

### States

| State | Behaviour |
|---|---|
| **Active** | Progress bar animates in on mount (spring from 0 to current %); Donate CTA active |
| **Goal reached** | Progress bar fills gold and pulses; badge "Goal Reached 🙌"; Donate button changes to "Keep Giving" |
| **Paused** | Muted card, "Paused" badge, Donate button disabled with tooltip "Giving is paused" |
| **Completed / Archived** | Full progress, "Campaign Closed" overlay with thank-you copy and total raised |
| **Loading** | Image placeholder, title skeleton, progress bar skeleton, button skeleton |
| **Urgency (< 48h left)** | Amber countdown timer ("48 hrs left"), urgency note below progress |

### Variants

**`featured`** — Full-bleed campaign image (aspect 16:9), gold-glass overlay with title, large animated progress bar, donor count, prominent CTA. Used on homepage and campaign highlights.

**`standard`** — Thumbnail image, title, description excerpt (2 lines), compact progress bar, donor count, Donate button. Used in campaign browse grid.

**`compact`** — Title + inline micro progress bar + single-line donor count + Donate button. Used in sidebars and widget placements.

**`completed`** — `standard` layout with a celebratory gold overlay, final raised amount, and "Read Impact Report" CTA in place of donate.

### Progress Bar Specification

```tsx
// Animated progress bar — always animate from 0 on mount
function CampaignProgress({ raised, goal }: { raised: number; goal: number }) {
  const pct = Math.min((raised / goal) * 100, 100);

  return (
    <div>
      <div className="flex justify-between text-xs text-[#94A3B8] mb-2">
        <span>
          <strong className="text-[#F8FAFC]">${raised.toLocaleString()}</strong> raised
        </span>
        <span>of ${goal.toLocaleString()}</span>
      </div>

      <div className="h-2 rounded-full bg-white/8 overflow-hidden" role="progressbar"
           aria-valuenow={Math.round(pct)} aria-valuemin={0} aria-valuemax={100}
           aria-label={`${Math.round(pct)}% of goal raised`}>
        <motion.div
          className="h-full rounded-full bg-gradient-to-r from-[#D4AF37] to-[#E8CC6A]"
          initial={{ width: 0 }}
          animate={{ width: `${pct}%` }}
          transition={{ duration: 1.2, ease: [0.0, 0.0, 0.2, 1], delay: 0.3 }}
        />
      </div>

      <p className="text-xs text-[#94A3B8] mt-2">
        <strong className="text-[#F8FAFC]">{donorCount.toLocaleString()}</strong> donors
      </p>
    </div>
  );
}
```

### Accessibility

- `<article>` with `aria-label` of campaign title
- Progress bar: `role="progressbar"` with `aria-valuenow`, `aria-valuemin`, `aria-valuemax`, `aria-label`
- Countdown timer: `aria-live="polite"` updates; `aria-label="Time remaining: {n} hours"`
- Donate button: `aria-label="Donate to {campaign title}"`
- Completed overlay: not `aria-hidden` — screen readers should hear the completion message
- Currency amounts: `aria-label` uses full text ("$12,500 raised of $20,000 goal") not just numbers

---

## 6. MissionCard

### Purpose
Highlights a specific mission initiative, outreach effort, or Kingdom focus area — used on the homepage, mission hub, and partnership pages to communicate the global scope of 120 Army's ministry.

### TypeScript Interface

```typescript
interface MissionCardProps {
  // Data
  mission: {
    id: string;
    title: string;
    region: string;
    countryCode?: string;
    description: string;
    imageUrl?: string;
    iconType?: 'cross' | 'globe' | 'hands' | 'fire' | 'dove' | 'book';
    prayerCount?: number;
    partnerCount?: number;
    status?: 'active' | 'urgent' | 'completed';
    tags?: string[];
  };

  // Interaction handlers
  onPray?: (id: string) => void;
  onPartner?: (id: string) => void;
  onViewDetails?: (id: string) => void;

  // Display
  variant?: 'hero' | 'standard' | 'compact' | 'map-pin';
  isLoading?: boolean;
  className?: string;
}
```

### States

| State | Behaviour |
|---|---|
| **Default** | Glass card with region flag, title, description, prayer/partner stats |
| **Urgent** | Red pulsing border glow; "Urgent" badge; description leads with urgency note |
| **Active** | Green pulse on status indicator; "Praying Now" count if live |
| **Completed** | Gold shimmer; "Mission Completed" badge; testimony count |
| **Hovered** | Image (if present) scales to 105%; card lifts; "Pray Now" CTA appears |
| **Loading** | Image placeholder, title skeleton, two description lines, stat row |

### Variants

**`hero`** — Full-bleed image background with gradient overlay, large title, region label, description, and dual CTA (Pray + Partner). Used at the top of mission pages.

**`standard`** — Glass card with small region image/flag, title, excerpt, prayer count, partner count. Standard grid placement.

**`compact`** — Icon + title + region + one-line description + prayer button. Used in sidebars and "Missions of the Week" widget.

**`map-pin`** — Compact popover content displayed when a user clicks a country on the Global Prayer Globe. Shows mission title, prayer count, and Pray CTA.

### Accessibility

- `<article>` element
- Country flag emoji or image: `aria-label="Flag of {country name}"` — never bare emoji
- Icon type: `aria-hidden="true"` — paired with visible text label
- Status indicators: text label always present alongside colour/animation
- "Pray" CTA: `aria-label="Pray for {mission title} in {region}"`
- "Partner" CTA: `aria-label="Partner with the {mission title} mission"`

---

## 7. RadioPlayer

### Purpose
A persistent, floating audio player for Righteous Radio that survives navigation without interruption. The player exists in two states: a minimised bottom bar and an expanded full-player sheet.

### TypeScript Interface

```typescript
interface RadioPlayerProps {
  // Current track
  currentTrack?: {
    id: string;
    title: string;
    artist: string;
    artworkUrl?: string;
    durationSec?: number;
    isLive?: boolean;
    isPremium?: boolean;
  };

  // Station context
  station?: {
    id: string;
    name: string;
    isPremium: boolean;
  };

  // Playback state
  isPlaying: boolean;
  isBuffering?: boolean;
  progressSec?: number;
  volume: number;
  isMuted?: boolean;
  isLiked?: boolean;

  // Interaction handlers
  onPlayPause: () => void;
  onNext?: () => void;
  onPrev?: () => void;
  onSeek?: (sec: number) => void;
  onVolumeChange?: (vol: number) => void;
  onMute?: () => void;
  onLike?: (trackId: string) => void;
  onUnlike?: (trackId: string) => void;
  onExpand?: () => void;
  onCollapse?: () => void;
  onClose?: () => void;

  // Display
  variant?: 'mini' | 'full';
  isKingdomPartner?: boolean;
  className?: string;
}
```

### States

| State | Behaviour |
|---|---|
| **Playing** | Artwork rotates slowly (CSS `animate-spin` at 20s); waveform animation visible; pause icon |
| **Paused** | Artwork stops rotating; waveform freezes; play icon |
| **Buffering** | Artwork pulses (`animate-pulse`); spinner overlaid on play/pause button |
| **Live** | Progress bar replaced with live pulse indicator; "LIVE" badge in red; no seek |
| **Premium locked** | Premium station: blur overlay on artwork; "Kingdom Partner Only" overlay; upgrade CTA |
| **Liked** | Heart icon fills gold with spring pop animation |
| **Mini (collapsed)** | 72px tall bottom bar; artwork thumbnail, title/artist, play/pause only |
| **Full (expanded)** | Bottom sheet or centred modal; large artwork, full controls, progress bar, volume |

### Mini Player — Visual Specification

```tsx
// Fixed bottom bar — always present when a station is loaded
<motion.div
  initial={{ y: 80 }}
  animate={{ y: 0 }}
  exit={{ y: 80 }}
  className="fixed bottom-16 md:bottom-0 inset-x-0 z-40
             bg-[#0F172A]/95 backdrop-blur-2xl
             border-t border-white/8
             px-4 h-18 flex items-center gap-4"
  role="region"
  aria-label="Righteous Radio player"
>
  {/* Artwork */}
  <div className="w-12 h-12 rounded-xl overflow-hidden shrink-0 cursor-pointer"
       onClick={onExpand} aria-label="Expand player">
    <motion.img
      src={currentTrack?.artworkUrl}
      animate={{ rotate: isPlaying ? 360 : 0 }}
      transition={{ duration: 20, repeat: Infinity, ease: 'linear' }}
      className="w-full h-full object-cover"
      alt={`Artwork for ${currentTrack?.title}`}
    />
  </div>

  {/* Track info */}
  <div className="flex-1 min-w-0 cursor-pointer" onClick={onExpand}>
    <p className="text-sm font-semibold text-[#F8FAFC] truncate">{currentTrack?.title}</p>
    <p className="text-xs text-[#94A3B8] truncate">{currentTrack?.artist}</p>
  </div>

  {/* Controls */}
  <div className="flex items-center gap-3 shrink-0">
    <LikeButton isLiked={isLiked} onLike={onLike} onUnlike={onUnlike} />
    <PlayPauseButton isPlaying={isPlaying} isBuffering={isBuffering} onClick={onPlayPause} />
    <SkipButton direction="next" onClick={onNext} />
  </div>
</motion.div>
```

### Accessibility

- Entire player wrapped in `role="region"` with `aria-label="Righteous Radio player"`
- Play/Pause: `aria-label="Play"` / `aria-label="Pause"`, `aria-pressed` for toggle state
- Skip buttons: `aria-label="Next track"` / `aria-label="Previous track"`
- Progress bar: `role="slider"`, `aria-valuemin={0}`, `aria-valuemax={durationSec}`, `aria-valuenow={progressSec}`, `aria-label="Seek"`
- Volume: `role="slider"`, `aria-label="Volume"`, `aria-valuenow={volume * 100}`
- Mute: `aria-label="Mute"` / `aria-label="Unmute"`, `aria-pressed`
- Like: `aria-label="Like track"` / `aria-label="Unlike track"`, `aria-pressed`
- Live badge: `aria-label="Live broadcast"` — colour not sole indicator
- Premium lock: locked state communicated via `aria-disabled="true"` and visible text
- Keyboard: full keyboard navigation; Space = play/pause when player region is focused

---

## 8. VideoCard

### Purpose
Display unit for Christian short-form video content in the browse grid and feed. Optimised for thumb-stopping first impressions — thumbnail, creator, engagement stats, and play intent.

### TypeScript Interface

```typescript
interface VideoCardProps {
  // Data
  video: {
    id: string;
    title: string;
    thumbnailUrl: string;
    previewUrl?: string;     // short looping preview for hover autoplay
    durationSec: number;
    category: VideoCategory;
    creator: {
      id: string;
      displayName: string;
      avatarUrl?: string;
      isVerified?: boolean;
      isKingdomPartner?: boolean;
    };
    viewCount: number;
    likeCount: number;
    publishedAt: Date | string;
    isLiked?: boolean;
    isSaved?: boolean;
  };

  // Interaction handlers
  onPlay?: (id: string) => void;
  onLike?: (id: string) => void;
  onUnlike?: (id: string) => void;
  onSave?: (id: string) => void;
  onUnsave?: (id: string) => void;
  onShare?: (id: string) => void;
  onCreatorClick?: (creatorId: string) => void;

  // Display
  variant?: 'grid' | 'feed' | 'compact' | 'featured';
  autoPlayPreview?: boolean;
  isLoading?: boolean;
  className?: string;
}

type VideoCategory =
  | 'testimony' | 'prayer' | 'scripture' | 'worship'
  | 'discipleship' | 'prophetic' | 'ministry';
```

### States

| State | Behaviour |
|---|---|
| **Default** | Thumbnail with duration badge; creator row below; stats |
| **Hovered** | Preview video autoplays (muted, looped) after 400ms hover delay; overlay actions appear; card lifts |
| **Playing (feed)** | Full-screen vertical video player with overlay controls |
| **Liked** | Heart icon fills gold with spring pop; count increments |
| **Saved** | Bookmark fills; toast "Saved to collection" |
| **Loading** | Thumbnail area shimmer (16:9 ratio), creator row skeleton, stats skeleton |
| **Premium content** | Lock icon on thumbnail; "Kingdom Partners Only" label; click prompts upgrade modal |

### Variants

**`grid`** — Vertical card (9:16 thumbnail), creator avatar + name below, view count + duration overlay. Used in browse grid.

**`feed`** — Horizontal card: thumbnail left (16:9, fixed width), title + creator + stats right. Used in activity feeds and search results.

**`compact`** — Thumbnail + title + creator name only. Used in "More from this creator" rail.

**`featured`** — Full-width hero card with large 16:9 thumbnail, category badge, title, creator info, play button CTA, and engagement stats.

### Duration Badge

```tsx
<span className="absolute bottom-2 right-2 px-2 py-0.5
                 bg-black/70 backdrop-blur-sm rounded text-white text-xs font-mono"
      aria-label={`Duration: ${formatDuration(durationSec)}`}>
  {formatDuration(durationSec)}
</span>
```

### Accessibility

- `<article>` with `aria-label` including title and creator name
- Thumbnail: `<img alt="">` (empty — adjacent text describes it) OR `alt={title}` if no adjacent text
- Play button overlay: `aria-label="Play ${video.title}"`
- Duration badge: `aria-label` with full spoken duration ("Duration: 2 minutes 34 seconds")
- Hover preview: `autoPlay muted loop playsInline`; stopped on `mouseleave` and when `prefers-reduced-motion` is set
- Like/Save/Share: all have `aria-label` with action + title; `aria-pressed` on toggle states
- Creator link: `aria-label="View {displayName}'s profile"`
- View count: `aria-label="{n} views"` — not just the raw number

---

## 9. GlobalPrayerGlobe

### Purpose
A Three.js/Globe.gl powered 3D interactive globe visualising real-time prayer activity worldwide. The centrepiece visual of the platform — must be performant, accessible, and spiritually evocative.

### TypeScript Interface

```typescript
interface GlobalPrayerGlobeProps {
  // Data
  countryData?: CountryActivityData[];
  liveArcs?: PrayerArc[];

  // Config
  autoRotate?: boolean;
  autoRotateSpeed?: number;    // default: 0.5
  showArcs?: boolean;
  showHeatmap?: boolean;
  showNightTerminator?: boolean;
  highlightCountry?: string;   // ISO 3166-1 alpha-2

  // Interaction handlers
  onCountryClick?: (countryCode: string, data: CountryActivityData) => void;
  onCountryHover?: (countryCode: string | null) => void;
  onArcAnimationComplete?: (arc: PrayerArc) => void;

  // Display
  variant?: 'hero' | 'widget' | 'fullscreen';
  height?: number | string;
  className?: string;
}

interface CountryActivityData {
  countryCode: string;
  countryName: string;
  activeRequests: number;
  activeUsers: number;
  prayers24h: number;
  lat: number;
  lng: number;
}

interface PrayerArc {
  id: string;
  fromLat: number;
  fromLng: number;
  toLat: number;
  toLng: number;
  color?: string;
  altitude?: number;
}
```

### States

| State | Behaviour |
|---|---|
| **Loading** | Pulsing gold sphere placeholder with "Loading global prayer data…" label |
| **Idle (auto-rotating)** | Globe rotates slowly; arcs animate in; country markers pulse |
| **User interacting** | Auto-rotation pauses; cursor changes; country highlights on hover |
| **Country selected** | Globe animates to centre selected country; side panel slides in with country data |
| **High activity** | Hotter heat-map colours; arc density increases; counter animates |
| **Error** | Static flat world map fallback image with error message |
| **Mobile** | Touch-enabled rotation and pinch-to-zoom; reduced arc count (max 5 simultaneous) |

### Variants

**`hero`** — Full-width, `600px` tall, visible arc animations, country heat-map, auto-rotating, stats overlay. Used on landing page.

**`widget`** — `400px × 400px`, minimal UI, no sidebar panel, click triggers navigation to full globe page. Used on dashboard.

**`fullscreen`** — `100vh`, full controls, country drill-down sidebar, arc legend, live counter, filter toggles.

### Implementation Notes

```tsx
// Must be dynamically imported — no SSR
const GlobeComponent = dynamic(() => import('./GlobeComponent'), {
  ssr: false,
  loading: () => <GlobePlaceholder />,
});

// Arc colour mapping
const arcColors = {
  prayer_sent:   'rgba(212, 175, 55, 0.8)',   // gold
  prayer_prayed: 'rgba(59, 130, 246, 0.7)',    // blue
  answered:      'rgba(34, 197, 94, 0.8)',      // green
};

// Performance: limit simultaneous arcs
const MAX_ARCS_DESKTOP = 20;
const MAX_ARCS_MOBILE  = 5;
```

### Accessibility

- Entire globe wrapped in `role="img"` with `aria-label="Interactive globe showing global prayer activity"`
- `aria-describedby` pointing to a visually-hidden summary: "Prayer is active in {n} countries. {total} prayers in the last 24 hours."
- Globe itself is not keyboard-navigable (complex 3D scene) — provide a text-based alternative view (country list table) via "View as table" toggle
- Country click panel: standard focusable panel, all content accessible via keyboard
- Motion: globe rotation respects `prefers-reduced-motion` — stops auto-rotate, disables arc animations
- Error/loading states: visible text equivalents, not icon-only

---

## 10. NotificationCard

### Purpose
A single notification item in the notification centre feed. Must communicate type, actor, content, urgency, and recency at a glance — and link directly to the relevant content.

### TypeScript Interface

```typescript
interface NotificationCardProps {
  // Data
  notification: {
    id: string;
    type: NotificationType;
    title: string;
    body?: string;
    imageUrl?: string;
    actor?: {
      id: string;
      displayName: string;
      avatarUrl?: string;
    };
    deepLink?: string;
    isRead: boolean;
    createdAt: Date | string;
  };

  // Interaction handlers
  onRead?: (id: string) => void;
  onDismiss?: (id: string) => void;
  onClick?: (notification: NotificationCardProps['notification']) => void;

  // Display
  variant?: 'full' | 'compact' | 'toast';
  isLoading?: boolean;
  className?: string;
}

type NotificationType =
  | 'prayer_prayed'   | 'prayer_answered'  | 'prayer_comment'
  | 'friend_request'  | 'friend_accepted'  | 'message'
  | 'group_post'      | 'event_reminder'   | 'testimony_posted'
  | 'subscription'    | 'announcement'     | 'badge_awarded';
```

### States

| State | Behaviour |
|---|---|
| **Unread** | Left border accent in type colour; slightly brighter background (`from-white/10`); bold title |
| **Read** | No left border; standard glass background; normal weight title |
| **Hovered** | Card lifts; dismiss (×) button fades in on right |
| **Dismissed** | Slides out to right with `AnimatePresence` exit animation |
| **Loading** | Avatar circle + two text lines + timestamp shimmer |
| **Toast** | Fixed top-right, auto-dismisses after 5s with progress bar countdown |

### Type → Visual Mapping

| Type | Icon | Accent colour | Left border |
|---|---|---|---|
| `prayer_prayed` | Hands | Gold `#D4AF37` | Gold |
| `prayer_answered` | Star | Emerald `#22C55E` | Emerald |
| `friend_request` | User+ | Blue `#3B82F6` | Blue |
| `friend_accepted` | Users | Blue `#3B82F6` | Blue |
| `message` | Message | Purple `#A855F7` | Purple |
| `event_reminder` | Calendar | Amber `#F59E0B` | Amber |
| `badge_awarded` | Award | Gold `#D4AF37` | Gold shimmer |
| `announcement` | Megaphone | White `#F8FAFC` | White |

### Toast Variant — Specification

```tsx
// Auto-dismissing toast — render in a ToastProvider at root
<motion.div
  layout
  initial={{ opacity: 0, x: 320 }}
  animate={{ opacity: 1, x: 0 }}
  exit={{ opacity: 0, x: 320 }}
  transition={{ type: 'spring', stiffness: 300, damping: 30 }}
  className="w-80 rounded-2xl overflow-hidden
             bg-[#1E293B]/95 backdrop-blur-xl
             border border-white/10
             shadow-[0_8px_32px_rgba(0,0,0,0.6)]"
  role="alert"
  aria-live="assertive"
  aria-atomic="true"
>
  {/* Progress bar auto-dismiss */}
  <motion.div
    className="h-0.5 bg-[#D4AF37] origin-left"
    initial={{ scaleX: 1 }}
    animate={{ scaleX: 0 }}
    transition={{ duration: 5, ease: 'linear' }}
  />
  {/* Content */}
  <div className="flex gap-3 p-4">
    <NotificationIcon type={type} />
    <div className="flex-1 min-w-0">
      <p className="text-sm font-semibold text-[#F8FAFC]">{title}</p>
      {body && <p className="text-xs text-[#94A3B8] mt-0.5 line-clamp-2">{body}</p>}
    </div>
    <button onClick={onDismiss} aria-label="Dismiss notification"
            className="text-[#94A3B8] hover:text-[#F8FAFC] shrink-0">
      <XIcon className="w-4 h-4" />
    </button>
  </div>
</motion.div>
```

### Accessibility

- Notification centre list: `role="list"`, each item `role="listitem"` + `article`
- Unread state: communicated via text "Unread" visually hidden alongside visual indicator
- Toast: `role="alert"` with `aria-live="assertive"` for urgent; `aria-live="polite"` for standard
- Auto-dismiss: a "Keep" button must be present so keyboard/screen reader users aren't timed out
- Dismiss button: `aria-label="Dismiss: {title}"`
- Deep link: `aria-label` describes destination ("View prayer request: {title}")
- Notification feed empty state: `aria-label="No notifications"` on container

---

## 11. MessageBubble

### Purpose
A single message unit within a conversation thread. Supports multiple message types (text, image, prayer card, verse card, voice note) with a consistent visual language for sender vs receiver.

### TypeScript Interface

```typescript
interface MessageBubbleProps {
  // Data
  message: {
    id: string;
    type: 'text' | 'image' | 'prayer_card' | 'verse_card' | 'voice';
    body?: string;
    imageUrl?: string;
    audioUrl?: string;
    audioDurationSec?: number;
    replyTo?: {
      id: string;
      senderName: string;
      bodyPreview: string;
    };
    reactions?: Array<{ emoji: string; count: number; hasReacted: boolean }>;
    isEdited?: boolean;
    deletedAt?: string | null;
    createdAt: Date | string;
    readBy?: string[];   // user IDs
  };

  sender: {
    id: string;
    displayName: string;
    avatarUrl?: string;
  };

  // Context
  isSelf: boolean;           // true if current user sent this message
  isGroupChat?: boolean;
  isFirstInGroup?: boolean;  // first message in a consecutive run from same sender
  isLastInGroup?: boolean;   // last in run — show avatar + timestamp here

  // Interaction handlers
  onReact?: (messageId: string, emoji: string) => void;
  onReply?: (messageId: string) => void;
  onCopy?: (body: string) => void;
  onDelete?: (messageId: string) => void;

  className?: string;
}
```

### States

| State | Behaviour |
|---|---|
| **Sending** | Bubble at 80% opacity; single grey tick icon |
| **Sent** | Full opacity; single grey tick |
| **Delivered** | Double grey tick |
| **Read** | Double gold tick (all recipients read) |
| **Deleted** | Bubble replaced with italic "Message deleted" in muted text; no content |
| **Edited** | "edited" label in `caption` size appended after message body |
| **Hover** | Reaction picker and action menu (Reply, Copy, Delete) fade in |
| **Voice playing** | Waveform bar animates; playhead moves; duration counts up |

### Bubble Styling

```tsx
// Self (sent) — right-aligned, gold-tinted
<div className={cn(
  'max-w-[75%] rounded-2xl px-4 py-2.5',
  isSelf
    ? 'bg-gradient-to-br from-[#D4AF37]/20 to-[#D4AF37]/8 border border-[#D4AF37]/20 ml-auto'
    : 'bg-white/8 border border-white/8 mr-auto'
)}>

// Reply quote — above message body
{replyTo && (
  <div className="border-l-2 border-[#D4AF37]/60 pl-3 mb-2 py-1
                  bg-white/4 rounded-r-lg">
    <p className="text-xs font-semibold text-[#D4AF37]">{replyTo.senderName}</p>
    <p className="text-xs text-[#94A3B8] line-clamp-2">{replyTo.bodyPreview}</p>
  </div>
)}
```

### Prayer Card Embed (type: `prayer_card`)

Renders a compact `PrayerCard` (compact variant) inline in the message, with "Pray Together" CTA replacing the standard engage bar.

### Verse Card Embed (type: `verse_card`)

```tsx
<div className="bg-gradient-to-br from-[#D4AF37]/10 to-transparent
                border border-[#D4AF37]/20 rounded-xl p-4">
  <p className="text-[#F8FAFC] font-serif italic leading-relaxed text-sm">
    "{verseText}"
  </p>
  <p className="text-xs text-[#D4AF37] font-semibold mt-2">{verseReference}</p>
</div>
```

### Accessibility

- Message list: `role="log"` with `aria-label="Conversation messages"` and `aria-live="polite"`
- Each bubble: `role="article"` not appropriate — use `role="listitem"`
- Sender identification: never rely on visual alignment alone — `aria-label` includes sender name
- Tick icons: `aria-label="Sent"` / `"Delivered"` / `"Read by {names}"` — not icon-only
- Deleted message: `aria-label="Message deleted"`
- Edited label: `aria-label="(edited)"` inline after message body
- Reaction picker: `role="toolbar"` with `aria-label="React to message"`
- Voice note: full audio player controls (`play/pause`, progress, duration) — never audio without controls
- Image: meaningful `alt` text or "Image from {senderName}"
- Reply action: `aria-label="Reply to {senderName}'s message"`

---

## 12. TestimonyCard

### Purpose
A celebration card for answered prayers and miraculous stories. Must radiate joy, gold energy, and spiritual authority — this is the platform's most emotionally resonant content type.

### TypeScript Interface

```typescript
interface TestimonyCardProps {
  // Data
  testimony: {
    id: string;
    title: string;
    body: string;
    category: TestimonyCategory;
    mediaUrl?: string;
    author: {
      id: string;
      displayName: string;
      avatarUrl?: string;
      isAnonymous?: boolean;
    };
    linkedRequest?: {
      id: string;
      title: string;
      prayerCount: number;
    };
    gloryCount: number;
    commentCount: number;
    hasGiven?: boolean;
    isFeatured?: boolean;
    createdAt: Date | string;
  };

  // Interaction handlers
  onGlory?: (id: string) => void;
  onUngory?: (id: string) => void;
  onComment?: (id: string) => void;
  onShare?: (id: string) => void;
  onViewLinkedRequest?: (requestId: string) => void;

  // Display
  variant?: 'featured' | 'standard' | 'compact' | 'hero';
  isLoading?: boolean;
  className?: string;
}

type TestimonyCategory =
  | 'healing'     | 'salvation'   | 'provision'  | 'restoration'
  | 'protection'  | 'breakthrough'| 'guidance'   | 'other';
```

### States

| State | Behaviour |
|---|---|
| **Default** | Gold-accented glass card; category badge; "Glory to God" button |
| **Featured** | Gold glass treatment; crown icon; elevated shadow; "Featured" badge |
| **Glory given** | Button fills gold, fires particle burst animation, count increments with spring |
| **Hovered** | Card lifts; full body text revealed (unclamped); share/comment actions appear |
| **Loading** | Shimmer with avatar, two text lines, body paragraph, stat bar |

### Variants

**`featured`** — Gold glass card, full body (up to 8 lines), media image if present, linked prayer request reference ("This prayer was answered after {n} people prayed"), large glory counter. Used in featured sections and homepage.

**`standard`** — Standard glass card, body clamped to 4 lines, category badge, compact stats bar. Used in testimony feed.

**`compact`** — Title + author name + category + glory count + one action. Used in sidebars and profile testimony tabs.

**`hero`** — Full-bleed media background, overlay gradient, title, author, and CTA. Used at top of testimony pages.

### "Glory to God" Button — Specification

```tsx
function GloryButton({
  count, hasGiven, onGlory, onUngory, testimonyId
}: GloryButtonProps) {
  return (
    <motion.button
      onClick={() => hasGiven ? onUngory(testimonyId) : onGlory(testimonyId)}
      whileTap={{ scale: 0.9 }}
      className={cn(
        'flex items-center gap-2 px-4 py-2 rounded-full text-sm font-semibold',
        'transition-all duration-200',
        hasGiven
          ? 'bg-[#D4AF37]/20 text-[#D4AF37] border border-[#D4AF37]/40'
          : 'bg-white/6 text-[#94A3B8] border border-white/10 hover:border-[#D4AF37]/30 hover:text-[#D4AF37]'
      )}
      aria-label={hasGiven ? 'Remove glory' : 'Give glory to God for this testimony'}
      aria-pressed={hasGiven}
    >
      {/* Particle burst on activation — use CSS confetti or canvas overlay */}
      <HandsRaisedIcon className="w-4 h-4" aria-hidden="true" />
      <motion.span
        key={count}
        initial={{ scale: 1.4, opacity: 0 }}
        animate={{ scale: 1, opacity: 1 }}
        transition={{ duration: 0.3, ease: [0.0, 0.0, 0.2, 1] }}
      >
        {count.toLocaleString()}
      </motion.span>
    </motion.button>
  );
}
```

### Linked Prayer Request Reference

When a testimony is linked to an original prayer request, render this callout inside the card:

```tsx
<div className="mt-4 p-3 rounded-xl bg-[#D4AF37]/6 border border-[#D4AF37]/15
                flex items-start gap-3">
  <LinkIcon className="w-4 h-4 text-[#D4AF37] shrink-0 mt-0.5" aria-hidden="true" />
  <div>
    <p className="text-xs text-[#94A3B8]">
      This testimony is linked to a prayer request
      that <strong className="text-[#F8FAFC]">{prayerCount} intercessors</strong> prayed for.
    </p>
    <button onClick={() => onViewLinkedRequest(linkedRequest.id)}
            className="text-xs text-[#D4AF37] hover:underline mt-0.5"
            aria-label={`View original prayer request: ${linkedRequest.title}`}>
      View the prayer → {linkedRequest.title}
    </button>
  </div>
</div>
```

### Accessibility

- `<article>` with `aria-label` of testimony title and author
- "Glory to God" button: `aria-pressed` for toggle state; count announced on change via `aria-live="polite"` on the count span
- Anonymous author: `aria-label` uses "Anonymous believer" — never exposes identity
- Media image: `alt` text is testimony title or author-provided caption
- Category badge: `aria-label` spelling out full category name
- Featured badge: visually hidden text "Featured testimony" for screen readers
- Linked request callout: fully readable by screen reader; action button has descriptive `aria-label`
- Particle burst / confetti: `aria-hidden="true"`, purely decorative

---

## Global Component Standards

All components in this library must adhere to the following rules without exception.

### Prop Conventions

```typescript
// Every component accepts these base props
interface BaseComponentProps {
  isLoading?: boolean;   // triggers skeleton state
  className?: string;    // layout overrides only — not style overrides
  testId?: string;       // for automated testing: data-testid
}
```

### Skeleton Rule

Every component with `isLoading={true}` renders a shimmer skeleton that:
1. Matches the **exact dimensions** of the loaded state
2. Uses `animate-pulse` with `bg-white/8` and `bg-white/6` elements
3. Never shows a spinner — skeletons only

### Animation Rule

All interactive components use Framer Motion. No raw CSS `transition` on hover states for structural transforms. CSS `transition` is acceptable only for `color`, `opacity`, and `border-color`.

### Error Boundary

Every component should be wrapped at the page level in an `<ErrorBoundary>` that renders a contextual error state — never a blank region or white screen.

### Testing Attributes

All interactive elements carry `data-testid` props using kebab-case: `data-testid="prayer-card-pray-button"`.

### Import Convention

```typescript
// Named exports only — no default exports from component files
export { PrayerCard, PrayerCardSkeleton } from '@/components/prayer/PrayerCard';
export type { PrayerCardProps } from '@/components/prayer/PrayerCard';
```
