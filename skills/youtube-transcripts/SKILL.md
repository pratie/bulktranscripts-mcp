---
name: youtube-transcripts
description: Fetch YouTube video transcripts, search YouTube, list channel or playlist videos, and track new uploads via the BulkTranscripts API. Use when the user shares a YouTube link, asks to summarize/analyze/quote a video, wants transcripts for a whole channel or playlist, needs YouTube research, or asks what a channel posted recently. Requires a free BulkTranscripts API key in BULKTRANSCRIPTS_API_KEY: the user creates one at https://bulktranscripts.co/app?tab=mcp (Google sign-in, 30 free credits, no card).
license: MIT (skill file); the BulkTranscripts API itself is a hosted service.
last-updated: 2026-09-16
---

# YouTube transcripts via BulkTranscripts

Base URL: `https://bulktranscripts.co`

All endpoints are plain GET returning JSON, and **every one of them needs an API
key** — including the free endpoints. Send it as a Bearer token on every
request:

```bash
curl -s "https://bulktranscripts.co/api/v1/..." \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```

If `BULKTRANSCRIPTS_API_KEY` is not set, ask the user for a key: they sign in
with Google at https://bulktranscripts.co/app?tab=mcp, open the **Connect AI**
tab, and hit **Create key** in the **API keys** card. It is free, includes 30
credits, and needs no card. Keys start with `bt_ak_`, are shown once, and up to
5 can be live per account. A license key from a credit-pack purchase works as a
Bearer token too.

## Costs (check `billing.remaining` in every response)

- Transcript: 1 credit when first added to this account's library — repeat reads are **FREE**
- Search / channel videos / playlist videos: 1 credit each
- `channel/latest` and `account`: always free
- Failures (no captions, video unreachable) are auto-refunded, never charged

When a call returns HTTP 402 `out_of_credits`, tell the user their credits are
used up and link them to https://bulktranscripts.co/#pricing (one-time packs,
never expire). Credits follow the Google account, so topping up keeps the same
API key working.

## Endpoints

### Get one transcript
```bash
curl -s "https://bulktranscripts.co/api/v1/transcript?video=VIDEO_URL_OR_ID" \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```
Params: `video` (watch URL, youtu.be, Shorts, bare 11-char id, or TikTok video
URL) · `language` (default `en`) · `segments=0` to omit timestamps (smaller) ·
`format=txt|md|srt|vtt|csv|ai` to get a rendered file instead of JSON ·
`fresh=1` to force re-extraction.

Response fields: `title`, `channel`, `duration`, `upload_date`, `language`,
`source` (manual_caption|auto_caption), `cached`, `word_count`, `text`,
`paragraphs[]` (silence-grouped — best for reading/chunking), `segments[]`
(`{text, start, duration}`), `billing`.

Long videos produce long text. For summarization, prefer `segments=0` and read
`paragraphs`.

### Search YouTube (1 credit)
```bash
curl -s "https://bulktranscripts.co/api/v1/search?q=QUERY&limit=10" \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```
Add `type=channel` or `type=playlist` to search for channels/playlists instead
of videos.

### Search inside one channel (1 credit)
```bash
curl -s "https://bulktranscripts.co/api/v1/channel/search?channel=@HANDLE&q=TOPIC&limit=10" \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```
Finds a creator's videos about a topic without listing the whole archive.

### List a channel's videos (1 credit, up to 1000)
```bash
curl -s "https://bulktranscripts.co/api/v1/channel/videos?channel=@HANDLE&limit=100" \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```
`channel` accepts @handle, channel URL, or UC… id.

### List a playlist in order (1 credit)
```bash
curl -s "https://bulktranscripts.co/api/v1/playlist/videos?playlist=PLAYLIST_ID_OR_URL" \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```

### Newest uploads — FREE, use for monitoring
```bash
curl -s "https://bulktranscripts.co/api/v1/channel/latest?channel=@HANDLE" \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```
Returns up to 15 recent videos. Poll this freely; only spend credits on videos
that are actually new. Diff on video `id`, not on `published`: when YouTube's
feed is unavailable the response says `"source": "listing"` and `published`
can be null.

### Balance
```bash
curl -s "https://bulktranscripts.co/api/v1/account" \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```

## Playbooks

- **"Summarize this video"** → transcript with `segments=0`, then summarize
  from `paragraphs`; cite the `title` and `url`.
- **Whole channel/playlist** → list videos first, show the user the count
  (each new library transcript = 1 credit), then fetch transcripts one by one,
  skipping failures (they are reported per video and refunded).
- **"What did X post this week?"** → `channel/latest` (free), compare video
  ids against what you have seen (use `published` when present), fetch
  transcripts only for the relevant new videos.
- **Deep research on a creator** → `channel/search` for the topic (or list all
  videos), pick candidates by title, fetch only those transcripts.

Errors come as `{"error": {"code", "message"}}`. The codes you will actually
hit, and what to do about each:

- `no_transcript` (404) — the video has no captions. Not charged. Skip it and
  carry on; in a batch this is normal, not a failure.
- `resolution_failed` (400) — a well-formed id or URL that could not be
  resolved (nonexistent, private, removed, or region-blocked). Handle it
  alongside `no_transcript`: skip the video and carry on.
- `playlist_private` (400) — YouTube says the playlist does not exist, which is
  also what it says about a *private* playlist. Playlist ids come in several
  lengths (13, 18, 26, 34) and all work, so do not second-guess the id: tell
  the user to open the playlist on YouTube → Edit → Visibility → Unlisted,
  then retry. Not charged.
- `missing_api_key` (401) — no key was sent. Ask the user to create a key at
  https://bulktranscripts.co/app?tab=mcp and provide it, then set
  `BULKTRANSCRIPTS_API_KEY` and retry.
- `invalid_api_key` (401) — the key sent is not valid. Ask the user to check it
  or create a new one at https://bulktranscripts.co/app?tab=mcp.
- `invalid_input` (400) — the `video` / `channel` / `playlist` parameter is
  missing, not a YouTube/TikTok reference, or otherwise malformed. A caller
  bug, not a video problem.
- `out_of_credits` (402) — balance exhausted. Tell the user and link
  https://bulktranscripts.co/#pricing.
- `rate_limited` (429) — respect the `Retry-After` header before retrying.
  Limits are per public IP: 120 requests/min overall, 30/min under `/api/v1/`
  (cache hits and `/account` count too), so pace bulk fetches at under 30/min.

Full reference: https://bulktranscripts.co/docs
