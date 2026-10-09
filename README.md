# Discord Server Backup Bot

A TypeScript / discord.js bot for saving and restoring a server's structural
configuration after accidental damage or a raid.

## What it backs up

- Non-managed roles, including their permissions, colors, display settings, and
  relative position metadata.
- Categories and supported text, announcement, voice, and stage channels.
- Channel topics, slowmode, NSFW setting, voice settings, and permission
  overwrites.
- Server name.
- Message content and embeds when explicitly requested for a premium server.

Snapshots are JSON files stored locally under `data/<server-id>/` by default.
The bot does not read or save messages, member lists, or message history.

## Setup

1. Create a bot application in the [Discord Developer Portal](https://discord.com/developers/applications)
   and add the bot to your server with the `bot` and `applications.commands`
   scopes plus **Manage Server**, **Manage Roles**, and **Manage Channels**
   permissions. For premium message history, also grant **View Channel**,
   **Read Message History**, and **Send Messages** in the channels to back up
   and restore.
   The bot's highest role must be above roles it needs to create or manage.
2. Set `DISCORD_TOKEN`, `CLIENT_ID`, and `BOT_OWNER_ID` in `.env`. The bot owner
   ID is the Discord user ID allowed to manage premium access. Optionally set
   `GUILD_ID` for quick guild-only command registration while developing.
   Existing `PREMIUM_GUILD_IDS` values seed the premium store the first time it
   is used; subsequent grants and revocations are saved to
   `data/premium-guilds.json` by default. Set `PREMIUM_STORE` to change that
   path. Keep `.env` private.
3. Install dependencies and start the bot:

   ```sh
   npm install
   npm run build
   npm start
   ```

   For a production build, run `npx tsc` and then `npm run start:prod`.

## Commands

- `/backup create [include_messages]` — save a snapshot; message history is
   optional and limited to IDs in `PREMIUM_GUILD_IDS`.
- `/backup list` — list snapshots for the current server.
- `/backup restore snapshot_id:<id>` — restore the server name and recreate
   missing roles and channels. Premium message history is replayed only for
   eligible servers and only into channels created by that restore.
- `/premium grant guild_id:<id>` and `/premium revoke guild_id:<id>` — manage
   premium status as the configured bot owner. These commands can be used in
   any server or in a direct message.

All commands are restricted to server administrators. The bot also checks its
own permissions before restoring. A restore is additive: it does not delete
server content or modify existing roles/channels. Existing roles are matched by
original ID, then name; channels are matched by original ID, then name and type.
Role permission overwrites are remapped to restored roles. Message replay creates
new messages authored by the bot; original authors, timestamps, reactions, and
attachments cannot be recreated.

Snapshots do not capture members, integrations, emojis, stickers, webhooks,
scheduled events, threads, forum/media channels, or the server's vanity/invite
state. Discord may prevent creating some roles because of role hierarchy or
permission limits. Restore channels in the same server only.

## Storage and operational notes

Backups contain permission configuration and server structure. Protect the
machine and backup directory accordingly, include the directory in your own
secure backup plan, and periodically test restoring to a test server. This bot
stores snapshots on the host only; it does not upload them to a third party.
