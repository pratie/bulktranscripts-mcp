# Summarize a video

**Prompt**

> Summarize this video and pull the five best quotes with timestamps:
> https://www.youtube.com/watch?v=VIDEO_ID

**What the agent does**

1. Fetches the transcript (keeps timestamps for quoting):

```bash
curl -s "https://bulktranscripts.co/api/v1/transcript?video=VIDEO_ID" \
  -H "Authorization: Bearer $BULKTRANSCRIPTS_API_KEY"
```

2. Reads `paragraphs[]` for the summary (silence-grouped — better section
   boundaries than raw captions), and `segments[]` (`{text, start, duration}`)
   to attach a `start` timestamp to each quote.
3. Cites `title`, `channel`, and `url` from the response metadata.

**Tips**

- For summaries without quoting, add `segments=0` — much smaller response.
- `format=ai` returns a ready-made markdown document instead of JSON if the
  user just wants a clean file.
- A video with no captions returns `error.code = "no_transcript"` (HTTP 404)
  and is never charged.
