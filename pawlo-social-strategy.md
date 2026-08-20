# Pawlo Social Media Strategy — @pawlohq

**Prepared:** 2026-08-20
**Workspace:** Fastlane AI, `pawlohq` (TikTok `@pawlohq`, Instagram `pawlohq`)
**Product:** Pawlo — an app for households (couples, families, roommates) who share pet care, giving them a shared log/reminders for feeding, walks, meds, and chores so nobody has to ask "did anyone feed the dog?"

This strategy is built from a live audit of the `pawlohq` Fastlane workspace, not a blank-slate plan — the account already has active angles, a built content library, and posts on the calendar. The job here is to assess what's there, fix the gaps, and lay out the next month.

---

## 1. Current State Audit

| Area | Status |
|---|---|
| **Connections** | TikTok `@pawlohq` and Instagram `pawlohq`, both OAuth-connected and publishable. Connected 2026-07-15 (~5 weeks ago). |
| **Warmed accounts** | None (0 slot balance). All posting goes through the OAuth connections above. |
| **Posts published** | **0.** Every post so far is `SCHEDULED`, none `POSTED`. The account has been connected for 5 weeks without going live. |
| **Posts scheduled** | 30 (15 TikTok, 15 Instagram), all clustered in a single week: **Sept 12–18, 2026** — three weeks from today. |
| **Content library** | 50+ items already built and `CREATED`: 27 slideshow, 14 wall-of-text, 9 green-screen. None used yet beyond the 30 scheduled. |
| **Active angles** | 4 (below) |
| **Inactive angles** | 6, several of which are strong and worth reviving |
| **Blitz preferences** | slideshow 40 / video-hook 24 / wall-of-text 21 / green-screen 15; remix 50%, own-media 50%, product-mention 50%, influencer chance 50% |

**Two structural problems to fix before anything else:**

1. **No warmup.** The account has been OAuth-connected for 5 weeks with zero posts. Fastlane's own guidance is to warm up gradually (engage without posting, then 1x/day, then 2x/day) before relying on the algorithm to distribute content. Jumping straight to a dense week of 15 posts/platform with no history behind it will get suppressed reach, not a growth spike.
2. **Dead air, then a pile-up.** Nothing is scheduled between now (Aug 20) and Sept 12, then 30 posts land in 7 days. That's the opposite of what performs — steady, spaced cadence beats bursts of volume.

The fix (Section 5) is to start warmup now with the existing content library and spread the already-built 30 posts across the following weeks instead of front-loading them into one.

---

## 2. Positioning

**Core tension Pawlo resolves:** in a shared household, pet care work becomes invisible. One person ends up doing (or double-checking) everything — feeding, walking, meds — while the group chat fails to actually coordinate it. Pawlo turns that mental load into a shared, visible log.

**Voice:** funny-but-real, told from inside the household, not from the brand. The pet is often the punchline (the "tiny scammer" running the two-breakfast hustle); the actual product benefit — visible task history, shared logs, no more double-dosing — is the payoff, not the hook.

**Primary audiences:**
- Couples/partners who cohabit and share a pet
- Roommates splitting pet responsibilities
- Families managing kids + pet care logistics
- Anyone managing a pet's medication schedule across multiple caregivers

---

## 3. Content Pillars

### Keep active (already performing this positioning well)

| Pillar | Angle | Why it works |
|---|---|---|
| **Fair Share Pet Care** | Household scorekeeping / resentment over who does the work | Highest-relatability pain point; broad household appeal |
| **The Invisible Pet Parent** | The default caregiver whose effort goes unseen | Emotional hook, strong for saves/shares/comments ("this is so me") |
| **Who Fed the Dog?** | Miscommunication, double-feeding, missed tasks | Most literal product-use-case content, easy to demo the app naturally |
| **Medication Without Guesswork** | Anxiety over dosing, missed/duplicate meds | Highest-stakes pain point — good for credibility and trust-building |

