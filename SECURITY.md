# Security Policy

## Reporting a Vulnerability

Please report security vulnerabilities **privately** via one of:

- GitHub Security Advisory: [Report a vulnerability](https://github.com/perplexityai/modelcontextprotocol/security/advisories/new)
- Email: `security@perplexity.ai`

Please do not open a public issue, draft PR, or discussion for security reports.

## Supported Versions

Only the latest minor of `@perplexity-ai/mcp-server` on npm is supported with security fixes. Operators should pin to the latest patch within that minor.

## Security model of the HTTP transport

The HTTP transport in `src/http.ts` exposes the MCP server over `POST /mcp`. The server authenticates **outbound** calls to `api.perplexity.ai` using the `PERPLEXITY_API_KEY` env var. It does **not** authenticate **inbound** callers. Any process or page that can reach `/mcp` can therefore consume the operator's API quota and read tool responses.

For this reason the defaults are loopback-only and deny-all:

| Setting | Default | Why |
|---|---|---|
| `BIND_ADDRESS` | `127.0.0.1` | Loopback only — not reachable from the LAN or the internet. |
| `ALLOWED_ORIGINS` | *(empty)* | Reject all cross-origin browser requests by default. |
| `ALLOWED_HOSTS` | loopback only | Reject requests whose `Host` header doesn't match a known loopback name. |

If you need to expose the server beyond loopback you should configure an explicit `ALLOWED_ORIGINS` allowlist and an explicit `ALLOWED_HOSTS` allowlist, and ideally front the server with a reverse proxy that enforces authentication.

## Configuration notes

- **`ALLOWED_ORIGINS=*`** — the `cors` middleware will reflect the requesting `Origin` header back into `Access-Control-Allow-Origin` rather than emitting a literal `*`. The server emits a startup warning when this is set.
- **`BIND_ADDRESS=0.0.0.0`** — exposes the server on every network interface. The server emits a startup warning when this is set. The `start:http:UNSAFE-public` / `dev:http:UNSAFE-public` npm scripts are intentionally named to make the configuration visible.
- **Sandboxed / `file://` callers** — these send `Origin: null`. The CORS handler rejects `null` unless `"null"` is explicitly present in `ALLOWED_ORIGINS`.
