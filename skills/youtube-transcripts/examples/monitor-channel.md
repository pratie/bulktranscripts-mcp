# Monitor a channel for new uploads

**Prompt**

> Every morning, check if @mkbhd posted a new video. If yes, summarize it for
> me.

**What the agent does**

1. Polls the latest-uploads endpoint — **free, no credit ever** (the API key is
   still required):

```bash
curl -s "https://bulktranscripts.co/api/v1/channel/latest?channel=@mkbhd" \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```

2. Compares the returned video ids against the ones already seen (keep them in
   a scratch file, e.g. `~/.cache/yt-monitor/mkbhd.seen`). Use `published` as a
   tie-breaker when present; it can be null when the response says
   `"source": "listing"`.
3. Only for genuinely new videos, fetches the transcript (1 credit) and
   summarizes:

```bash
curl -s "https://bulktranscripts.co/api/v1/transcript?video=NEW_VIDEO_ID&segments=0" \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```

**Why this pattern**

The check itself is free because it rides YouTube's RSS feed — so a daily
monitor costs credits only on days the creator actually posts. Watching three
channels daily for a month typically costs a handful of credits total.

**Scaling up**: the same loop works in cron, n8n, or Make — see
https://bulktranscripts.co/integrations for node-by-node setups.
