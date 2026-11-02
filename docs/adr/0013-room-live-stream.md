# ADR 0013 — Renewable browser room streams

## Decision

Keep HTTP mutations and the MCP bounded wait contract. Add opt-in `live=true` to the existing authenticated browser SSE endpoint; default legacy batches remain unchanged. Live connections send a complete `RoomResult` immediately, then whenever the durable room sequence changes, including after close when a reviewed summary arrives. Equal/older snapshots do not overwrite newer local state. Complete transcripts recover gaps without client-side event reducers or changes to mutation receipts.

Use a streaming fetch reader with the existing generated API client and cookie authentication, not URL credentials. Send heartbeat comments every five seconds and normally renew after 25 seconds, within the existing 30-second server write / 35-second proxy timeouts. Every outgoing snapshot/heartbeat revalidates the app session and workspace. Provider availability is checked during observation. Disconnects cancel work; slow clients remain bounded by the server write deadline. Normal renewal reauthorizes through the full request boundary.

Share the existing two concurrent waits per workspace / 128 per process between browser observers and MCP waits. Read durable room state once a second per connection (not database pub/sub); this preview does not claim instantaneous cross-process notifications or high fanout scalability. Query-rate limits apply at connection entry; observations do not repeatedly invoke the request limiter. Successful snapshot reads record ordinary `rooms.get` operations without message bodies. No extra charge, queue, migration or model execution is introduced.

The frontend handles snapshots as they arrive, automatically renews healthy connections, and makes at most six retries after consecutive failures using exponential delay/jitter; 429 waits at least 60 seconds. Authentication/permission/not-found/invalid-protocol errors stop immediately. A healthy full renewal resets the retry budget, not merely receipt of one frame. Online/foreground transitions resume non-terminal connections; a stopped connection needs explicit retry. Cancel pending reads/delays when leaving a room. Display connecting/connected/reconnecting/stopped in both supported languages. Scroll only if the reader is near the transcript bottom; otherwise offer a new-message button.

## Consequences and validation

No WebSocket or MCP subscription/wake-up claim. Full snapshots are simple and safe for the current bounded transcript, but long rooms and large audiences will need incremental subscriptions and shared event fanout. Closed room observation lasts while its page is open, using the same caps and renewals. Dashboard studio live updates are outside this change.

Tests cover UTF-8/chunk-split SSE delivery before EOF, reader cancellation, retry boundaries, rate limits, late closed-room summaries, snapshot gap recovery, shared concurrency caps, cancellation and heartbeat/renewal. A real HTTP integration test verifies flushing, a pushed close event and session revocation. Live deployment checks must verify heartbeat/renewal through the public reverse proxy; retain the previous API/app images for rollback.
