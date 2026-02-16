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

### 3. Tournament round events

#### `tournament.round.participant.assigned`

**When:** One or more participants are assigned to a round (e.g. bracket slots).

**Payload `data`:**

| Field             | Type   | Description                          |
|------------------|--------|--------------------------------------|
| `tournamentId`   | string | UUID of the tournament               |
| `roundId`        | string | UUID of the round                    |
| `roundName`      | string | Round display name                   |
| `participantIds` | array  | List of UUIDs of assigned participants |
| `participantType`| string | `"Player"` or `"Team"`              |

**Example `data`:**
```json
{
  "tournamentId": "t1t2t3t4-t5t6-7890-tour-nament1234567",
  "roundId": "ro1ro2ro3-ro4-ro5-7890-round12345678",
  "roundName": "Round 1",
  "participantIds": ["p1p2p3p4-p5p6-7890-play-er1234567890", "p2p3p4p5-p6p7-8901-laye-r2345678901"],
  "participantType": "Team"
}
```

---

#### `tournament.round.participant.removed`

**When:** A participant is removed from a round.

**Payload `data`:** Same structure as `tournament.round.participant.assigned`; `participantIds` contains only the removed participant(s).

| Field             | Type   | Description                          |
|------------------|--------|--------------------------------------|
| `tournamentId`   | string | UUID of the tournament               |
| `roundId`        | string | UUID of the round                    |
| `roundName`      | string | Round display name                   |
| `participantIds` | array  | List of UUIDs (removed participants)|
| `participantType`| string | `"Player"` or `"Team"`              |

---

### 4. Tournament group events

#### `tournament.group.participant.assigned`

**When:** Participants are assigned to a group (e.g. league group).

**Payload `data`:**

| Field             | Type   | Description                          |
|------------------|--------|--------------------------------------|
| `tournamentId`   | string | UUID of the tournament               |
| `groupId`        | string | UUID of the group                    |
| `groupName`      | string | Group display name                   |
| `roundId`        | string | UUID of the round this group belongs to |
| `participantIds` | array  | List of UUIDs of assigned participants (from comma-separated string) |
| `participantType`| string | `"Player"` or `"Team"`              |

**Example `data`:**
```json
{
  "tournamentId": "t1t2t3t4-t5t6-7890-tour-nament1234567",
  "groupId": "g1g2g3g4-g5g6-7890-grou-p1234567890",
  "groupName": "Group A",
  "roundId": "ro1ro2ro3-ro4-ro5-7890-round12345678",
  "participantIds": ["p1p2p3p4-p5p6-7890-play-er1234567890"],
  "participantType": "Team"
}
```

---

#### `tournament.group.participant.removed`

**When:** A participant is removed from a group.

**Payload `data`:** Same structure as `tournament.group.participant.assigned`; `participantIds` lists the removed participant(s).

---

### 5. Match events

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

#### `match.participant.assigned`

**When:** A participant (player/team) is assigned to a match slot.

**Payload `data`:** Same shape as `match.created` / `match.updated` (tournamentId, matchId, roundId, status).

| Field          | Type   | Description                |
|---------------|--------|----------------------------|
| `tournamentId`| string | UUID of the tournament     |
| `matchId`     | string | UUID of the match          |
| `roundId`     | string | UUID of the round (if any) |
| `status`      | string | Match type (e.g. `SingleElimination`) |

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
| `tournament.round.participant.assigned`| Participants assigned to round  | tournamentId, roundId, roundName, participantIds, participantType |
| `tournament.round.participant.removed` | Participant removed from round  | tournamentId, roundId, roundName, participantIds, participantType |
| `tournament.group.participant.assigned`| Participants assigned to group   | tournamentId, groupId, groupName, roundId, participantIds, participantType |
| `tournament.group.participant.removed` | Participant removed from group  | tournamentId, groupId, groupName, roundId, participantIds, participantType |
| `match.created`                        | Match created                    | tournamentId, matchId, roundId, status |
| `match.updated`                        | Match updated                    | tournamentId, matchId, roundId, status |
| `match.participant.assigned`           | Participant assigned to match    | tournamentId, matchId, roundId, status |
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
