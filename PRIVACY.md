# Privacy Policy — Belote et Rebelote

> 🇫🇷 [Version française](./PRIVACY.fr.md)

**Effective date:** 2026-05-22
**Last updated:** 2026-06-01
**App:** Belote et Rebelote (iOS)
**Developer:** Pascal Peltriaux
**Contact:** dev@peltriaux.com

---

## TL;DR

- We collect the minimum data needed to make live multiplayer work.
- Your identity comes from Apple Game Center, which you control via iOS Settings → Game Center.
- Gameplay runs over Supabase. During a live match a small amount of game state is briefly stored server-side (so you can rejoin if you drop) and is deleted within 24 hours.
- If you turn on match-availability notifications, we store your Apple push token so we can alert you when a game is forming. You can turn this off anytime.
- We do **not** sell, share, or use any data for advertising, profiling, or cross-app tracking.
- We do **not** use any third-party analytics, advertising, or attribution SDKs.

If anything below is unclear, email dev@peltriaux.com.

---

## 1. Data we collect

### 1.1 Game Center identifiers

When you sign into Game Center to play multiplayer, the app receives from Apple:

| Field | Value | Why we need it |
|---|---|---|
| `teamPlayerID` | Opaque Apple-issued identifier | To identify your seat in a match and bind your Supabase session to a verifiable identity (anti-impersonation) |
| `alias` | Your Game Center display name | To label your nameplate so opponents know who's playing |
| Profile photo (optional) | Your Game Center avatar (if set) | To show a real face on your nameplate (peers see what you'd see in Game Center) |

We never receive your real name, email address, phone number, or Apple ID from Game Center.

### 1.2 Gameplay data (live multiplayer only)

While a match is in progress, the following data transits between paired players via Supabase Realtime as a relay:

- Card plays (which card was played from which seat)
- Bids (trump suit + contract value)
- Round scores and end-of-match scores
- Belote-Rebelote / Capot / declarations announcements

Gameplay data is **only** exchanged between the 2–4 paired players in your specific match. To let you **rejoin a match after a disconnect** (or to recover if the hosting player drops), the current game state is briefly stored server-side in a `room_authority` row during the match. This state, and the `room_members` rows that record which `teamPlayerID` holds which seat, are deleted automatically within 24 hours.

### 1.3 Match metadata (transient)

To pair you with opponents and manage rooms, the app writes short-lived rows to Supabase:

| Table | Contents | Retention |
|---|---|---|
| `rooms` | Random match ID (UUID), creator's `teamPlayerID`, game mode, status, invite code (private rooms), timestamps | Closed/abandoned rows deleted automatically within 24 hours |
| `room_members` | Match ID + your `teamPlayerID` + seat number | Deleted within 24 hours |
| `room_authority` | Current game state for rejoin/recovery (see §1.2) | Deleted within 24 hours |
| `match_queue` | Transient matchmaking entry | Deleted within ~1 hour |

### 1.3a Match-availability notifications (opt-in)

If you turn on **Notifications** in the app (off by default; also requires the iOS notification permission), the app stores a row in a `push_subscriptions` table containing:

- Your `teamPlayerID`
- Your device's **Apple Push Notification service (APNs) token** — an Apple-issued identifier for your device used solely to deliver a push to it
- Which game variants you want to be notified about, and your quiet-hours preference

We use this **only** to send you a notification when a public online match is forming ("a Coinche game is available"). We never use it for advertising or tracking. You can turn it off anytime in the app (which removes/disables the row) or in iOS Settings → Notifications, and stale tokens are deleted automatically when Apple reports them invalid.

### 1.3b "Founding player" record

The first time you play an online match, we store a single row in a `founders` table containing your `teamPlayerID` and a timestamp. This records that you were an early player so we can keep online play free for you if a paid option is introduced later. It is retained until you ask us to delete it (see §4).

### 1.4 Game Center achievements + leaderboards

When you win games, earn capots, or hit other milestones, scores and achievements are submitted directly to Apple Game Center. **This data does not touch Supabase.** It is governed by [Apple's Privacy Policy](https://www.apple.com/legal/privacy/) and your Game Center privacy settings.

### 1.5 What we do not collect

We do **not** collect, store, or transmit:

- Your real name, email address, phone number, or postal address
- Your IP address (beyond what's needed to route the live multiplayer connection — Supabase does not log this for our project)
- Advertising or vendor identifiers (IDFA, IDFV) — the only device identifier we ever store is the optional APNs push token described in §1.3a, and only if you turn notifications on
- Location data (precise or coarse)
- Contacts, calendar, photos, microphone, or camera content
- Browsing history or third-party app usage
- Analytics or crash telemetry (we use no analytics SDK)
- Advertising identifiers

---

## 2. Where data goes

| Recipient | Data shared | Purpose | Their privacy policy |
|---|---|---|---|
| **Apple Game Center** | Your scores, achievements, sign-in state, alias | Multiplayer matchmaking infrastructure (per Apple's GameKit), leaderboards, achievements | [Apple Privacy Policy](https://www.apple.com/legal/privacy/) |
| **Apple Push Notification service** | Your APNs token + the notification text (only if you opt in) | Deliver match-availability notifications to your device | [Apple Privacy Policy](https://www.apple.com/legal/privacy/) |
| **Supabase, Inc.** | Match metadata, transient game state, and (if you opt in) your push token | Real-time multiplayer transport + the small server-side rows described in §1.2–§1.3b | [Supabase Privacy Policy](https://supabase.com/privacy) |

We use Supabase as a "data processor" (in GDPR terms): they handle the transport on our behalf and do not use your gameplay data for their own purposes. Supabase is SOC 2 Type II compliant.

We do **not** share data with any other third party. There are no advertising networks, attribution platforms, analytics providers, or data brokers integrated into the app.

---

## 3. How long we keep data

| Data | Retention |
|---|---|
| Game Center alias + photo | In-memory only — re-fetched from Game Center on each app launch |
| `rooms` / `room_members` / `room_authority` rows (incl. your `teamPlayerID`) | Lifetime of the match + at most 24 hours, then auto-deleted |
| `match_queue` matchmaking entry | At most ~1 hour |
| Gameplay broadcast traffic | Not persisted — relayed in real time then discarded |
| Push subscription (`teamPlayerID` + APNs token), if you opt in | Until you turn notifications off, or the token becomes invalid |
| "Founding player" row (`teamPlayerID` + timestamp) | Until you request deletion (§4) |
| Game Center scores / achievements | Managed by Apple per your Game Center settings |

When a match ends, the transient match rows are cleaned up within 24 hours. The only data that persists longer is the optional push subscription (which you control) and the small founding-player record (deletable on request).

---

## 4. Your rights

Under GDPR, CCPA, and similar laws, you have the right to:

- **Access** — request a copy of any data we hold about you.
- **Delete** — request deletion of any data we hold about you.
- **Rectify** — correct inaccurate data.
- **Object** — refuse certain processing.
- **Portability** — receive your data in a machine-readable format.
- **Withdraw consent** — at any time.

In practice, the easiest way to exercise any of these rights for our app is:

1. **Turn off Notifications** in the app (or iOS Settings → Notifications) to remove your push subscription, and **sign out of Game Center** (iOS Settings → Game Center) so the app has no identity to associate with you.
2. **Delete the app** from your device. There's no password-based account to wind down.
3. **Email dev@peltriaux.com** to have us delete any remaining server-side rows tied to your `teamPlayerID` — including the optional push subscription and the founding-player record, which are the only items not on the automatic 24-hour cleanup. We'll honour the request within 7 days.

For Game Center data (leaderboards, achievements), use the Game Center app on your device or contact Apple — that data is theirs to manage, not ours.

---

## 5. Children

The app uses Apple Game Center, which requires users to be at least 13 years old in most jurisdictions (per Apple's policy). We do not knowingly collect data from anyone under 13. If you believe a child under 13 has used the app and we hold data about them, email dev@peltriaux.com — we will delete it on request.

We do not market the app to children and do not include child-directed content beyond what's typical for a casual card game.

---

## 6. Security

- All traffic between the app and Supabase uses HTTPS (TLS 1.2+).
- Game Center identity is verified server-side via Apple's GameKit RSA-SHA256 identity-verification signature against Apple's public certificate, preventing impersonation.
- Supabase Row-Level Security policies restrict who can read or write each match row to its actual participants.
- We do **not** transmit raw card hands to non-recipient players (trust-based MVP threat model — see "Limitations" below).

### Limitations we disclose

- **Threat model is "casual card game, no real money."** We do not implement betting, currency, or features that would justify a heavier anti-cheat investment. A motivated peer could in principle observe per-recipient broadcast traffic for hands meant for other seats. This is acceptable for a casual game and explicitly disclosed.
- **No server-side gameplay validation.** Dealer authority is trusted per round. Anti-cheat is via deterministic deck-hash verification across all 4 clients.

---

## 7. Changes to this policy

If we change this policy, we will:

1. Update the "Last updated" date at the top.
2. Push the new version to this same repository.
3. For material changes (new data types collected, new third-party recipients), bump the app version and surface the change in the App Store update notes.

You can see the full revision history of this document at the repository linked in the App Store listing.

---

## 8. Jurisdiction

The developer is based in Australia. This policy is governed by the laws of New South Wales, Australia, without regard to conflict-of-laws principles.

For users in the EU/EEA, you may contact your local data protection authority if you have unresolved concerns. For users in California, our practices comply with the CCPA — we do not "sell" or "share" personal information as those terms are defined.

---

## 9. Contact

Email: **dev@peltriaux.com**

Repository for this document: https://github.com/ppeltriaux/belote-rebelote-docs

App Store listing: (to be updated upon launch)
