# 16Arena Webhook Events & Payloads

This document lists all webhook events and the **exact payload** sent to your endpoint when using the **default** payload format. Use it to integrate with 16Arena webhooks.

---

## Delivery details

- **Method:** `POST`
- **Content-Type:** `application/json`
- **Body:** JSON (camelCase). Null fields are omitted.
- **Signature (optional):** If you set a **Secret Key** on the webhook, each request includes header:
  - `X-Webhook-Signature: sha256=<hex>` (HMAC-SHA256 of the raw JSON body).

---

## Default payload envelope (all events)

Every webhook uses this top-level structure. Only `data` varies by event.

```json
{
  "event": "<event-type>",
  "eventId": "<uuid>",
  "timestamp": "<ISO 8601>",
  "tenantId": "<uuid>",
  "data": { ... }
}
```

| Field       | Type   | Description                          |
|------------|--------|--------------------------------------|
| `event`    | string | Event type (e.g. `team.member.joined`) |
| `eventId`  | string | Unique id for this delivery (idempotency) |
| `timestamp`| string | When the event occurred (ISO 8601)   |
| `tenantId` | string | Your tenant ID                       |
| `data`     | object | Event-specific data (see per-event)   |

---

## Event list and payloads

### 1. Team events

#### `team.member.joined`

**When:** A user is added to a team.

**Payload `data`:**

| Field      | Type   | Description        |
|-----------|--------|--------------------|
| `teamId`  | string | UUID of the team   |
| `teamName`| string | Team display name  |
| `userId`  | string | UUID of the user   |
| `userName`| string | User display name  |

**Example `data`:**
```json
{
  "teamId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "teamName": "Alpha Squad",
  "userId": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "userName": "PlayerOne"
}
```

---

#### `team.member.removed`

**When:** A user is removed from a team.

**Payload `data`:** Same as `team.member.joined`.

| Field      | Type   | Description        |
|-----------|--------|--------------------|
| `teamId`  | string | UUID of the team   |
| `teamName`| string | Team display name  |
| `userId`  | string | UUID of the user   |
| `userName`| string | User display name  |

---

### 2. Tournament registration events

#### `tournament.player.joined`

**When:** A **player** (individual) is approved/joined to a tournament.

**Payload `data`:**

| Field               | Type   | Description                    |
|--------------------|--------|--------------------------------|
| `tournamentId`     | string | UUID of the tournament        |
| `participantId`    | string | UUID of the player            |
| `participantType`  | string | `"Player"`                    |
| `registrationId`   | string | UUID of the registration      |
| `registrationStatus` | string | e.g. `"Approved"`         |

**Example `data`:**
```json
{
  "tournamentId": "t1t2t3t4-t5t6-7890-tour-nament1234567",
  "participantId": "p1p2p3p4-p5p6-7890-play-er1234567890",
  "participantType": "Player",
  "registrationId": "r1r2r3r4-r5r6-7890-reg-istration12345",
  "registrationStatus": "Approved"
}
```

---

#### `tournament.team.joined`

**When:** A **team** is approved/joined to a tournament.

**Payload `data`:** Same structure as `tournament.player.joined`; `participantType` is `"Team"`.

| Field               | Type   | Description                    |
|--------------------|--------|--------------------------------|
| `tournamentId`     | string | UUID of the tournament        |
| `participantId`    | string | UUID of the team              |
| `participantType`  | string | `"Team"`                      |
| `registrationId`   | string | UUID of the registration      |
| `registrationStatus` | string | e.g. `"Approved"`         |

---

### 3. Tournament assignment events

Assignment and removal use **context-specific** event names: `participant.assigned.round` / `.group` / `.match` and `participant.removed.round` / `.group` / `.match`.

#### `participant.assigned.round`

**When:** Participants are assigned to a round (e.g. bracket slots).

**Payload `data`:**

| Field             | Type   | Description                          |
|------------------|--------|--------------------------------------|
| `tournamentId`   | string | UUID of the tournament               |
| `roundId`        | string | UUID of the round                    |
| `roundName`      | string | Round display name                   |
| `participantIds` | array  | UUIDs of assigned participants       |
| `participantType`| string | `"User"` or `"Team"`                 |

