# Why "connecting to host…" got stuck, and how it was fixed

## Symptom

A friend trying to join a room got stuck on "connecting to host…" forever — no
error, just a permanent hang. Confirmed across three different network paths
on her end (home wifi, cellular on her laptop, cellular on her phone), all
failing identically.

## Root cause

`watch-party/index.html` uses [PeerJS](https://peerjs.com) for the
peer-to-peer connection between host and viewers. By default, PeerJS only
configures **STUN** servers for NAT traversal, no **TURN** server.

STUN lets two peers discover their public IP/port and connect directly — but
it only works if at least one side is behind a "well-behaved" NAT. Behind a
**restrictive or symmetric NAT** (common on mobile carrier networks, school/
office wifi, some routers, or CGNAT), STUN can't establish a direct
connection, and there's no automatic fallback — the WebRTC ICE negotiation
just hangs indefinitely with no error surfaced to either side. That matches
the symptom exactly.

Because the failure was identical across three different networks on her
side, and the host's session *was* being found correctly (no "room not
found" error, which fires quickly and separately), the restrictive NAT was
on one end of the connection (host or joiner) with no relay available to
route around it — not a device- or browser-specific issue.

## Fix

Added a TURN relay via [Metered.ca](https://metered.ca)'s free tier (20GB or
500MB/month depending on plan — trivial for this use case, since only tiny
chat/sync messages go through the data channel, not the video itself).

The setup has two layers, both documented at
`metered.ca/docs/turn-rest-api/`:

1. **One-time, server-side (never in the page's JS):** mint a long-lived
   credential using the account **Secret Key**:
   ```bash
   curl -X POST "https://<app-name>.metered.live/api/v1/turn/credential?secretKey=<SECRET_KEY>" \
     -H "Content-Type: application/json" \
     -d '{"label": "watch-party"}'
   ```
   Returns `{ username, password, apiKey }`. The Secret Key itself must
   never be exposed client-side — only this response's `apiKey` is safe for
   that.

2. **Client-side, at runtime (this is what's in the page):** `index.html`
   fetches the actual ICE server list using that `apiKey` before creating
   the `Peer`:
   ```js
   fetch(`https://${METERED_APP_NAME}.metered.live/api/v1/turn/credentials?apiKey=${METERED_API_KEY}`)
   ```
   This returns a mix of STUN and TURN entries (UDP, TCP, and TLS on ports
   80/443, so it also gets through firewalls that block arbitrary UDP), fed
   into `new Peer(..., { config: { iceServers } })`. If the fetch fails for
   any reason, it falls back to Google's public STUN servers only.

A 12-second "still connecting" status message was also added so a stuck
connection is visible in the UI instead of hanging silently forever — useful
for catching this class of problem even if TURN itself weren't the cause
next time.

## If the TURN credential ever needs rotating

Re-run the mint step above with the account Secret Key (Metered dashboard →
**Developers**), then swap the returned `apiKey` into `METERED_API_KEY` near
the bottom of `index.html`'s `<script>` block.

## The actual debugging detour

Most of the time sinks along the way weren't the NAT/TURN theory itself
(that was right from early on) — they were:
- The free "Open Relay Project" static TURN credentials that used to work
  with zero signup (`openrelay.metered.ca` + fixed username/password) are
  dead — Metered retired that hostname, now free TURN requires an account.
- The account's default "Free: 20GB" plan doesn't include API access (only
  visible by comparing the plan feature list closely) — needed the "Free
  Trial Global: 500MB" plan instead, which does include it.
- A single misread character (`I` vs `l`) in the Secret Key, copied from a
  screenshot rather than pasted as text, caused every credential-mint
  attempt to fail with a generic-looking 401 until the key was pasted
  directly into the file instead of transcribed from an image.