These four cover the emotional pillar (resentment/invisibility) and the functional pillar (coordination/meds). That's a solid foundation — don't add more serious "problem" angles on top of these four, or the feed starts to feel like a string of complaints.

### Recommend reactivating (for variety and reach — currently inactive)

- **"The Pet Is Running a Scam"** — pure comedy, pet-as-con-artist framing. This is the release valve the current lineup is missing: all 4 active angles are pain-point/problem content. A pure-comedy pillar gets more raw reach and shareability, and it's on-brand (the existing content already uses "tiny scammer" language).
- **"Away From Home"** — travel/date-night anxiety about care happening while you're gone, resolved by remote visibility. Good complementary use case, low overlap with the other 4.

### Leave inactive
- "Roommate Chronicles" and "Invisible Pet Labor" substantially duplicate "Fair Share Pet Care" / "The Invisible Pet Parent" — reactivating them would just split weight across near-identical content.
- "Raising Responsible Pet Helpers" (kids angle) is a fine future expansion but a distraction from the core couples/roommates wedge right now.
- The duplicate "Who Fed the Dog?" (inactive copy) should stay off — it's redundant with the active one.

**Action:** activate the 2 comedy/travel angles, leave the rest as-is. That brings the roster to 6 active angles: 4 serious/functional, 2 lighter, which is a healthier mix for retention.

---

## 4. Content Mix & Format

Current weights (slideshow 40 / video-hook 24 / wall-of-text 21 / green-screen 15) are reasonable. Two adjustments:

- **Green-screen is underused relative to how well it fits this content.** The pet-as-scammer bits work naturally as green-screen (reactive, talking-to-camera format). Consider nudging it to ~20% and trimming video-hook to ~19% — video-hook needs a demo clip stacked with a hook, which is a heavier lift for this content type than the format is earning back yet.
- **Product-mention rate at 50% is high for this stage.** With zero posts published and no earned trust yet, that risks reading as an ad account out of the gate. Drop `mentionBusinessPercentage` to ~25-30% for the warmup month, then raise it once the account has posting history and engagement. Native/relatable content should dominate early; direct app mentions should be the minority, not the majority.

---

## 5. Posting Cadence & Warmup Plan

Starting **this week** (not Sept 12):

| Phase | Weeks | Cadence | Notes |
|---|---|---|---|
| **Warmup** | Week 1 (now) | 1 post/day/platform | Use TikTok inbox posting mode (not direct) for roughly the first 5 TikTok posts, per Fastlane's warmup guidance |
| **Ramp** | Week 2 | 2 posts/day/platform | Switch to TikTok direct posting once inbox posts are landing fine |
| **Steady state** | Week 3+ | 2-3 posts/day/platform | Treat the account as warmed once avg views cross ~500/video |

This reshuffles the existing 30 already-built posts across ~2 weeks instead of cramming them into one, and fills the current Aug 20–Sept 12 dead zone with the 50-item content library that's already sitting `CREATED` and unused.

**TikTok vs Instagram split:** keep parity for now (roughly 1:1), since both are freshly connected. Re-evaluate after 2-3 weeks of data — expect TikTok to outperform Instagram for this format (green-screen/POV pet comedy skews TikTok-native), and shift ratio toward the stronger platform once that's visible.

---

## 6. Conversion / CTA Strategy

Awareness content (the pillars above) should stay CTA-light — the humor and relatability *is* the hook, and over-selling kills reach on both platforms. Concentrate direct asks in a minority of posts:

- Bio link on both profiles should point to the app download/landing page (confirm this is set — not visible via the API).
- Reserve explicit "download Pawlo" CTAs for the ~25-30% product-mention slice (Section 4), ideally attached to the highest-relatability posts (Fair Share Pet Care, Who Fed the Dog?) rather than the pure-comedy pillar.
- Once posting history exists, test a recurring low-key CTA format: a slideshow ending on "this is what Pawlo's log looks like" as the final slide, rather than a hard sell mid-hook.

