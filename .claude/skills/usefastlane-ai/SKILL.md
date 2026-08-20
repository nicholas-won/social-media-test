---
name: usefastlane-ai
description: Use for Fastlane AI at usefastlane.ai, the short-form content platform and REST/MCP API for generating, building, remixing, scheduling, and analyzing organic TikTok, Instagram Reels, and YouTube Shorts content. Use when an agent needs to plan or automate Fastlane campaigns, call the Fastlane API, create content angles, tune Blitz preferences, build content from explicit payloads, upload media, create influencers, schedule or cancel posts, inspect analytics, design account warmup/posting strategy, or build reusable prompt/workflow files for Fastlane. Do not use for fastlane.tools mobile app deployment automation.
---

# UseFastlane AI

This skill is an operational index.
It gives the model enough context to start safely, then tells it to use the Fastlane API as the live source of truth for workspace data, schemas, states, and examples.
It works from `SKILL.md` alone in any agent that can read Markdown and call HTTP APIs.

Full reference docs live at `https://developers.usefastlane.ai`.

## What Fastlane Is

Fastlane (`usefastlane.ai`) is an AI short-form marketing platform.
It learns a product/business from its website and profile, generates or renders short-form content, and schedules it across TikTok, Instagram Reels, YouTube Shorts, and API-supported destinations.

Core concepts:

- **Workspace**: company context. API keys are scoped to the workspace that was active when the key was created. A key does not follow the current-workspace selection in the UI.
- **Blitz**: the AI content-generation loop. Fastlane keeps a discovery queue of AI suggestions. `POST /blitz` pops one suggestion and starts an async rendered media build.
- **Angles**: reusable personas/framing prompts that guide generation. Capped at 100 per workspace.
- **Preferences**: content-type mix, remix ratio, own-media ratio, product mention rate, gender filter, and per-angle weights.
- **Content**: generated or rendered media in the library. Created by Blitz, by explicit build payloads, or by custom video upload.
- **Media Bank**: workspace-owned images and videos. Build payloads only accept media Fastlane already holds, so uploads go here first.
- **Audio library**: Fastlane's shared, read-only audio catalogue. The source of `audio.url` for `wall-of-text` and `video-hook` payloads.
- **Influencers**: AI personas trained from a base image. Can be linked to angles. Capped at 50 active per workspace.
- **Connections**: OAuth-connected social accounts owned by the user.
- **Warmed accounts**: Fastlane-provisioned TikTok/Instagram accounts that post without the user connecting their own OAuth account.
- **Posts**: scheduled, inbox-delivered, published, failed, or deleted records.
- **Analytics**: synced engagement metrics for posts.

Core content types:

- `slideshow`: image slides with strong hook. Up to 20 slides.
- `wall-of-text`: bold overlay text over a background video, where the hook is the content.
- `video-hook`: hook clip stacked with a demo clip.
- `green-screen`: green-screen video composited over a background image.
- `custom`: uploaded video content.

Use this for `usefastlane.ai`, not `fastlane.tools`.

## Two Doors Onto The Same Workspace

- **REST API**: `https://api.usefastlane.ai/api/v1`. Every endpoint in this skill's index, but not the MCP-only automation tools. Limited to 20 requests/minute per workspace.
- **MCP server**: `https://api.usefastlane.ai/mcp`, JSON-RPC 2.0 over a single stateless POST, authenticated with the same `fsln_live_` key as a Bearer token. Limited to 30 requests/minute per user, in a rate-limit bucket separate from the REST one.

The two doors share workspace data and the core business rules: the same content-save pool, daily Blitz swipe cap, credits, and platform post limits apply whichever door a call comes through.
They are not the same surface.
Each transport validates its own request shape, so field names, defaults, and rejection messages can differ, and neither surface is a superset of the other.
Multi-week **automation** campaigns (`create_automation`, `generate_automation`, `launch_automation`, and the slot review tools) are MCP-only.
Media Bank uploads, the audio library, explicit `POST /content` builds, custom video upload, and warmed-account listing are REST-only; fetch `warmedAccountId` values from `GET /api/v1/warmed-accounts`.

Register it in Claude Code without writing the key to disk:

```bash
claude mcp add --transport http fastlane \
  https://api.usefastlane.ai/mcp \
  --header 'Authorization: Bearer ${FASTLANE_API_KEY}'
```

The single quotes matter: they stop the shell expanding the variable, so the stored config holds the literal `${FASTLANE_API_KEY}` placeholder and Claude Code resolves it from the environment on each connection.
Double quotes bake the real key into the config file in plaintext.
The same placeholder works in a project `.mcp.json`, which is the form to prefer when the config is committed:

