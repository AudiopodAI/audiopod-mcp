# AudioPod MCP Server

Hosted [Model Context Protocol](https://modelcontextprotocol.io) server for **AudioPod's audio AI**. Give any MCP-capable agent — Claude Desktop, Claude Code, Cursor, Continue, Cline, Goose, Codex — ten audio tools over a single endpoint. No install, no local models, no GPU.

| | |
|---|---|
| **Endpoint** | `https://mcp.audiopod.ai` |
| **Transport** | Streamable-HTTP (MCP `2025-06-18`) |
| **Auth** | `X-API-Key: ap_*` (or OAuth2 / JWT) — [get a free key](https://audiopod.ai/dashboard/account/api-keys) |
| **Docs** | https://docs.audiopod.ai/sdks/mcp |
| **Discovery** | [`/.well-known/mcp/server-card.json`](https://audiopod.ai/.well-known/mcp/server-card.json) |

**Registry name:** `ai.audiopod/audiopod` (official [MCP Registry](https://registry.modelcontextprotocol.io)).

**Installing with an agent?** [`llms-install.md`](./llms-install.md) is the unattended-install guide — config for each client, how to verify, and the failures to expect.

**Using a coding agent?** [`AudiopodAI/audiopod-plugins`](https://github.com/AudiopodAI/audiopod-plugins) wraps this server plus fourteen task skills as a one-command install for the major agent CLIs and editors.

## Tools

| Tool | Does |
|---|---|
| `text_to_speech` | Speech in 200+ languages, 500+ voices and custom clones |
| `clone_voice` | Clone a voice from a 5–30s reference clip |
| `change_voice` | Convert a recording to a different target voice |
| `generate_music` | Songs, instrumentals, rap, or vocal stems from a text prompt |
| `separate_stems` | Split a track into stems (vocals, drums, bass, …); two-stem mode = karaoke |
| `separate_speakers` | Isolate each speaker into a separate track |
| `transcribe_audio` | Transcribe with word-level timestamps and speaker diarization |
| `denoise_audio` | Remove background noise while preserving voice character |
| `convert_media` | Convert audio/video formats (mp3/wav/flac/ogg/m4a, mp4/mov) |
| `check_job_status` | Poll the status/result of a long-running job (free) |

Long-running tools return a `job_id`; poll with `check_job_status`.

## Add it

**Claude Code**
```bash
claude mcp add --transport http audiopod https://mcp.audiopod.ai \
  --header "X-API-Key: ap_YOUR_KEY" --scope user
```

**Claude Desktop / Cursor / Continue / Cline** — add to your `mcpServers` config:
```json
{
  "mcpServers": {
    "audiopod": {
      "url": "https://mcp.audiopod.ai",
      "headers": { "X-API-Key": "ap_YOUR_KEY" }
    }
  }
}
```

## Try it (raw JSON-RPC)

```bash
# List tools
curl -s https://mcp.audiopod.ai \
  -H "X-API-Key: ap_YOUR_KEY" -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'

# Text to speech
curl -s https://mcp.audiopod.ai \
  -H "X-API-Key: ap_YOUR_KEY" -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call",
       "params":{"name":"text_to_speech","arguments":{"text":"Hello from my agent."}}}'
```

## Building a startup on this?

Apply to **[AudioPod for Startups](https://audiopod.ai/startups)** — free Pro for 3 months plus developer API credits for eligible early-stage teams.

## Links

- Website — https://audiopod.ai
- API + SDK docs — https://docs.audiopod.ai
- MCP setup guide — https://docs.audiopod.ai/sdks/mcp

Two manifests live here and they are not interchangeable: `server.json` is the
official registry entry (registry schema), `server-card.json` is the MCP server
card — a reference copy, since the live card is served at the Discovery URL above.

## License

MIT — see [LICENSE](./LICENSE). (Covers this listing/README; the AudioPod service itself is governed by the [AudioPod Terms](https://audiopod.ai/terms).)
