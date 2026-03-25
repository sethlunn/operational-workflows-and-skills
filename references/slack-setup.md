# Slack Setup

Use this only when Slack publishing is requested and Slack is not configured yet.

## Required Environment Variables

Add these to `~/.zshrc`:

- `SLACK_BOT_TOKEN`
  - The bot token for the Slack app.
- `SLACK_HEALTHCHECK_CHANNEL_ID`
  - The default channel id for health-check rollups, for example `C0123456789`.

## Recommended Slack App Shape

Create a Slack app with a bot user and install it to the workspace.

Recommended bot scopes:

- `chat:write`
  - Required to post parent messages and thread replies.
- `chat:write.public`
  - Optional. Add this only if the bot should post in public channels without being invited first.
- `channels:read`
  - Optional. Needed only when resolving public channel names like `#healthchecks`.
- `groups:read`
  - Optional. Needed only when resolving private channel names instead of using a channel id.

## Channel Choice

Prefer channel ids over channel names.

Why:

- fewer Slack scopes are needed
- no extra channel lookup call is required
- it avoids ambiguity when channels are renamed

## Posting Model

For health checks, prefer one parent summary message plus one threaded reply per service.

If the parent is already posted, reply using the existing thread timestamp.

## Failure Rules

- If `SLACK_BOT_TOKEN` is missing, fail fast and ask for setup.
- If `SLACK_HEALTHCHECK_CHANNEL_ID` is missing, require the user to specify a channel explicitly or add the default channel id.
- If name-based channel resolution fails, ask for the channel id instead of widening Slack scopes unnecessarily.