```json
{
  "mcpServers": {
    "fastlane": {
      "type": "http",
      "url": "https://api.usefastlane.ai/mcp",
      "headers": { "Authorization": "Bearer ${FASTLANE_API_KEY}" }
    }
  }
}
```

Claude.ai and Claude Desktop cannot add the MCP server yet: their custom-connector UI only accepts OAuth servers, and Fastlane MCP authenticates with an API key. Use Claude Code, Codex, or plain REST.

## Safety

Use `FASTLANE_API_KEY` from the environment.
Never store API keys in files and never print them.
If the user pasted a key, treat it as compromised after the session and recommend rotation.
Keys are shown once at creation and can be revoked from Settings → API.

Confirm before destructive, billable, or externally visible actions unless the user already explicitly requested them:

- `POST /content` (billable, consumes a content save)
- `POST /content/custom/video` (billable, consumes a content save)
- `POST /blitz` (billable, consumes a swipe and a content save)
- `POST /influencers` (consumes credits)
- `POST /content/:id/schedule` (externally visible)
- `DELETE /content/:id`
- `POST /posts/cancel`
- `POST /posts/delete`
- `DELETE /blitz/angles/:id`

Blitz is not idempotent and has no dry-run or queue-preview endpoint.
Do not auto-retry an ambiguous timeout or `5xx` from `POST /blitz`: the original request may already have accepted and billed a suggestion.

`POST /content` and `POST /influencers` require an `Idempotency-Key` header.
A retry is only safe when it resends the **same** key with the same body, which replays the original result; a fresh key on a retry creates and bills a second item.
Generate one key per intended item, store it next to the payload, and reuse it for every retry of that item.
Reusing a key with a different body returns `409 conflict`, as does a retry that lands while the first call is still in flight.

Send a real `User-Agent`.
Python `urllib` defaults can receive a Cloudflare `403` before reaching Fastlane, and media hosts may reject them on download.

## API Starting Point

Base URL:

```text
https://api.usefastlane.ai/api/v1
```

Headers:

```text
Authorization: Bearer $FASTLANE_API_KEY
Accept: application/json
Content-Type: application/json; charset=utf-8
User-Agent: usefastlane-ai-agent/1.0
```

Keys look like `fsln_live_` plus 32 random base62 characters.
All `401` responses carry `WWW-Authenticate: Bearer realm="api"`.

Envelopes:

```json
{ "data": {} }
```

```json
{ "data": [], "pagination": { "cursor": "opaque_or_null", "hasMore": true } }
```

```json
{ "error": { "code": "...", "message": "...", "details": {} } }
```

## Rate Limits And Quotas

- **Workspace API limit**: 20 requests/minute, token bucket, refilled evenly across each 60-second window. On `429`, respect `Retry-After` or `details.retryAfterMs`.
- **Sub-limits on top of that**: preference updates 3/minute, angle generation 5 successful suggestions/minute, content builds 10/minute.
- **MCP**: 30 requests/minute per user, independent of the REST limit.
- **Blitz swipe cap**: each successful `POST /blitz` counts against the workspace's daily swipe cap. Exhaustion returns `blitz_quota_exceeded` with `details.resetAt` (UTC epoch ms, refills at 00:00 UTC).
- **Content saves**: every content-creating call (blitz, `POST /content`, custom video finalize) consumes one save from the workspace pool, included monthly saves first, then purchased extra saves. Exhaustion returns `content_quota_exceeded` unless API overage billing is enabled in Settings → API and the subscription is live, in which case the call is accepted and metered as overage. A failed build refunds the save.
- **Scheduling and publishing** from a regular workspace key are not usage-metered, though a warmed-account post still consumes one prepaid post slot at schedule time.
- **Platform limits** surface as distinct codes: `tiktok_post_limit_exceeded`, `youtube_post_limit_exceeded`, `instagram_post_limit_exceeded`.

## Endpoint Index

Use this index first.
If more detail is needed, inspect the live API with safe read calls.

