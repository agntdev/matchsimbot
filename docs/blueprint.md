# Game QA Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that emulates precise, deterministic in-match actions for game QA and demo purposes. It executes pre-defined action scripts against configured game servers, providing status updates, logs, and run history to validate multiplayer logic and telemetry.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Game QA engineers
- Demo/product teams
- Game developers

## Success criteria

- Scripts execute with millisecond-precise timing
- Status updates and logs are delivered via Telegram
- Game server actions are sent via configurable HTTP POST endpoints
- Run history and script storage persist across sessions

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with script management and run controls
- **/new_script** (command, actor: user, command: /new_script) — Upload or create a named action script (CSV/JSON)
- **/list_scripts** (command, actor: user, command: /list_scripts) — List stored scripts with metadata
- **/run** (command, actor: user, command: /run) — Start a script run against a match (requires script name and match ID)
- **/stop** (command, actor: user, command: /stop) — Stop the current active run
- **/status** (command, actor: user, command: /status) — Show current run status and last-run summary
- **/set_target** (command, actor: user, command: /set_target) — Configure the target game server endpoint
- **List Scripts** (button, actor: user, callback: list_scripts) — Show stored scripts in main menu
- **Upload Script** (button, actor: user, callback: new_script) — Initiate script upload process
- **Start Run** (button, actor: user, callback: run:start) — Trigger /run command with script selection

## Flows

### script_upload
_Trigger:_ /new_script

1. Request script name
2. Receive script file (CSV/JSON)
3. Validate format and store with metadata

_Data touched:_ Script

### match_run
_Trigger:_ /run

1. Validate script and match ID
2. Initialize scheduler with script timing
3. Send actions to game server endpoint
4. Report progress and final status

_Data touched:_ MatchSession, Script, RunHistory

### target_configuration
_Trigger:_ /set_target

1. Request game server URL
2. Validate endpoint connectivity
3. Store endpoint as active target

_Data touched:_ TargetEndpoint

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Script** _(retention: persistent)_ — Stored action sequence with metadata
  - fields: name, content, format_type, upload_timestamp
- **MatchSession** _(retention: persistent)_ — Active or completed match run context
  - fields: match_id, script_name, start_time, status, log
- **TargetEndpoint** _(retention: persistent)_ — Configured game server API endpoint
  - fields: url, auth_token, last_used
- **RunHistory** _(retention: persistent)_ — Historical run metadata for auditing
  - fields: run_id, script_name, match_id, outcome, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging and file transfers
- **Game Server API** (required) — Send emulated player actions
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure game server endpoint URL
- Set script retention policy (default: keep all until deleted)
- Adjust run progress ping frequency
- Enable/disable log file attachments

## Notifications

- Run status updates in controlling Telegram chat
- Log file attachments on run completion
- Error alerts with retry attempts

## Permissions & privacy

- Scripts and run history are stored securely and only accessible to the owner
- Game server credentials are encrypted at rest
- No third-party data sharing

## Edge cases

- Network failures during run execution
- Invalid or malformed script files
- Concurrent /run commands (blocked by default)
- Expired or invalid game server endpoints

## Required tests

- End-to-end script execution with timing validation
- Error handling during mid-run network interruption
- Script format validation for malformed inputs
- Run history persistence across bot restarts

## Assumptions

- Scripts use millisecond offsets for timing
- Default notification channel is the controlling chat
- HTTP POST used for game server actions
- Single active run per bot instance
