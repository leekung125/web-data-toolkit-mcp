# Web Data Toolkit — MCP server

Four public-data tools for any MCP client: **YouTube transcripts** (one video or a whole channel),
**Google Trends** (interest over time, by region, related queries), and **Google Play reviews**.

Hosted and remote — there is nothing to install, build, or keep running. Point your client at one URL.

```json
{
  "mcpServers": {
    "web-data-toolkit": {
      "type": "http",
      "url": "https://web-data-toolkit.vercel.app/mcp",
      "headers": { "x-api-key": "YOUR_KEY" }
    }
  }
}
```

Clients that accept only a URL can carry the key in the query string instead:
`https://web-data-toolkit.vercel.app/mcp?key=YOUR_KEY`

**Try it before signing up.** Put `wdt_demo_public` in as the key. It is a shared demo key: rate limited,
and capped at 10 rows per call, but every tool answers and you can see the exact shape of the data before
you decide anything.

## Tools

| Tool | What it returns | Required |
|---|---|---|
| `youtube_transcript` | Transcript text for up to 50 videos, with language, an auto-generated flag, word count, title and channel | `url` |
| `youtube_channel_transcripts` | Every recent video of a channel, `@handle` or playlist, each with metadata and transcript | `source` |
| `google_trends` | Interest over time, interest by region, and top plus rising related queries for up to 5 keywords | `keyword` |
| `google_play_reviews` | Rating, text, author, thumbs-up, app version and the developer's reply | `app` |

Optional arguments are described in each tool's input schema, so your client will show them. Full REST
documentation, an OpenAPI specification and an `llms.txt` live at
**<https://web-data-toolkit.vercel.app>**.

## Getting a key

Subscribe on RapidAPI: <https://rapidapi.com/leetanakung98/api/web-data-toolkit>. There is a free tier.

## How it behaves

- **Transport:** JSON-RPC 2.0 over Streamable HTTP, protocol `2025-06-18` (it will also negotiate
  `2025-03-26` and `2024-11-05`). The server is stateless: it issues no session id, and `GET` returns 405
  because there is no server-initiated stream.
- **Latency:** every call is a live fetch from the source, not a cache. Measured 2026-09-14, one call each: a
  YouTube transcript in 3.7s, Google Play reviews in 3.8s, a Google Trends keyword in 8.6s; the slowest single
  call that day was 21s. **Set your client timeout to at least 60 seconds.**
- **Large results:** a tool result is truncated at 120,000 characters and says so in its first line, so a whole
  channel cannot silently blow up a context window. Narrow the request for the rest.
- **Errors:** a failed call comes back as an MCP tool result with `isError` set and the reason in the text,
  rather than as a protocol error, so an agent can read and recover from it.

## Honest limits

- Public data only. Nothing behind a login, and no personal data beyond the public author name a platform
  already displays on a review.
- Transcripts are whatever the platform publishes. Auto-generated captions carry the errors the machine made,
  and the response tells you which kind you received.
- Google Trends values are relative interest as Google reports them. They are **not** search volumes.
- There is no published uptime figure, because there is not yet enough history to state one honestly.
- An App Store reviews tool was offered until 2026-09-14 and was withdrawn: Apple's public customer-reviews
  feed stopped returning entries for every app and country, and shipping a tool that returns nothing is worse
  than shipping none. It returns only if a reliable public source does.

## License

MIT — see [LICENSE](LICENSE). The license covers this repository; the hosted service is a separate offering
with its own terms on RapidAPI.
