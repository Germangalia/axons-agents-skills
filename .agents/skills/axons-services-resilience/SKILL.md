---
name: axons-services-resilience
description: Message queuing, at-least-once delivery semantics, FIFO ordering, dead letter handling, and resilience patterns for AXONS. Use when implementing reliable message delivery in unreliable network conditions.
---

# AXONS Services: Resilience & Message Queuing

## When to Load

Activate this skill when the task involves:
- Implementing the mandatory message queue
- At-least-once delivery semantics and deduplication
- Offline message buffering and replay on reconnection
- Dead letter queue management
- Exponential backoff retry logic
- Queue persistence and state management

## Key References

- `content/docs/axons-specification/06-services/06-01-message-queue.md`
- `content/docs/axons-specification/06-services/06-02-resilience.md`

## Queue Model

**At-least-once delivery**: Every message delivered at least once. Duplicates possible, never silently dropped.

| Characteristic | Value |
|----------------|-------|
| Guarantee type | At-least-once |
| Deduplication | By messageId |
| Ordering | Strict FIFO |
| Acknowledgment | Per-message |

## Queue States

```
Connected → Disconnected → Syncing → Connected
Connected → Paused → Connected
Disconnected → Paused → Disconnected
```

| State | Behavior |
|-------|----------|
| Connected | Messages dequeued and sent immediately |
| Disconnected | Messages enqueued, delivery suspended |
| Syncing | Queue replay initiated, new messages queued |
| Paused | Messages rejected at enqueue |

## Data Types

### QueueItemType

| Field | Type | Cardinality | Description |
|-------|------|-------------|-------------|
| messageId | UUID v7 | 1 | For deduplication |
| timestamp | dateTime | 1 | When enqueued |
| data | object | 1 | Complete AXONS message |
| retryCount | integer | 1 | Delivery attempts (starts at 0) |
| maxRetries | integer | 1 | Max attempts (unlimited = continuous retry) |
| lastAttempt | dateTime | 0..1 | Last delivery attempt timestamp |

### DeadLetterType

| Field | Type | Description |
|-------|------|-------------|
| messageId | UUID v7 | Original message ID |
| originalTimestamp | dateTime | When originally enqueued |
| failedTimestamp | dateTime | When moved to dead letter |
| retryCount | integer | Attempts made |
| lastError | string | Error reason |
| messageData | object | Complete message |

## Queue Operations

### Enqueue
- Validates capacity before accepting
- Rejects with `QueueFull` if at max capacity
- Persists to non-volatile storage immediately

### Dequeue
- Returns oldest non-expired message (FIFO)
- Messages exceeding maxRetries → moved to dead letter

### Acknowledge
- Removes message permanently
- Duplicate ack handled gracefully

### Retry
- Increments retryCount, updates lastAttempt
- Exceeds maxRetries → moves to dead letter

## Message Persistence

| Requirement | Description |
|-------------|-------------|
| Durability | Persist to non-volatile before returning from enqueue |
| Atomicity | Each enqueue MUST be atomic |
| Integrity | Protection via checksums or similar |
| Capacity | Configurable max queue size |

## Configuration Keys

| Key | Type | Default | Level | Description |
|-----|------|---------|-------|-------------|
| MessageQueueEnabled | boolean | true | MUST | Enable message queue |
| MessageQueuePersistence | string | "always" | MUST | Persistence mode |
| MessageQueueMaxSize | integer | 10000 | MUST | Max queue items |
| MessageQueueRetentionSeconds | integer | -1 | SHOULD | Retention (-1 = infinite) |
| MessageQueueRetryDelaySeconds | integer | 5 | MUST | Delay between retries |
| MessageQueueDeadLetterEnabled | boolean | true | MUST | Enable DLQ |
| MessageQueueDeadLetterRetentionSeconds | integer | 604800 | SHOULD | DLQ retention (7 days) |
| MessageQueueMaxSizePerPeer | integer | 5000 | SHOULD | Max per peer connection |

## Deduplication on Receiving End

- Maintain cache of recently delivered messageIds
- Cache duration = MessageQueueRetentionSeconds
- Duplicates acknowledged without processing
- Operations MUST be idempotent

## Replay Behavior

- On reconnection, queue replays pending messages FIFO
- If message N fails and N+1 succeeds: hold N+1 until N succeeds or dead letters
- If message moves to dead letter → continue with next message
- Offline messages queued, then replayed in order

## Common Pitfalls

- Not persisting queue to non-volatile storage → message loss on crash
- Using unordered delivery instead of strict FIFO → race conditions
- Not checking maxMessageSize → memory exhaustion
- Not implementing deduplication on recipient → duplicate processing
- Setting maxRetries too low in unreliable network → excessive dead letters
- Not handling QueueFull error → silent message loss
- Forgetting to reset delay counter on successful reconnection