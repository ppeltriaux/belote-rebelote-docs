# Privacy Policy — Belote et Rebelote

**Effective date:** 2026-05-22
**Last updated:** 2026-05-22
**App:** Belote et Rebelote (iOS)
**Developer:** Pascal Peltriaux
**Contact:** pascal@peltriaux.fr

---

## TL;DR

- We collect the minimum data needed to make live multiplayer work.
- Everything we collect comes from Apple Game Center, which you control via iOS Settings → Game Center.
- Gameplay data (card plays, bids, scores) transits Supabase Realtime as a relay between paired players during a live match. It is **not** retained after the match ends.
- We do **not** sell, share, or use any data for advertising, profiling, or cross-app tracking.
- We do **not** use any third-party analytics, advertising, or attribution SDKs.

If anything below is unclear, email pascal@peltriaux.fr.

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

Gameplay data is **only** exchanged between the 2–4 paired players in your specific match. It is **not** persisted server-side beyond the lifetime of the match.

### 1.3 Match metadata (transient)

To pair you with opponents, the app writes a short-lived row to the `rooms` table on Supabase containing:

- A random match ID (UUID)
- Your `teamPlayerID`
- Game mode (standard / declarations / etc.)
- Match status (open / in-progress / closed)
- Timestamps

Closed match rows are deleted automatically when older than 24 hours.

### 1.4 Game Center achievements + leaderboards

When you win games, earn capots, or hit other milestones, scores and achievements are submitted directly to Apple Game Center. **This data does not touch Supabase.** It is governed by [Apple's Privacy Policy](https://www.apple.com/legal/privacy/) and your Game Center privacy settings.

### 1.5 What we do not collect

We do **not** collect, store, or transmit:

- Your real name, email address, phone number, or postal address
- Your IP address (beyond what's needed to route the live multiplayer connection — Supabase does not log this for our project)
- Device identifiers (IDFA, IDFV, etc.)
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
| **Supabase, Inc.** | Match metadata + transient gameplay broadcast traffic | Real-time multiplayer transport (live data relay only) | [Supabase Privacy Policy](https://supabase.com/privacy) |

We use Supabase as a "data processor" (in GDPR terms): they handle the transport on our behalf and do not use your gameplay data for their own purposes. Supabase is SOC 2 Type II compliant.

We do **not** share data with any other third party. There are no advertising networks, attribution platforms, analytics providers, or data brokers integrated into the app.

---

## 3. How long we keep data

| Data | Retention |
|---|---|
| Game Center identifiers + alias | Only in-memory during a session — re-fetched from Game Center on each app launch |
| Active match row in Supabase `rooms` table | Lifetime of the match + at most 24 hours |
| Gameplay broadcast traffic | Not persisted — relayed in real time then discarded |
| Game Center scores / achievements | Managed by Apple per your Game Center settings |

When a match ends, all gameplay data is gone. When you close the app, the in-memory session is gone.

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

1. **Sign out of Game Center** (iOS Settings → Game Center → toggle off). The app then has no identity to associate with you.
2. **Delete the app** from your device. There's no server-side account to wind down.
3. **Email pascal@peltriaux.fr** if any residual transient match data remains in Supabase and you want it deleted sooner than the 24-hour automatic cleanup window — we'll honour the request within 7 days.

For Game Center data (leaderboards, achievements), use the Game Center app on your device or contact Apple — that data is theirs to manage, not ours.

---

## 5. Children

The app uses Apple Game Center, which requires users to be at least 13 years old in most jurisdictions (per Apple's policy). We do not knowingly collect data from anyone under 13. If you believe a child under 13 has used the app and we hold data about them, email pascal@peltriaux.fr — we will delete it on request.

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

Email: **pascal@peltriaux.fr**

Repository for this document: https://github.com/ppeltriaux/belote-rebelote-docs

App Store listing: (to be updated upon launch)
