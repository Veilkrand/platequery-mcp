# PlateQuery MCP — US License Plate & Vehicle Information

**License plate lookup for AI agents.** PlateQuery exposes a remote [Model
Context Protocol](https://modelcontextprotocol.io) server that lets an
agent check whether a license plate has messages posted about it, or
public ALPR (automated license plate reader) sighting history from the
Flock Safety camera network — by U.S. state or region.

- **Website:** https://platequery.com/
- **API docs / pricing:** https://api.platequery.com/
- **MCP endpoint:** `https://api.platequery.com/mcp` (Streamable HTTP)
- **Registry listing:** `com.platequery/plate-api` on the
  [official MCP registry](https://registry.modelcontextprotocol.io/)

This repository holds public documentation for the server. The
implementation lives in PlateQuery's main application repository, which
is closed-source.

## Tools

| Tool | Cost | Description |
| --- | --- | --- |
| `check_credits` | Free | Check your account's remaining credit balance. Doesn't count against the rate limit. |
| `lookup_plate` | 1 credit | Quick summary for one plate: privacy status, the latest 3 messages (plus a total count), and a count of ALPR mentions, if any are public. |
| `list_plate_messages` | 1 credit | Full, paginated list of messages posted about a plate. |
| `list_plate_alpr_mentions` | 1 credit | Full, paginated ALPR search history for a plate, including the searching agency and source. |

A plate marked private by its owner returns zeroed messages/mentions to
everyone except that owner. Free plan grants a starting credit balance on
signup; additional credits are purchased in bulk packages — see
[api-settings](https://platequery.com/api-settings/) for current pricing.
Paid tools are rate-limited per account; call `check_credits` as often as
you like to check balance without spending a credit.

## Authentication

The server is OAuth 2.1 only — no manual API keys. It supports PKCE and
dynamic client registration (DCR), so a compliant MCP client (Claude,
or any client following the
[MCP authorization spec](https://modelcontextprotocol.io/specification/basic/authorization))
can connect by:

1. Pointing at `https://api.platequery.com/mcp`.
2. Following the `WWW-Authenticate` challenge to discover
   `https://platequery.com` as the authorization server (RFC 9728
   protected resource metadata).
3. Registering itself dynamically (RFC 7591) and running the standard
   Authorization Code + PKCE flow.
4. Signing in with (or creating) a PlateQuery account when prompted.

No client secret or pre-issued API key is needed — sign-in happens in
the browser the first time an agent connects.

## Adding it to an MCP client

Most MCP clients (including Claude) let you add a remote server by URL.
Add `https://api.platequery.com/mcp` as a Streamable HTTP connector and
follow the sign-in prompt on first use.

## Support

Questions or issues: https://platequery.com/contact/