| Area        | Endpoint                                  | Purpose                                                           |
| ----------- | ----------------------------------------- | ----------------------------------------------------------------- |
| Blitz       | `POST /blitz`                             | Pop or force a suggestion and start an async build                |
| Blitz       | `GET /blitz/preferences`                  | Read generation preferences                                       |
| Blitz       | `PATCH /blitz/preferences`                | Update generation preferences                                     |
| Angles      | `GET /blitz/angles`                       | List all angles, active and inactive                              |
| Angles      | `GET /blitz/angles/page`                  | Paginated angle list (`limit` 1-100, `cursor`)                    |
| Angles      | `POST /blitz/angles`                      | Create angle                                                      |
| Angles      | `PATCH /blitz/angles/:id`                 | Update angle                                                      |
| Angles      | `DELETE /blitz/angles/:id`                | Deactivate angle (soft delete)                                    |
| Angles      | `POST /blitz/angles/generate`             | Generate one unpersisted AI angle suggestion                      |
| Influencers | `GET /influencers`                        | List active, trained influencers                                  |
| Influencers | `GET /influencers/:id`                    | Poll one influencer's training status                             |
| Influencers | `POST /influencers/base-image/upload-url` | Reserve a base-image upload                                       |
| Influencers | `POST /influencers`                       | Create influencer, async training                                 |
| Connections | `GET /connections`                        | List OAuth social accounts                                        |
| Warmed      | `GET /warmed-accounts`                    | List warmed TikTok/Instagram accounts and slot balance            |
| Media       | `POST /media/upload-url`                  | Reserve a Media Bank upload                                       |
| Media       | `POST /media`                             | Finalize a Media Bank asset                                       |
| Media       | `GET /media/:id`                          | Poll a Media Bank asset's processing status                       |
| Audio       | `GET /audio`                              | List shared audio tracks (`for=wall-of-text` or `for=video-hook`) |
| Content     | `POST /content`                           | Build content from an explicit payload                            |
| Content     | `POST /content/custom/video/upload-url`   | Reserve a custom video upload                                     |
| Content     | `POST /content/custom/video`              | Finalize a custom video into content                              |
| Content     | `GET /content`                            | List content                                                      |
| Content     | `GET /content/:id`                        | Fetch content item                                                |
| Content     | `GET /content/:id/editable`               | Fetch safe editable copy and presentation values                  |
| Content     | `PATCH /content/:id`                      | Rebuild the same item with sparse text/presentation edits          |
| Content     | `POST /content/:id/regenerate`             | Regenerate copy while preserving media and presentation           |
| Content     | `DELETE /content/:id`                     | Delete content                                                    |
| Content     | `POST /content/:id/schedule`              | Schedule content                                                  |
| Posts       | `GET /posts`                              | List posts                                                        |
| Posts       | `GET /posts/:id`                          | Fetch post                                                        |
| Posts       | `POST /posts/cancel`                      | Cancel scheduled posts (all-or-nothing, max 100)                  |
| Posts       | `POST /posts/delete`                      | Delete posts from lists and analytics (all-or-nothing, max 100)   |
| Analytics   | `POST /analytics/posts`                   | Batch post metrics (max 100 ids)                                  |

A separate `/api/v1/partner/*` surface exists for white-label multi-tenant integrations.
It needs a Partner key and is out of scope for a normal workspace agent.

## Blitz Contract

`POST /blitz` pops one ready suggestion, starts an async build, and returns `202` with `data.contentId`, `data.suggestion`, and `data.swipesRemaining`.
Builds take roughly 30 seconds to a couple of minutes.
Poll `GET /content/:id` every ~10 seconds until `status` is `CREATED` or `FAILED`.

The body is optional.
Two overrides are supported:

```json
{ "contentType": "slideshow", "aspectRatio": "9:16" }
```

- `contentType`: one of `slideshow`, `wall-of-text`, `green-screen`, `video-hook`. When set, Blitz skips the queue and generates that exact type on demand.
- `aspectRatio`: one of `1:1`, `4:5`, `3:4`, `9:16`. Defaults to `4:5`. Applies to `slideshow` output only.

Errors worth handling:

- `404 not_found`: the discovery queue is empty. Retry shortly. Does not apply when `contentType` is passed.
- `422 content_unavailable_for_type`: valid type, but this workspace cannot produce it right now (for example `video-hook` with no demo video). Try another type.
- `429 blitz_quota_exceeded`, `429 content_quota_exceeded`, `429 rate_limited`.

## Preferences Contract

```json
{
  "slideshowWeight": 40,
  "wallOfTextWeight": 20,
  "greenScreenWeight": 20,
  "videoHookWeight": 20,
  "remixPercentage": 50,
  "ownMediaPercentage": 50,
  "mentionBusinessPercentage": 30,
  "influencerChance": 50,
  "genderFilter": null,
  "angleWeights": { "<angleId>": 100 }
}
```

Rules:

- The four content-type weights are a unit. Send all four or none, and they must sum to exactly 100. A partial set returns `400 bad_request`.
- `angleWeights` is a full replacement and must cover every active angle exactly once, summing to 100.
- `influencerChance` is an integer 0-100, default 50. It is the probability that a given suggestion is influencer-flavored, and it only has an effect when the workspace has at least one trained influencer.
- `genderFilter` is `"man"`, `"woman"`, or `null` to clear. Omitting it leaves the stored value alone.
- An empty body returns `400 bad_request`.
- Preference changes apply to future generations only. They never flush or reset the queue, so already-queued suggestions still reflect the older settings.
- `PATCH` echoes the full post-update state, so no read-back call is needed.