**Example `data`:**
```json
{
  "tournamentId": "22222222-2222-2222-2222-222222222222",
  "roundId": "55555555-5555-5555-5555-555555555555",
  "roundName": "Round 1",
  "participantIds": [
    "66666666-6666-6666-6666-666666666666",
    "77777777-7777-7777-7777-777777777777"
  ],
  "participantType": "Team"
}
```

---

#### `participant.assigned.group`

**When:** Participants are assigned to a group (e.g. within a round).

**Payload `data`:**

| Field             | Type   | Description                          |
|------------------|--------|--------------------------------------|
| `tournamentId`   | string | UUID of the tournament               |
| `roundId`        | string | UUID of the parent round             |
| `roundName`      | string | Round display name                   |
| `groupId`        | string | UUID of the group                    |
| `groupName`      | string | Group display name                   |
| `participantIds` | array  | UUIDs of assigned participants       |
| `participantType`| string | `"User"` or `"Team"`                 |

**Example `data`:**
```json
{
  "tournamentId": "22222222-2222-2222-2222-222222222222",
  "groupId": "88888888-8888-8888-8888-888888888888",
  "groupName": "Group A",
  "roundId": "55555555-5555-5555-5555-555555555555",
  "roundName": "Round 1",
  "participantIds": [
    "66666666-6666-6666-6666-666666666666"
  ],
  "participantType": "User"
}
```

---

#### `participant.assigned.match`

**When:** Participants are assigned to a match.

**Payload `data`:**

| Field             | Type   | Description                          |
|------------------|--------|--------------------------------------|
| `tournamentId`   | string | UUID of the tournament               |
| `roundId`        | string | UUID of the round (if any)           |
| `matchId`        | string | UUID of the match                    |
| `participantIds` | array  | UUIDs (may be omitted until the match tracks them) |
| `participantType`| string | `"User"` or `"Team"`                 |
| `status`         | string | Match type or status                 |

**Example `data`:**
```json
{
  "matchId": "33333333-3333-3333-3333-333333333333",
  "tournamentId": "22222222-2222-2222-2222-222222222222",
  "roundId": "55555555-5555-5555-5555-555555555555",
  "status": "Duel"
}
```

---

#### `participant.removed.round`

**When:** A participant is removed from a round.

**Payload `data`:** Same shape as `participant.assigned.round`.

**Example `data`:**
```json
{
  "tournamentId": "22222222-2222-2222-2222-222222222222",
  "roundId": "55555555-5555-5555-5555-555555555555",
  "roundName": "Round 1",
  "participantIds": [
    "66666666-6666-6666-6666-666666666666"
  ],
  "participantType": "Team"
}
```

---

#### `participant.removed.group`

**When:** A participant is removed from a group.

**Payload `data`:** Same shape as `participant.assigned.group`.

**Example `data`:**
```json
{
  "tournamentId": "22222222-2222-2222-2222-222222222222",
  "groupId": "88888888-8888-8888-8888-888888888888",
  "groupName": "Group A",
  "roundId": "55555555-5555-5555-5555-555555555555",
  "roundName": "Round 1",
  "participantIds": [
    "66666666-6666-6666-6666-666666666666"
  ],
  "participantType": "Team"
}
```

---

#### `participant.removed.match`

**When:** A participant is removed from a match.

**Payload `data`:** Same shape as `participant.assigned.match`.

**Example `data`:**
```json
{
  "tournamentId": "22222222-2222-2222-2222-222222222222",
  "roundId": "55555555-5555-5555-5555-555555555555",
  "matchId": "33333333-3333-3333-3333-333333333333",
  "participantIds": [
    "66666666-6666-6666-6666-666666666666"
  ],
  "participantType": "User"
}
```

---

### 4. Match events

#### `match.created`

**When:** A match is created (e.g. in a round).

**Payload `data`:**

| Field          | Type   | Description                |
|---------------|--------|----------------------------|
| `tournamentId`| string | UUID of the tournament     |
| `matchId`     | string | UUID of the match          |
| `roundId`     | string | UUID of the round (if any) |
| `status`      | string | Match status (e.g. match type or status) |

