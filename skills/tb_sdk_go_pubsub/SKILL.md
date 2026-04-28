---
name: tb_sdk_go_pubsub
description: Go PubSub consumers, pubsub/node producers, WebSocket URL, wiring vs messaging YAML.
---

## Consumer (`trigger.type: pubsub`)

- `Data()` → `([]byte, error)`; `Channel()` → `(*ChannelObject, error)`.

## Producer (HTTP) — `pubsub/node`

- `import pubsubnode "github.com/taubyte/go-sdk/pubsub/node"`
- `ch, err := pubsubnode.Channel("<channelName>")` — string must equal messaging `channel.match` and PubSub function `trigger.channel`.
- `ch.Publish(payload)`; for browsers: `ch.WebSocket().Url()` with `bridges.websocket.enable: true` in messaging YAML.

**Checklist:** `messaging.channel.match` == `trigger.channel` == `pubsubnode.Channel("...")`.

Rules: `tb_sys_core_constraints`.