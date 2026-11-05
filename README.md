# PlateQuery MCP — US License Plate & Vehicle Information

**License plate lookup for AI agents, built on [PlateQuery](https://platequery.com/)**
— the license plate lookup site people use to notify vehicle owners,
report bad driving, or flag illegal parking. This [Model Context
Protocol](https://modelcontextprotocol.io) server lets an agent read
those same messages for any U.S. license plate, plus public ALPR
(automated license plate reader) sighting history from the Flock Safety
camera network, by state or region.

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
| `post_plate_message` | Free | Post a new message about a plate -- e.g. "reported for speeding" -- optionally with vehicle make/model/year/color and an approximate location. Requires the `write` scope. See [Posting messages](#posting-messages) below. |

A plate marked private by its owner returns zeroed messages/mentions to
everyone except that owner. Free plan grants a starting credit balance on
signup; additional credits are purchased in bulk packages — see
[api-settings](https://platequery.com/api-settings/) for current pricing.
Paid tools are rate-limited per account; call `check_credits` as often as
you like to check balance without spending a credit.

### Posting messages

`post_plate_message` is free but deliberately conservative, since it's
the only tool that writes anything:

- Requires a PlateQuery account with a **confirmed email**.
- Rate-limited: 2 posts/minute, 10 posts/day, 3 posts/day about the same
  plate.
- **Not published immediately.** Every post goes into the same
  moderation queue a website submission does -- it's invisible (and the
  plate's followers aren't notified) until a human moderator approves it.
- `vehicle_make`/`vehicle_model`/`vehicle_year`/`vehicle_color` and
  `location` are all optional free text -- `location` should be an
  approximate description (e.g. "Hwy 37 near Los Altos, CA" or a
  cross-street), never precise GPS coordinates.

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
5. Approving access on the consent screen, if one appears. It shows the
   app's self-chosen name and where its callback goes, labeled as
   unverified. It's skipped only for known hosted MCP clients (Claude,
   ChatGPT) and loopback callbacks.

No client secret or pre-issued API key is needed — sign-in happens in
the browser the first time an agent connects.

Authorization Code + PKCE and its refresh token are the only grants
accepted. Registered `redirect_uris` must use https, or http on a loopback
address, and registration is rate limited per IP.

Two scopes exist: `read` (the default -- covers every tool except
`post_plate_message`) and `write` (required for `post_plate_message`,
opt-in only). A client must explicitly request `write` at connect time to
be able to post; an existing `read`-only connection can't post without
reconnecting.

## Adding it to an MCP client

Most MCP clients (including Claude) let you add a remote server by URL.
Add `https://api.platequery.com/mcp` as a Streamable HTTP connector and
follow the sign-in prompt on first use.

## Support

Questions or issues: https://platequery.com/contact/
