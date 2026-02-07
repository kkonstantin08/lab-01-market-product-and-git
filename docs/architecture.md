## Product Choice

- **Telegram**
- [Link](https://web.telegram.org/)
- Telegram is a cloud-based instant messaging app focused on speed, security, and cross-platform availability. It offers end-to-end encrypted chats, large group support, and a powerful API for bots and custom integrations.

![Telegram Component Diagram](../../../docs/diagrams/out/telegram/component-diagram/Component%20Diagram.svg)

[Telegram Component Diagram](https://github.com/inno-se-toolkit/lab-01-market-product-and-git/blob/main/docs/diagrams/src/telegram/component-diagram.puml)

- **MTProto Gateway**: First server users connect to; checks encryption and sends requests to the right internal service.
- **Auth & Session Service**: Handles logins (like SMS codes), keeps track of active sessions, and logs out old devices if needed.
- **Message Handling Service**: Receives messages, makes sure they’re delivered once (no duplicates), and sends them to the right chat using Kafka.
- **Media & File Service**: Stores and serves files (photos, videos); files are encrypted and can be resized or converted when needed.
- **Secret Chat Relay**: Moves secret chat messages between users without reading them—keeps them end-to-end encrypted.

## Data Flow

![Telegram Sequence Diagram](../../../docs/diagrams/out/telegram/component-diagram/Sequence%20Diagram.svg)

[Telegram Sequence Diagram](https://github.com/kkonstantin08/lab-01-market-product-and-git/blob/main/docs/diagrams/src/telegram/sequence-diagram.puml)

- **What happens**:  
  In the _Send Message (with File Ref)_ flow, Alice’s **Mobile App** sends a message that includes a reference to a previously uploaded file (`file_id_A`). The **MTProto Gateway** checks her session with the **Auth Service**, then passes the request to the **Message Service**, which saves the message to the **Shared DB** and notifies other services via the **Kafka Event Bus**. Finally, the **MTProto Gateway** sends a confirmation back to the **Mobile App**.

- **Component interactions**:
  - **Mobile App** → **MTProto Gateway**: `sendMessage(peer=Bob, file_id=file_id_A)`
  - **MTProto Gateway** → **Auth Service**: validates session token
  - **MTProto Gateway** → **Message Service**: forwards message data
  - **Message Service** → **Shared DB**: stores `{sender, receiver, file_id_A, timestamp}`
  - **Message Service** → **Kafka Event Bus**: publishes `NewMessageUpdate` event
  - **MTProto Gateway** → **Mobile App**: returns assigned `msg_id`

## Deployment

![Telegram Deployment Diagram](../../../docs/diagrams/out/telegram/component-diagram/Deployment%20Diagram.svg)

[Telegram Deployment Diagram](https://github.com/kkonstantin08/lab-01-market-product-and-git/blob/main/docs/diagrams/src/telegram/deployment-diagram.puml)

- **User devices**: Phones, computers, or browsers where Telegram apps run.
- **Edge servers**: First servers users connect to (handle login and messages).
- **Main servers**: Run core features like sending messages, storing files, and user auth.
- **Databases & cache**: Store chats (DB) and fast temporary data like sessions (Redis).
- **File storage**: Holds photos, videos, and documents (like a cloud disk).
- **Message queue (Kafka)**: Sends updates between services (e.g., “new message” alerts).
- **External services**: SMS for login codes and push notifications (Apple/Google).

### Assumptions

- I assume the **Message Handling Service** performs message deduplication using client-side message IDs and sequence numbers, because Telegram’s MTProto protocol documentation mentions idempotency but doesn’t specify server-side implementation details.
- I assume the **Media & File Service** stores files in encrypted form at rest, because Telegram claims end-to-end encryption for secret chats and cloud chats are described as “encrypted,” though the exact storage-layer encryption scheme isn’t publicly documented.

### Open Questions

- Telegram: How does the **Secret Chat Relay** handle offline recipients—does it store encrypted payloads temporarily, and if so, for how long and with what eviction policy?
- Telegram: What consistency model does the **Shared DB** use for message history across data centers—strong, eventual, or causal—and how are conflicts resolved during network partitions?
