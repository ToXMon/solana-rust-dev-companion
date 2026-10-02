# Solana streaming, indexing, and backfill

Use this module when a project needs data pipelines, event indexing, analytics, bots, dashboards, or reliable off-chain state.

Primary sources:

- Fumarole blog: https://blog.triton.one/introducing-yellowstone-fumarole/
- Yellowstone Fumarole repo: https://github.com/rpcpool/yellowstone-fumarole
- Fumarole client docs: https://docs.rs/yellowstone-fumarole-client/latest/yellowstone_fumarole_client/
- Jetstreamer repo: https://github.com/anza-xyz/jetstreamer
- Jetstreamer docs: https://docs.rs/jetstreamer
- Anza Geyser docs: https://docs.anza.xyz/validator/geyser/
- Solana WebSocket RPC docs: https://solana.com/docs/rpc/websocket

## Choice rule

| Need | Start with | Why | Watch out |
| --- | --- | --- | --- |
| Small reads or repair jobs | JSON-RPC | Simple and universal | Rate limits and expensive account scans |
| UI notifications | RPC WebSockets | Easy browser-compatible subscriptions | Not a durable cursor |
| Production live indexing | Geyser or Fumarole | High-throughput account and transaction streams | Requires idempotent sinks |
| Reliable near-live stream | Fumarole | Persistent subscribers and replay after disconnects | At-least-once delivery and retention windows |
| Historical replay/backfill | Jetstreamer | High-throughput Old Faithful replay into plugins | No account updates in Old Faithful data |

## Fumarole mental model

Fumarole is a persistent stream, not just a socket.
It tracks a subscriber position over an internal log so clients can reconnect after failures.
It is designed for reliable near-live Geyser event streaming.

Design requirements:

- Treat delivery as at-least-once.
- Make every sink idempotent.
- Store a durable cursor or persistent subscriber name.
- Monitor staleness and retention windows.
- Expect block-oriented behavior where block data arrives before slot status.
- Prefer official SDKs over depending on internal protobufs.

Dedupe keys:

- Transactions: `(slot, signature)` or `signature` with an explicit finality policy.
- Account updates: `(pubkey, slot, write_version)`.
- Slot status: `(slot, status)`.
- Block metadata: `slot` plus blockhash or parent when needed.

## Jetstreamer mental model

Jetstreamer is for historical replay, research, analytics, and large backfills.
It streams Old Faithful historical ledger data into Jetstreamer plugins, Geyser-style plugins, ClickHouse, or custom sinks.

Design requirements:

- Select an epoch or slot range deliberately.
- Expect high throughput to expose sink bottlenecks.
- Do not assume global arrival order when parallel workers are enabled.
- Use explicit ordering fields: slot, transaction index, signature, entry index, and reward identity.
- Build databases with dedupe and retry semantics.
- Do not use Jetstreamer when the task requires historical account updates, because Old Faithful does not provide them.

## Indexer correctness checklist

- [ ] Delivery guarantee is written down.
- [ ] Dedupe keys exist for every table or sink.
- [ ] Checkpoints survive process restart.
- [ ] Replay from checkpoint is safe.
- [ ] Duplicate events do not double-count balances, volume, rewards, or state transitions.
- [ ] Ordering assumptions are explicit.
- [ ] Finality or reorg policy is explicit.
- [ ] Backfill and live ingestion converge to the same schema.
- [ ] Re-ingesting the same range is safe.
- [ ] Monitoring catches stalled cursors, stale subscribers, and sink write failures.
