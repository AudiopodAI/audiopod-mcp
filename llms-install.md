# Installing the AudioPod MCP server (instructions for an AI agent)

This file tells an agent — Cline, Claude Code, Cursor, or any MCP client's installer —
how to add the AudioPod MCP server unattended. There is nothing to clone, build or run:
the server is hosted, so installation is one config entry plus an API key.

## What you are installing

| | |
|---|---|
| Server name | `audiopod` |
| Endpoint | `https://mcp.audiopod.ai` |
| Transport | Streamable HTTP (MCP `2025-06-18`) |
| Auth | `X-API-Key` request header, value starts with `ap_` (OAuth2 / JWT bearer also accepted) |
| Runtime needed | none — no Node, Python, Docker or GPU |

## Step 1 — get the API key from the user

The key is the only thing you cannot obtain yourself. Ask the user for it, or tell them
to create one at:

    https://www.audiopod.ai/dashboard/account/api-keys

The first key on an account is granted a $1 API starter credit, so the user does not
need to add a card to try the server. A key looks like `ap_` followed by a random
string. Never write the key into a file that is committed to a repository; put it in
the MCP client's config, which is where the client expects secrets to live.

If the user has no key yet, install the config anyway with a placeholder and tell them
the server will return `401 AUTHENTICATION_REQUIRED` until the key is filled in.

## Step 2 — write the config

### Cline, Claude Desktop, Cursor, Continue, Windsurf

Add this object under `mcpServers` in the client's MCP settings JSON (for Cline that is
`cline_mcp_settings.json`). Merge it in — do not overwrite other servers:

```json
{
  "mcpServers": {
    "audiopod": {
      "url": "https://mcp.audiopod.ai",
      "headers": {
        "X-API-Key": "ap_YOUR_KEY"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

### Claude Code

```bash
claude mcp add --transport http audiopod https://mcp.audiopod.ai \
  --header "X-API-Key: ap_YOUR_KEY" --scope user
```

### A client that only speaks stdio

Bridge with `mcp-remote` (needs Node):

```bash
npx -y mcp-remote https://mcp.audiopod.ai --header "X-API-Key:${AUDIOPOD_API_KEY}"
```

Set `AUDIOPOD_API_KEY` in that server entry's `env`. The header argument has no space
after the colon on purpose — `mcp-remote` splits on the first colon and a space breaks
argument parsing on some shells.

## Step 3 — verify the install

Restart or reload the MCP client, then confirm the server connected and lists ten
tools. You can check the endpoint directly without the client:

```bash
curl -s https://mcp.audiopod.ai \
  -H "X-API-Key: ap_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"installer","version":"1"}}}'
```

A healthy response is HTTP 200 with a `result` carrying `serverInfo`. Two failures you
may see instead:

- `401 AUTHENTICATION_REQUIRED` — no key was sent. The header name is `X-API-Key`.
- `401 INVALID_API_KEY` — the key is wrong, revoked, or belongs to a deleted account.

Then call a free tool to prove the round trip end to end. `check_job_status` costs no
credits, so it is the safe smoke test; pass any job id and expect a structured
"not found" rather than an error:

```json
{"jsonrpc":"2.0","id":2,"method":"tools/call",
 "params":{"name":"check_job_status","arguments":{"job_id":"00000000-0000-0000-0000-000000000000"}}}
```

## The ten tools

`text_to_speech`, `clone_voice`, `change_voice`, `generate_music`, `separate_stems`,
`separate_speakers`, `transcribe_audio`, `denoise_audio`, `convert_media`,
`check_job_status`.

Every tool except `check_job_status` starts a job and returns a `job_id`; poll it with
`check_job_status` until the status is terminal, then read the output URL from the
result. Do not busy-poll — a few seconds between polls is enough, and long jobs such as
music generation or stem separation can take minutes.

## Things that commonly go wrong

- **Configured as stdio with a `command`.** There is no local binary. It is a URL.
- **Key in the wrong place.** `X-API-Key` is a request header. An `api_key` query
  parameter also works, but `Authorization: Bearer` does not take an `ap_` key — that
  header is for OAuth2 / JWT, a different auth mode.
- **Expecting audio bytes back from the tool call.** Tools return job ids and, once
  complete, URLs — not inline audio.
- **No credits left.** Paid tools stop accepting work once the account's credits run
  out. Top up from the dashboard at https://www.audiopod.ai/dashboard.

## Where to read more

- MCP setup guide — https://docs.audiopod.ai/sdks/mcp
- API docs — https://docs.audiopod.ai
- Server card — https://audiopod.ai/.well-known/mcp/server-card.json
- Support — support@audiopod.ai