**Example `data`:**
```json
{
  "tournamentId": "t1t2t3t4-t5t6-7890-tour-nament1234567",
  "matchId": "m1m2m3m4-m5m6-7890-mat-ch1234567890",
  "roundId": "ro1ro2ro3-ro4-ro5-7890-round12345678",
  "status": "SingleElimination"
}
```

*Note: For `match.created`, `status` is currently the match type (e.g. `SingleElimination`, `DoubleElimination`, `Ffa`).*

---

#### `match.updated`

**When:** A match is updated (e.g. score, status change).

**Payload `data`:** Same structure as `match.created`; `status` is the match **status** (e.g. `Scheduled`, `InProgress`, `Completed`, `Cancelled`).

| Field          | Type   | Description                |
|---------------|--------|----------------------------|
| `tournamentId`| string | UUID of the tournament     |
| `matchId`     | string | UUID of the match          |
| `roundId`     | string | UUID of the round (if any) |
| `status`      | string | Match status               |

---

### 5. Tournament published

#### `tournament.published`

**When:** A tournament successfully published making it publicly available to users.

**Payload `data`:**

| Field          | Type   | Description                |
|---------------|--------|----------------------------|
| `tournamentId`| string | UUID of the tournament     |
| `tournamentName` | string | Name of the tournament |
| `status`      | string | Status of tournament publication |

**Example `data`:**

```json
{
  "tournamentId": "22222222-2222-2222-2222-222222222222",
  "tournamentName": "Spring Championship",
  "status": "Published"
}
```

---

#### `tournament.rules.updated`

**When:** Tournament **rules** are saved or changed.

**Payload `data`:** Same shape as `tournament.published` — identifies the tournament and its current high-level status.

| Field             | Type   | Description                    |
|-------------------|--------|--------------------------------|
| `tournamentId`    | string | UUID of the tournament         |
| `tournamentName`  | string | Name of the tournament         |
| `status`          | string | Tournament status (e.g. lifecycle) |

**Example `data`:**

```json
{
  "tournamentId": "94b24563-4816-451f-b7e7-1720e2a8e7c9",
  "tournamentName": "Auto Test Tournament",
  "status": "Upcoming"
}
```

---

#### `tournament.points.updated`

**When:** The tournament **points** configuration is saved or changed.

**Payload `data`:** Same fields as `tournament.rules.updated`.

| Field             | Type   | Description                    |
|-------------------|--------|--------------------------------|
| `tournamentId`    | string | UUID of the tournament         |
| `tournamentName`  | string | Name of the tournament         |
| `status`          | string | Tournament status (e.g. lifecycle) |

**Example `data`:**

```json
{
  "tournamentId": "94b24563-4816-451f-b7e7-1720e2a8e7c9",
  "tournamentName": "Auto Test Tournament",
  "status": "Upcoming"
}
```

---

### 6. Announcement events (chat-based)

Sent when an announcement message is posted to a tournament/round/group/match conversation.

**Common `data` fields for all announcement events:**

| Field           | Type   | Description                |
|----------------|--------|----------------------------|
| `tournamentId` | string | UUID of the tournament     |
| `messageContent` | string | Announcement text       |
| `senderId`     | string | UUID of the user who sent the message |

**Context field (one of):**

| Field     | Type   | Present when              |
|----------|--------|---------------------------|
| `matchId`| string | Match-specific announcement |
| `groupId`| string | Group-specific announcement |
| `roundId`| string | Round-specific announcement |

If none of `matchId`, `groupId`, or `roundId` are set, the announcement is **tournament-wide**.

---

#### `id.pass.announcement`

**When:** Match targeted announcement specifically meant to share room ID and password details or joining credentials.

**Payload `data`:** `tournamentId`, `messageContent`, `senderId`, `matchId`.

**Example `data`:**
```json
{
  "tournamentId": "22222222-2222-2222-2222-222222222222",
  "matchId": "33333333-3333-3333-3333-333333333333",
  "messageContent": "Room ID: 123456, PASS: abcd",
  "senderId": "44444444-4444-4444-4444-444444444444"
}
```