Creating an angle, flipping `isActive`, or deleting an angle rewrites stored `angleWeights` to an even split summing to 100 across the new active set.
To keep custom weights, follow the angle mutation with an explicit `PATCH /blitz/preferences`.

## Angles Contract

Create:

```json
{
  "title": "Product education",
  "description": "Explain how the product solves a specific problem.",
  "targetAudience": "Small business owners",
  "influencerIds": ["<influencerId>"],
  "expectedBrandRevision": 3
}
```

The three strings are required and non-empty (max 200 / 2000 / 500 chars).
`influencerIds` is optional, must be unique, and must reference active, trained influencers in this workspace.
Each influencer can be linked to at most 10 angles.

`POST /blitz/angles/generate` returns an unpersisted suggestion plus a `brandRevision`.
Pass that value back as `expectedBrandRevision` when saving so stale positioning cannot be persisted after a Brand refresh.

`PATCH` accepts any subset. `influencerIds` is a full replacement: omit to keep links, send `[]` to clear them.
Deactivating an angle clears its influencer links, and reactivating does not restore them.
`DELETE` is a soft delete: the row survives and can be reactivated with `PATCH`.

## Media And Build Contract

`POST /content` renders content from an explicit payload, giving full control where Blitz gives AI decisions.
It requires an `Idempotency-Key` header, returns `202` with `data.contentId`, and consumes one content save.

Every media URL in a payload is fetched server-side at build time, so **only media Fastlane already holds is accepted**: your own Media Bank uploads or Fastlane's shared library.
An arbitrary external URL is rejected with `400 bad_request` before any build starts.

Upload order for your own assets:

1. `POST /media/upload-url` with `filename`, `contentType`, `sizeBytes` (images `image/jpeg|png|webp` up to 10 MB, videos `video/mp4|quicktime` up to 50 MB, optional video-only `durationSec`).
2. `PUT` the bytes to the returned `uploadUrl` with the returned headers.
3. `POST /media` with the `uploadId` and optional `categories` (images: `slideshow-image`, `green-screen-background`; videos: `background-video`, `hook-video`, `green-screen-video`). It returns only `mediaId` and `status`.
4. `GET /media/:id` to read `r2Url`. This call is required for every asset, images included: the finalize response never carries a URL.
5. Images finalize as `ready` immediately, so one `GET` is enough. Videos finalize as `processing` and normalize in the background, so keep polling `GET /media/:id` until `ready` or `failed`.
6. Use the `r2Url` from step 4 in build payloads.

Audio cannot be uploaded.
Pull `audio.url` from `GET /audio?for=wall-of-text` or `GET /audio?for=video-hook`.

Body shape:

```json
{ "contentType": "wall-of-text", "contentData": {} }
```

Shared text object, used by every type:

```json
{
  "content": "hook text",
  "x": 100,
  "y": 200,
  "width": 800,
  "fontSize": 48,
  "color": "#ffffff",
  "textAlign": "center",
  "fontWeight": 700,
  "strokeWidth": 5,
  "strokeColor": "#000000",
  "backgroundColor": "#000000"
}
```

`canvas` is the output size in pixels, and each media element carries a draw rect (`drawX`, `drawY`, `drawW`, `drawH`) placing it on that canvas.

- `wall-of-text`: `canvas`, `video`, `audio`, `text`.
- `green-screen`: `canvas`, `image` (background), `video` (green screen), `text`.
- `video-hook`: `canvas`, `hook_video`, `demo_video`, `audio`, `text`.
- `slideshow`: `num_images` plus an `images` array, each entry with its own `canvas`, `image`, and `texts` array. `num_images` must match the array length, max 20. Every slideshow `canvas` requires `aspectRatio` alongside `width` and `height`: one of `1:1`, `4:5`, `3:4`, `9:16`. Omitting it fails validation before the build starts. The other content types take a `canvas` of `width` and `height` only.

Slideshow rendering stretches source images to the target canvas, so pre-crop them to the target ratio or expect distortion.

Content created through `POST /content` has no saved editor state and is not editable in the in-app Studio, but supported payloads can use the REST content-edit endpoints below.

For a plain uploaded video, the order is `POST /content/custom/video/upload-url`, then `PUT` the bytes to the returned `uploadUrl` with the returned headers, then `POST /content/custom/video` with the `uploadId`.
Finalizing before the `PUT` completes returns `409 conflict` ("Uploaded video is not reachable yet"), because the reservation has no object behind it.
Finalize creates `custom` / `video` content and returns `status: "BUILDING"`, so poll to `CREATED` before scheduling.

## Content Read Contract

`GET /content` and `GET /content/:id` return `_id`, `type`, `status`, `files`, `thumbnailUrl`, and optional `generatedText`.

