# RecordHTML MCP server

[![RecordHTML MCP connector – tool definition quality and endpoint health on Glama](https://glama.ai/mcp/connectors/com.recordhtml.mcp/record-html/badges/score.svg)](https://glama.ai/mcp/connectors/com.recordhtml.mcp/record-html)

Connect [RecordHTML](https://recordhtml.com) to an AI assistant and ask it for videos. The hosted
MCP server records web pages and HTML5 creatives (banners, ads, animations, landing pages) to MP4,
WebM or GIF in a real browser. It uses the same recorder, limits and storage as the web app and the
[API](https://recordhtml.com/docs/api).

```
https://mcp.recordhtml.com
```

Streamable HTTP, sign in with OAuth. There is nothing to install or run on your machine.

> This repository holds the server's listing in the
> [MCP Registry](https://registry.modelcontextprotocol.io) (`com.recordhtml/mcp`), the Cursor
> plugin and the Gemini CLI extension. The server itself is hosted and not open source; the files in
> this repository are MIT-licensed. Full documentation: https://recordhtml.com/docs/mcp

## Connect

Add a remote MCP server with the URL above. Your client opens a sign-in page: sign in or create an
account, approve the access and go back to the client. It should then list five tools.

**Claude Code**: then type `/mcp`, choose recordhtml and sign in.

```sh
claude mcp add --transport http recordhtml https://mcp.recordhtml.com
```

**Claude (claude.ai and Claude Desktop)**: Settings → Connectors → Add custom connector, with
`https://mcp.recordhtml.com` as the URL, then Connect.

**Cursor** (`~/.cursor/mcp.json`, or install the plugin in this repository):

```json
{
  "mcpServers": {
    "recordhtml": { "url": "https://mcp.recordhtml.com" }
  }
}
```

**Gemini CLI**: install the extension, start `gemini` and sign in when the browser opens (or run
`/mcp auth recordhtml`).

```sh
gemini extensions install https://github.com/recordHTML/mcp
```

**VS Code** (`.vscode/mcp.json`):

```json
{
  "servers": {
    "recordhtml": { "type": "http", "url": "https://mcp.recordhtml.com" }
  }
}
```

ChatGPT, Codex and other clients: see
[Set up your client](https://recordhtml.com/docs/mcp#clients).

### With an API key

Headless agents, and clients that can set headers, can send an API key from the
[API settings page](https://recordhtml.com/settings/api) instead of signing in. Keys are refused
from browser pages.

```sh
claude mcp add --transport http recordhtml https://mcp.recordhtml.com \
  --header "Authorization: Bearer $RECORDHTML_API_KEY"
```

## Tools

| Tool | What it does |
| --- | --- |
| `record_html` | Record a public URL, or an HTML document the assistant wrote, to a video. Waits for the result and returns a signed video link. |
| `get_recording` | A recording's status, report and a fresh video link; can wait for one that is still running. |
| `estimate_recording` | Check a request without recording or using up quota: what to fix, the estimate, your limits and usage. |
| `list_recordings` | Your recordings, newest first. |
| `delete_recording` | Delete a recording's video files for good. |

Example prompts:

- *Record https://recordhtml.com at 1280×720 for 5 seconds as an MP4.*
- *Write a 300×250 HTML5 banner that fades in the headline "Summer Sale" and a "Shop now" button, then record it as a 6 second GIF.*

## Privacy and support

- Privacy policy: https://recordhtml.com/privacy (what the MCP server receives is covered under
  "AI assistants and the MCP server").
- Terms: https://recordhtml.com/terms
- Support: hello@recordhtml.com or https://recordhtml.com/contact

## What is in this repository

| File | Used by |
| --- | --- |
| `server.json` | The [MCP Registry](https://registry.modelcontextprotocol.io) listing (`com.recordhtml/mcp`) |
| `.cursor-plugin/plugin.json`, `mcp.json`, `assets/` | The Cursor plugin |
| `gemini-extension.json`, `GEMINI.md` | The Gemini CLI extension (`gemini extensions install https://github.com/recordHTML/mcp`) |

## Publishing (maintainers)

### MCP Registry

The server is published under the `com.recordhtml` namespace, proven by a DNS TXT record at the
apex of `recordhtml.com`. You need [`mcp-publisher`](https://github.com/modelcontextprotocol/registry)
and OpenSSL 3 (macOS's built-in LibreSSL has no Ed25519).

One time: create a key and add its TXT record.

```sh
OPENSSL=/opt/homebrew/opt/openssl@3/bin/openssl
$OPENSSL genpkey -algorithm Ed25519 -out key.pem    # keep it secret; *.pem is git-ignored
echo "recordhtml.com. IN TXT \"v=MCPv1; k=ed25519; p=$($OPENSSL pkey -in key.pem -pubout -outform DER | tail -c 32 | base64)\""
```

Add that record at the apex (`recordhtml.com`, not a subdomain), next to the existing TXT
records. When you rotate the key, remove the old record.

Each release:

```sh
mcp-publisher validate
mcp-publisher login dns --domain recordhtml.com \
  --private-key "$($OPENSSL pkey -in key.pem -noout -text | grep -A3 'priv:' | tail -n +2 | tr -d ' :\n')"
mcp-publisher publish
curl -s "https://registry.modelcontextprotocol.io/v0.1/servers?search=com.recordhtml/mcp"
```

`version` follows the server's own version (`MCP_SERVER_VERSION` in html-recorder-web). To
republish with only metadata changes, use a prerelease such as `1.0.0-1`. A version can't be
published twice.

### Gemini CLI extension and Cursor plugin

Keep `version` in `gemini-extension.json` and `.cursor-plugin/plugin.json` in step, and tag each
release (`git tag v1.0.0 && git push --tags`, or a GitHub release with the same tag). The Gemini
gallery indexes this repository through its `gemini-cli-extension` topic. Cursor reviews every
plugin update before it reaches the Marketplace.
