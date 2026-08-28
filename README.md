# BulkTranscripts YouTube MCP Server

Give your AI agent YouTube — transcripts, search, channel and playlist listings,
and free new-upload tracking — through one hosted [MCP](https://modelcontextprotocol.io)
server.

**No signup, no OAuth, no API key to start.** Point your client at the URL and it
works. The free tier is 30 transcripts per IP; a one-time credit pack unlocks
more, and credits never expire.

- **Server URL:** `https://bulktranscripts.co/mcp` (streamable HTTP)
- **Docs:** https://bulktranscripts.co/youtube-mcp-server
- **Registry:** [`co.bulktranscripts/youtube`](https://registry.modelcontextprotocol.io/v0/servers?search=bulktranscripts)

## Why a remote server

Most YouTube transcript tools need you to run a local process, keep `yt-dlp`
fresh, and fight IP blocks. This one is hosted: extraction, proxy rotation, and
PO-token handling happen server-side. Your agent just calls a tool.

Transcripts are cached per account — re-reading one you already fetched is free
and instant, so agents that revisit the same video don't burn credits.

## Install

### Claude Code

```bash
claude mcp add --transport http bulktranscripts https://bulktranscripts.co/mcp
```

### Claude on the web or desktop

Add it as a **custom connector** (Settings → Connectors → Add custom
connector) with the URL `https://bulktranscripts.co/mcp`. There is no OAuth
step — the free tier is keyless.

### ChatGPT

Add it as a connector in developer mode, same URL.

### Any client with a JSON config

```json
{
  "mcpServers": {
    "bulktranscripts": {
      "type": "http",
      "url": "https://bulktranscripts.co/mcp"
    }
  }
}
```

### Cursor

Settings → MCP → Add new server → type `http`, URL `https://bulktranscripts.co/mcp`.

### VS Code

```bash
code --add-mcp '{"name":"bulktranscripts","type":"http","url":"https://bulktranscripts.co/mcp"}'
```

### With credits

Pass your license key as a bearer token:

```json
{
  "mcpServers": {
    "bulktranscripts": {
      "type": "http",
      "url": "https://bulktranscripts.co/mcp",
      "headers": { "Authorization": "Bearer YOUR_LICENSE_KEY" }
    }
  }
}
```

Clients that can't send headers can use `https://bulktranscripts.co/mcp?key=YOUR_LICENSE_KEY`.

## Tools

| Tool | What it does | Cost |
| --- | --- | --- |
| `get_transcript` | Full transcript for one video, optionally with timestamped segments. Accepts a URL or an 11-char id. TikTok video URLs work too. | 1 credit (free if already in your library) |
| `get_transcripts` | Up to 20 videos in one call. | 1 credit each |
| `search_youtube` | Search videos, channels, or playlists. | 1 credit |
| `search_channel` | Search inside one channel for a topic. | 1 credit |
| `get_channel_videos` | List up to 1,000 of a channel's videos. | 1 credit |
| `get_playlist_videos` | List up to 1,000 videos in a playlist. | 1 credit |
| `get_latest_videos` | Newest uploads for a channel, via RSS. | **Always free** |

`get_latest_videos` being free is deliberate: poll it to detect new uploads, then
spend a credit only on what's actually new.

## Example prompts

> Summarise the last 10 videos from @veritasium and tell me which ones cover quantum mechanics.

> Find talks about founder mode on the Y Combinator channel and pull the transcripts.

> Watch @simonwillison for new uploads and transcribe anything about local models.

## Pricing

Free: 30 transcripts per IP, no account. Then one-time packs — $4.99 / 200
credits, $14.99 / 1,200, $29 / 3,500. No subscription. Credits don't expire.

See https://bulktranscripts.co/#pricing.

## Also available

- **REST API** — https://bulktranscripts.co/docs
- **Agent skill** (Claude Code, Codex, any `SKILL.md` agent) — https://github.com/pratie/youtube-transcript-skill
- **Web app** for bulk exports to TXT/MD/SRT/VTT/CSV/JSON — https://bulktranscripts.co

## License

MIT — see [LICENSE](LICENSE). This repo holds the manifest and docs for the
hosted server; the extraction service itself is operated at bulktranscripts.co.