---

#### `match.announcement`

**When:** Announcement targets a specific match.

**Payload `data`:** `tournamentId`, `messageContent`, `senderId`, `matchId`.

**Example `data`:**
```json
{
  "tournamentId": "t1t2t3t4-t5t6-7890-tour-nament1234567",
  "messageContent": "Match starts in 5 minutes.",
  "senderId": "s1s2s3s4-s5s6-7890-sen-der1234567890",
  "matchId": "m1m2m3m4-m5m6-7890-mat-ch1234567890"
}
```

---

#### `group.announcement`

**When:** Announcement targets a group.

**Payload `data`:** `tournamentId`, `messageContent`, `senderId`, `groupId`.

---

#### `round.announcement`

**When:** Announcement targets a round.

**Payload `data`:** `tournamentId`, `messageContent`, `senderId`, `roundId`.

---

#### `tournament.announcement`

**When:** Announcement is tournament-wide (no match/group/round target).

**Payload `data`:** `tournamentId`, `messageContent`, `senderId` only.

---

## Summary table

| Event type                              | Trigger                         | Key `data` fields |
|----------------------------------------|----------------------------------|-------------------|
| `team.member.joined`                   | User added to team               | teamId, teamName, userId, userName |
| `team.member.removed`                  | User removed from team          | teamId, teamName, userId, userName |
| `tournament.player.joined`            | Player joined tournament         | tournamentId, participantId, participantType, registrationId, registrationStatus |
| `tournament.team.joined`              | Team joined tournament           | tournamentId, participantId, participantType, registrationId, registrationStatus |
| `participant.assigned.round`           | Assigned to a round              | tournamentId, roundId, roundName, participantIds, participantType |
| `participant.assigned.group`           | Assigned to a group              | tournamentId, roundId, roundName, groupId, groupName, participantIds, participantType |
| `participant.assigned.match`           | Assigned to a match              | tournamentId, roundId, matchId, participantIds, participantType, status |
| `participant.removed.round`            | Removed from a round             | tournamentId, roundId, roundName, participantIds, participantType |
| `participant.removed.group`            | Removed from a group             | tournamentId, roundId, roundName, groupId, groupName, participantIds, participantType |
| `participant.removed.match`            | Removed from a match             | tournamentId, roundId, matchId, participantIds, participantType, status |
| `match.created`                        | Match created                    | tournamentId, matchId, roundId, status |
| `match.updated`                        | Match updated                    | tournamentId, matchId, roundId, status |
| `tournament.published`                 | Tournament published             | tournamentId, tournamentName, status |
| `tournament.rules.updated`           | Tournament rules updated         | tournamentId, tournamentName, status |
| `tournament.points.updated`          | Tournament points updated        | tournamentId, tournamentName, status |
| `id.pass.announcement`                 | Match ID and credentials         | tournamentId, messageContent, senderId, matchId |
| `match.announcement`                   | Announcement to a match          | tournamentId, messageContent, senderId, matchId |
| `group.announcement`                   | Announcement to a group          | tournamentId, messageContent, senderId, groupId |
| `round.announcement`                   | Announcement to a round          | tournamentId, messageContent, senderId, roundId |
| `tournament.announcement`              | Tournament-wide announcement     | tournamentId, messageContent, senderId |

---

## Subscription

When creating or updating a webhook, set the `events` array to the event types you want (e.g. `["match.created", "match.updated"]`). If `events` is null or empty, the webhook subscribes to **all** events above.

---

## Custom payload format

If you use **payload format** `custom` and a **Custom Payload Template**, you can use placeholders such as:

- `{{eventType}}`, `{{eventId}}`, `{{timestamp}}`, `{{tenantId}}`
- `{{tournamentId}}`, `{{externalId}}`, `{{participantId}}`, `{{participantType}}`
- Any key from the event’s `Data` (e.g. `{{matchId}}`, `{{registrationId}}`, `{{messageContent}}`).

The payload sent to your URL will then be the result of applying these placeholders to your template JSON.

---

*Document generated from 16Arena webhook implementation. Default payload format; optional signature via `X-Webhook-Signature` when Secret Key is set.*
