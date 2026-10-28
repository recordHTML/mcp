# RecordHTML

The `recordhtml` MCP server records web pages and HTML documents to MP4, WebM or GIF.

- Sign in once: the first connection opens a browser sign-in. If the tools are missing, run `/mcp auth recordhtml`.
- `record_html` takes a public `url`, or `html` (one self-contained file), plus `width`, `height` and `durationSec`.
  localhost and private networks are out of reach: pass the HTML instead, with assets loaded from public URLs.
- If `record_html` returns `queued` or `running`, call `get_recording` with the id and `waitSec`. Don't record again.
- `estimate_recording` checks a request against the account's limits without recording.
- Video links expire after 1 hour; `get_recording` returns a fresh one.
