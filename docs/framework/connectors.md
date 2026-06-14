# Connectors

Connectors bridge external chat platforms to the LegionForge gateway. They translate platform messages into gateway task submissions and stream the SSE response back as platform-native updates.

All connectors follow the same pattern:

1. Listen for messages with a trigger pattern (e.g., `!<task>` for Discord)
2. Submit the task to the gateway with the platform user's credentials
3. Subscribe to the SSE stream
4. Edit a placeholder reply with streaming updates
5. Post the final result

## Available connectors

| Connector | Platform | Trigger | Module |
|---|---|---|---|
| Discord | Discord servers | `!<task>` or `@bot <task>` | `src/connectors/discord.py` |
| Telegram | Telegram bot | `/task <task>` or `@bot <task>` | `src/connectors/telegram.py` |
| Slack | Slack workspaces | `/legionforge <task>` slash command | `src/connectors/slack.py` |
| WhatsApp | WhatsApp Business | message starting with `!` | `src/connectors/whatsapp.py` |
| Webhook | Generic HTTP | `POST` with JSON body | `src/connectors/webhook.py` |

## Starting a connector

Each connector has its own Make target:

```bash
make discord-start
make telegram-start
make slack-start
make whatsapp-start
make webhook-start
```

Connectors read their platform credentials (Discord bot token, Slack signing secret, etc.) from macOS Keychain at startup. Service names follow `legionforge_<platform>_<credential>`.

## Auth bridging

Connectors don't have their own API key into the gateway. Instead, each platform user is mapped to a `gateway_users` row at first message via the `connector_identities` table:

```sql
CREATE TABLE connector_identities (
    connector TEXT NOT NULL,      -- 'discord', 'telegram', etc.
    platform_user_id TEXT NOT NULL,
    gateway_user_id BIGINT REFERENCES gateway_users(id),
    PRIMARY KEY (connector, platform_user_id)
);
```

First-time users get auto-provisioned with a default quota. Admins can adjust per-user quotas via `make set-quota`.

## Discord connector

The Discord connector is the most polished and serves as the reference implementation.

### Setup

1. Create a Discord application and bot at <https://discord.com/developers/applications>
2. Enable the **Message Content Intent**
3. Copy the bot token, store it:
   ```bash
   security add-generic-password -A \
     -s legionforge_discord_bot_token \
     -a api_key -w <bot-token>
   ```
4. Invite the bot to your server with the `Send Messages` and `Read Message History` permissions
5. Run `make discord-start`

### Usage

```
!summarize the latest Python 3.13 release notes
```

The bot acknowledges with a "Thinking..." reply, then edits it as the SSE stream produces output. The final reply contains the full result and a footer with token usage.

## Slack connector

Slack uses a slash command rather than a message prefix. The slash command points to the gateway's `/connectors/slack/command` endpoint, which validates the signing secret and forwards to the worker.

```bash
security add-generic-password -A \
  -s legionforge_slack_signing_secret -a api_key -w <secret>

security add-generic-password -A \
  -s legionforge_slack_bot_token -a api_key -w <token>

make slack-start
```

## Webhook connector

The generic webhook connector is the simplest:

```http
POST /connectors/webhook
Content-Type: application/json

{
  "prompt": "What's the weather in Tokyo?",
  "callback_url": "https://example.com/legionforge/callback"
}
```

The gateway processes the task and posts the result to `callback_url` when done.

## Writing a new connector

The connector contract is minimal:

```python
class Connector:
    async def listen(self) -> None:
        """Listen for platform messages and submit them to the gateway."""

    async def stream_back(self, task_id: str, platform_target: str) -> None:
        """Subscribe to /tasks/{task_id}/stream and post updates to the platform."""
```

Copy `src/connectors/discord.py` as a template. The trickiest part is platform-specific: how to edit a previously-sent message as the stream progresses without spamming the API.