---

## 7. Measurement

Track weekly once posting starts:

- Views/likes/comments per post, split by platform and by angle (via `POST /analytics/posts`)
- Which of the 6 active angles is outperforming — reweight `angleWeights` toward winners after ~2 weeks of real data instead of leaving them on an even split
- Save/share rate as a proxy for the "this is so me" relatability payoff, since that's the core mechanic this content is betting on
- Time-to-500-views as the warmup graduation marker

---

## 8. Immediate Next Actions

**Update (2026-08-20, follow-up session):** A re-audit before executing this section found the account had drifted materially from the snapshot above — see Section 10. Items 3-4 below were built on a stale premise and were *not* executed. Items 1-2 were re-confirmed against live state and executed.

1. ~~Activate "The Pet Is Running a Scam" and "Away From Home" angles.~~ **Done.** Both angles set `isActive: true` via `update_blitz_angle`.
2. ~~Patch preferences: `mentionBusinessPercentage` → ~25-30%, `greenScreenWeight` → ~20% / `videoHookWeight` → ~19%.~~ **Done.** `update_blitz_preferences` called with `mentionBusinessPercentage: 25`, `greenScreenWeight: 20`, `videoHookWeight: 19` (slideshow 40 / wall-of-text 21 held constant so the four content-type weights still sum to 100).
3. ~~Re-spread the 30 already-scheduled posts from the single Sept 12-18 week across a Week-1/Week-2 warmup cadence...~~ **Not executed — premise was wrong.** There is no pile-up to fix: live state shows 118 posts already scheduled at a near-daily cadence from today through Sept 18, not 30 crammed into one week. See Section 10.
4. ~~Start publishing from the existing 50-item content library to fill the gap, at 1 post/day/platform this week.~~ **Not executed — already happening.** 50 posts have been publishing continuously since 2026-07-15 through today. See Section 10.
5. Confirm the app-download link is live in both bio links before the first real post goes out. **Still open** — not verifiable via the API; needs manual confirmation.

---

## 10. Live-State Drift Found in Follow-Up Session (2026-08-20)

A re-audit immediately before executing Section 8 found this document's Section 1 audit no longer matches the live `pawlohq` workspace. The gap is large enough that the original warmup/reschedule plan (Section 5, Section 8 items 3-4) does not apply as written:

| Section 1 claim | Live state at re-audit |
|---|---|
| 0 posts published | **50 POSTED**, continuously from 2026-07-15 through 2026-08-20 (today) |
| 30 scheduled, all in one Sept 12-18 week | **118 SCHEDULED**, spread near-daily from today through Sept 18 — the Sept 12-18 batch is real but is only the newest ~25% of what's queued |
| ~50 content items (27/14/9) | **97 items** (48 slideshow / 32 wall-of-text / 17 green-screen), library roughly doubled |
| — | 1 transient `FAILED` post (`instagram_disconnected_skipped`, succeeded on retry) and 3 TikTok posts sitting `IN_USER_INBOX` unclaimed — worth a look |

Angles and preferences (pre-patch) matched the doc exactly, so only the posting/content state had drifted. Given the account is already mid-campaign with real engagement (~9,500 combined views on the 50 posted items), the warmup framing in Section 5 and the reschedule/backfill actions in Section 8 items 3-4 were skipped rather than executed against outdated assumptions. Angle activation and the preference patch (items 1-2) were re-verified as still correct and applied.

**Open question for next session:** what/who has kept this campaign running since the original audit (a Blitz automation, a scheduled job, another session) wasn't investigated here — worth checking before making further changes, so nothing gets double-scheduled or interrupted.

---

## 9. Security Note

The API key shared in chat for this workspace should be treated as exposed going forward — rotate it from Fastlane Settings → API once this work is done, and prefer setting it as the `FASTLANE_API_KEY` environment variable (as this project's `.mcp.json` already expects) rather than pasting it in conversation next time.