`generatedText` is the current extractable copy: hook, caption, or slide text.
It can appear while content is `BUILDING`, `CREATED`, or `FAILED`, and is omitted when no stored copy exists.
Editable Studio copy takes precedence over the original build copy, whitespace is normalized, duplicates are removed, and output is capped at 2,000 characters.
Raw `studioState` and `buildPayload` are private and never returned.

List filters: `limit` (1-100, default 20), `cursor`, `type`, and `status` as a comma-separated list of `BUILDING`, `CREATED`, `FAILED`.

## Content Editing Contract

`GET /content/:id/editable` supports `slideshow`, `wall-of-text`, `green-screen`, and `video-hook` content. It returns a `revision`, canvas dimensions, and safe `textBoxes`; it never returns raw build state or media URLs. Slideshow text boxes have stable IDs such as `slide-0-text-1` and can be edited independently.

`PATCH /content/:id` requires an `Idempotency-Key` and `{ "revision", "textBoxes": [...] }`. Each text-box entry requires its returned `id` and may sparsely change `text`, `position`, `widthPercent`, `font`, `color`, `stroke`, or `background`. Unknown fields and stale revisions are rejected atomically.

`POST /content/:id/regenerate` also requires an `Idempotency-Key` and accepts `{ "revision", "prompt"? }`. It regenerates every text box while preserving media, layout, and styling. The prompt is optional and capped at 500 characters.

Both writes return `202` with the same `contentId`, a new revision, and `status: "BUILDING"`. Poll `GET /content/:id` normally. These minor edits create no new row or content-save charge; failed renders restore the last successful item. Content with linked posts, unsupported types, a non-ready state, or ended paid access cannot be edited. A request that fails before acceptance can be retried with the same idempotency key.

`DELETE /content/:id` soft-deletes the row and best-effort removes R2 objects.
It returns `409 content_has_linked_posts` when posts reference the content (delete those first; cancelling retains the reference and there is no cascade), `409 content_still_building` for content under 15 minutes old that is still building, and `409 content_edit_in_progress` while a minor edit is rebuilding.
Deleting content does not refund the content save.

## Influencers Contract

1. `POST /influencers/base-image/upload-url` with `filename`, `contentType` (`image/jpeg`, `image/png`, `image/webp`, extension must match), `sizeBytes` (max 10 MB).
2. `PUT` the image to `uploadUrl`.
3. `POST /influencers` with an `Idempotency-Key` header and `{ "name", "gender", "age", "ethnicity", "baseImageUploadId" }`. Returns `202` with `status: "generating_variations"`.
4. Poll `GET /influencers/:id` until `status` is `ready` or `failed`. `progress` is 0-100.

`GET /influencers` lists only active, trained influencers, so a new one appears there only after training finishes.
Cap is 50 active per workspace.
Insufficient credits, the cap, or a storage-quota block all return `402 payment_required`.
Reusing an idempotency key with a different body returns `409 conflict`.

## Scheduling Contract

The OAuth and warmed-account routes take different field sets, and mixing them is rejected.
Start from whichever of these two shapes applies, and do not merge them.

OAuth connection, TikTok direct post:

```json
{
  "platform": "tiktok",
  "utc_datetime": "2026-09-01T18:00:00Z",
  "caption": "...",
  "description": "...",
  "enableCaptionGeneration": true,
  "connectionId": "...",
  "posting_mode": "direct",
  "privacy_level": "PUBLIC_TO_EVERYONE"
}
```

Warmed account, Instagram slideshow:

```json
{
  "platform": "instagram",
  "utc_datetime": "2026-09-01T18:00:00Z",
  "caption": "...",
  "warmedAccountId": "...",
  "disclose_as_ads": true,
  "ai_content_disclaimer": true,
  "instagram_slideshow_format": "carousel"
}
```

