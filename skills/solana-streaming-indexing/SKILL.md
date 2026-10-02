---
name: solana-streaming-indexing
description: Design or review Solana data ingestion, streaming, indexing, historical backfill, analytics, bots, and dashboard pipelines. Use when choosing JSON-RPC, WebSockets, Geyser, Fumarole, Jetstreamer, Old Faithful, ClickHouse, or an indexer schema.
argument-hint: <data need, program, or indexer path>
---

# Solana streaming and indexing

Load this resource first:

- `resources/knowledge/production/streaming-indexing-backfill.md`

## Procedure

1. Classify the data need: small read, UI notification, production live stream, historical backfill, analytics, or bot.
2. Choose the least complex data source that meets the reliability requirement.
3. State the delivery guarantee and failure mode.
4. Design dedupe keys before writing ingestion code.
5. Define cursor, checkpoint, replay, finality, and reorg behavior.
6. Review schema for ordering assumptions.
7. Add monitoring for stalled cursors, stale subscribers, RPC errors, and sink write failures.

## Output

Return:

1. Recommended source: JSON-RPC, WebSocket, Geyser/Fumarole, Jetstreamer, or other.
2. Why that source fits.
3. Dedupe keys.
4. Checkpoint strategy.
5. Schema notes.
6. Failure and replay tests to add.
7. Minimal implementation plan.