- `platform` is `tiktok`, `instagram`, or `youtube`.
- `utc_datetime` accepts an ISO-8601 string or UTC epoch milliseconds. It must be in the future.
- **Scheduling window**: an OAuth-connection post can be scheduled up to **6 weeks** ahead. A warmed-account post can be scheduled up to **30 days** ahead. Both ceilings are enforced server-side and both come back as `400 bad_request`, so clamp campaign plans to the tighter of the two when a plan mixes routes.
- `connectionId` is optional only when exactly one active connection exists on that platform. Otherwise `400 connection_ambiguous` comes back with `candidates`.
- `connectionId` and `warmedAccountId` are mutually exclusive. Warmed-account scheduling supports `tiktok` and `instagram` only; `youtube` stays OAuth-only.
- `enableCaptionGeneration` defaults to `false`. When `true`, Fastlane fills blank or missing caption/description for `instagram`, `tiktok`, and `youtube`. Explicit non-blank values always win. Generated text is not echoed in the response, so read it back with `GET /posts/:id`.
- `posting_mode` is OAuth TikTok only: `inbox` (default, lands in the creator's TikTok drafts) or `direct` (publishes to the feed, requires the `tiktokDirect` entitlement and a `privacy_level`). Sending it for another platform or alongside `warmedAccountId` is a `400`.
- `privacy_level` is required with `posting_mode: "direct"` and must be a value TikTok's Creator Info API allows for that connection. Sandbox or unaudited apps may be limited to `SELF_ONLY`.
- `disclose_as_ads` requires `warmedAccountId` and enables the paid-partnership disclosure.
- `ai_content_disclaimer` enables the platform AI-content label. Supported on warmed posts, provider-routed Instagram posts, and connected TikTok direct video posts. Native Instagram, connected TikTok slideshows, and TikTok inbox posts return `400 bad_request`. The label cannot be changed after publishing.
- `instagram_slideshow_format` is warmed Instagram slideshows only: `carousel` (default) or `reel`. It requires `warmedAccountId` and is ignored for TikTok and non-slideshow content.

The response returns `postId`, plus `posting_mode` / `privacy_level` only when applicable and explicitly supplied, plus `accountType` and `warmedAccountId` for warmed routing.
Absent fields mean "server default", which is inbox for TikTok.

`GET /warmed-accounts` returns up to 100 accounts with `warmedAccountId`, `platform`, `username`, `status`, `postable`, and a per-account `minLeadTimeMs`, plus a top-level `slotBalance` and `minLeadTimeMs`.
Schedule against the **per-account** `minLeadTimeMs`: accounts differ, some accept posts a few hours out and others need about two days.
The top-level `minLeadTimeMs` exists for backwards compatibility and always reports the most conservative floor, so using it can delay a post that the chosen account would have taken sooner.
Scheduling a warmed post inside that account's lead time returns `400 bad_request`.
`402 payment_required` also covers running out of warmed-account post slots.

Post statuses: `BUILDING`, `CREATED`, `SCHEDULED`, `UPLOADING_TO_TIKTOK`, `IN_USER_INBOX`, `POSTED`, `FAILED`, `DELETED`, `AWAITING_RECONNECT`.
`AWAITING_RECONNECT` means the OAuth connection was inactive at execution time; it auto-resumes to `SCHEDULED` when the user reconnects, or moves to `FAILED` if the slot has passed.
`DELETED` posts are excluded from `GET /posts` unless requested in `status`.
When Fastlane safely adjusted an Instagram caption, `caption` keeps the user's original copy and `publishedCaption` holds the live copy, with `captionAdjustmentReason` explaining why.

## Cancel, Delete, Analyze

Cancel and delete are atomic: if any id in the batch fails validation, nothing changes.

```json
{ "postIds": ["..."] }
```

- `POST /posts/cancel` only accepts `SCHEDULED` posts, max 100 ids. Failures come back per id in `details.failures` as `not_found`, `not_cancellable_status:<status>`, `past_cutoff` (within 30 seconds of the scheduled time), `past_submit_cutoff` (a warmed post already handed off for publishing, roughly 36 hours before its slot), or `task_running`. Warmed posts therefore stop being cancellable long before OAuth posts do.
- `POST /posts/delete` marks rows `DELETED` so they leave lists and analytics. It does not delete the linked content and does not remove already-published media from the platform. Already-deleted ids are no-ops.
- `POST /analytics/posts` takes up to 100 ids and returns `views`, `likes`, `comments`, `postUrl`, `postedAt`. Unknown or cross-workspace ids come back as `{ "_id": "...", "notFound": true }`; malformed ids fail the whole request with `400 bad_request` and an `invalidIds` list. Metrics reflect the last platform sync, so treat zeros on very recent posts as pre-sync and check `postedAt`.

## Live Discovery Protocol

When the model needs more than this index provides, connect to the API and learn from safe reads.
Do not guess fields when the API can show them.

Recommended first pass:

1. `GET /blitz/preferences`
2. `GET /blitz/angles`
3. `GET /connections`
4. `GET /warmed-accounts`
5. `GET /influencers`
6. `GET /content?limit=5`
7. `GET /posts?limit=5`
8. If content exists, `GET /content/:id` for one representative item.
9. If posts exist, `GET /posts/:id` and `POST /analytics/posts` for known post ids.

Summarize schema by field names, enum/status values, counts, and error codes.
Avoid exposing private ids, usernames, captions, generated text, or media URLs unless the user needs them.

Do not use write endpoints as schema probes unless the user asked to mutate state.
If a write fails during a real task, use the returned `error.code`, `message`, and `details` to update the plan.

## Automation Artifacts

When building an autonomous campaign in a local project, create:

```text
fastlane/
  campaign-brief.md
  prompts.md
  angles.json
  preferences.json
  schedule.json
  api-log.jsonl
  metrics-snapshot.json
  scripts/
    fastlane_api.py
    inspect_workspace.py
    configure_generation.py
    generate_blitz_batch.py
    build_content.py
    upload_media.py
    poll_content.py
    download_media.py
    qa_slideshow.py
    schedule_content.py
```

Never store API keys.
Keep `api-log.jsonl` sanitized: endpoint, method, status, counts, redacted error summaries, and ids only where needed.
Generated suggestion text, captions, media URLs, and raw error bodies belong in the run artifacts under `fastlane/runs/<timestamp>/`, never in the log.

`fastlane/runs/` holds customer content and signed media URLs, so add it to `.gitignore` before the first run and verify with `git status` that no run artifact is staged.
Sanitize anything that leaves that folder: strip `Authorization` headers, key fragments, and signed URL query strings from every error body you copy into a log, a summary, or a chat reply.
Report the error `code` and `message`, not the raw response.

For real generation work, prefer creating this local toolkit over one-off manual HTTP calls.
Keep it generic enough to rerun:

- `fastlane_api.py`: shared client. Auth headers, pagination, rate-limit backoff, idempotency-key generation, sanitized logging.
- `inspect_workspace.py`: the safe reads from the Live Discovery Protocol.
- `configure_generation.py`: snapshot preferences, patch weights, restore the snapshot afterwards.
- `upload_media.py`: reserve, `PUT`, finalize, then `GET /media/:id` for `r2Url`.
- `generate_blitz_batch.py`, `build_content.py`, `poll_content.py`: create content and poll to `CREATED` or `FAILED`, one idempotency key per intended build.
- `download_media.py`, `qa_slideshow.py`: fetch rendered media with a browser-like `User-Agent` and check type, file count, and slide structure.
- `schedule_content.py`: schedule only after explicit user approval, then verify.

Scripts should read `FASTLANE_API_KEY` from the environment, write results into `fastlane/runs/<timestamp>/`, and be safe to rerun without losing the original preference snapshot.

## Workflow: Inspect Workspace

1. Run the Live Discovery Protocol.
2. Report connected platforms, warmed accounts and slot balance, active angles, influencers, preferences, content counts by type/status, and post counts by platform/status.
3. Flag likely issues: no connections, no active angles, failed content, posts stuck in `AWAITING_RECONNECT`, scheduled posts near platform limits, weak content mix, missing analytics.

## Workflow: Generate Content With Blitz

1. Inspect preferences and angles.
2. Optionally create or update campaign angles, using `POST /blitz/angles/generate` for ideas.
3. Optionally patch preferences. Remember these apply to future generations only.
4. Call `POST /blitz` for the requested quantity, passing `contentType` when the user wants a specific format.
5. Poll `GET /content/:id` every ~10 seconds until the status leaves `BUILDING`.
6. QA `CREATED` items by type, file count, thumbnail, optional `generatedText`, and campaign fit.
7. If `FAILED`, report and decide whether to retry. The content save was refunded.

Blitz findings from live use:

- Freshly patched angles or preferences do not affect already-queued suggestions, so the first render after a change can still reflect the old setup. Do not trust it blindly.
- A `404` from Blitz means the queue is momentarily empty. Wait and retry, or pass `contentType` to generate on demand.
- For slideshows, verify both the suggestion text and the rendered images. A five-line suggestion can become one cover plus four advice slides; if the user wanted five actual tips, ask for "no cover, exactly five slides, each slide is one numbered tip".
- Keep `mentionBusinessPercentage` low when the content should feel native, and raise it deliberately for product-led tests.
- Restore the previous preferences after a focused run unless the user wants the workspace left tuned for that campaign.

## Workflow: Build Content From A Payload

Use when the user supplies their own media, wants exact text placement, or wants deterministic output instead of AI decisions.

1. Upload assets through the Media Bank and wait for `ready`.
2. Pull an `audio.url` from `GET /audio?for=...` for `wall-of-text` or `video-hook`.
3. Assemble `contentData` for the chosen type, keeping draw rects inside the canvas.
4. `POST /content` with a fresh `Idempotency-Key`.
5. Poll `GET /content/:id` to `CREATED`, then inspect `files` as the preview step.
6. Schedule or `DELETE` based on QA.

## Workflow: Automatic Campaign Engine

Use when asked for a weekly workflow, content engine, or "do it all" campaign.

1. **Brief**: product, ICP, pains, outcomes, proof, offer, tone, claims to avoid, platforms.
2. **Angles**: produce 3-8 angle objects and create or update them via `/blitz/angles`.
3. **Preferences**: choose content-type weights and remix/own-media/mention rates.
4. **Generate and poll**: call Blitz for the requested volume, respecting the rate limit, swipe cap, and content-save pool, then wait for `CREATED` or `FAILED`.
5. **QA**: recommend schedule, regenerate, or discard per item.
6. **Schedule**: read `/connections` and `/warmed-accounts`, convert times to UTC, then schedule only when explicitly requested.
7. **Monitor and iterate**: fetch posts and analytics, then adjust angles and preferences based on winners.

If the user has MCP access and wants a multi-week campaign managed for them, the MCP automation tools do steps 2-7 as one campaign object.

## Workflow: Schedule Existing Content

1. `GET /connections` and `GET /warmed-accounts`; confirm the target account can publish (`publishable` or `postable`).
2. `GET /content/:id`; require `CREATED`.
3. Convert local time to UTC ISO-8601, respecting the chosen account's own `minLeadTimeMs` for warmed accounts.
4. Include `connectionId` when multiple accounts exist on the platform, or `warmedAccountId` for warmed routing. Never both.
5. `POST /content/:id/schedule`.
6. Verify with `GET /posts/:postId`, including generated caption text if `enableCaptionGeneration` was used.

## Workflow: Cancel And Analyze

Cancel: list `SCHEDULED` posts, match the intended ones, confirm with the user, call `POST /posts/cancel`, verify.
Use `POST /posts/delete` only when the user wants records gone from lists and analytics, and say plainly that published media stays live on the platform.

Analyze: list posted posts or use known ids, call `POST /analytics/posts`, group by platform, content type, and status, treat recent zeros as possibly pre-sync, and recommend the next 7-day plan.

## Strategy Index

Account warmup, for a user's own OAuth account:

- Days 1-5: complete the profile and engage in the niche daily. Do not post.
- Week 1: post once a day. Week 2: twice a day. Treat the account as warmed once videos average 500+ views.
- On TikTok, use inbox posting for roughly the first 5 Fastlane posts.
- Warmed accounts skip all of this. Fastlane provisions and manages them, so they are postable from day one.

Growth defaults: post 1-3 times a day per account per platform after warmup, add accounts rather than overposting one, and judge results over 30 days rather than per post.

Default posting limits:

- TikTok: 10 direct posts/day per workspace and 5 inbox posts/day per connected account, counted on the local calendar day, so this one does reset at local midnight.
- Instagram: 25 posts per rolling 24-hour window per workspace.
- YouTube: 10 videos per channel per **rolling 24-hour window**, not per calendar day, so capacity frees up 24 hours after each counted post rather than at any midnight. That 10 is a default and a channel's budget can be raised to 100. When one channel is connected to several workspaces, the lowest configured budget wins.
- Only the Fastlane-wide ceilings reset on the Pacific calendar day: 1000 YouTube videos/day and 500 TikTok direct posts/day.

Treat these as defaults and read the actual failure back from the API, since a workspace can carry raised or lowered overrides.

## HTTP Examples

```bash
export FASTLANE_BASE="https://api.usefastlane.ai/api/v1"
```

```bash
curl -sS "$FASTLANE_BASE/blitz/preferences" \
  -H "Authorization: Bearer $FASTLANE_API_KEY" \
  -H "User-Agent: usefastlane-ai-agent/1.0" | jq .
```

```bash
curl -sS -X POST "$FASTLANE_BASE/blitz" \
  -H "Authorization: Bearer $FASTLANE_API_KEY" \
  -H "Content-Type: application/json" \
  -H "User-Agent: usefastlane-ai-agent/1.0" \
  -d '{"contentType":"slideshow","aspectRatio":"9:16"}' | jq .
```

```bash
curl -sS -X POST "$FASTLANE_BASE/content" \
  -H "Authorization: Bearer $FASTLANE_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: build-2026-09-01-001" \
  -H "User-Agent: usefastlane-ai-agent/1.0" \
  -d '{
    "contentType": "wall-of-text",
    "contentData": {
      "canvas": { "width": 1080, "height": 1920 },
      "video": { "url": "'"$MEDIA_URL"'", "drawX": 0, "drawY": 0, "drawW": 1080, "drawH": 1920 },
      "audio": { "url": "'"$AUDIO_URL"'" },
      "text": { "content": "hook", "x": 90, "y": 700, "width": 900, "fontSize": 64, "color": "#ffffff", "textAlign": "center" }
    }
  }' | jq .
```

```bash
curl -sS -X POST "$FASTLANE_BASE/content/$CONTENT_ID/schedule" \
  -H "Authorization: Bearer $FASTLANE_API_KEY" \
  -H "Content-Type: application/json" \
  -H "User-Agent: usefastlane-ai-agent/1.0" \
  -d '{"platform":"tiktok","utc_datetime":"2026-09-01T18:00:00Z","caption":"...","connectionId":"..."}' | jq .
```

Plain HTTP and live API discovery are authoritative for cross-agent compatibility.
