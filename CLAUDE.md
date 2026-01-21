# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DailyHoroscope is a LangBot plugin that provides daily horoscope functionality via an external API. Users can query their horoscope once per day by sending a message containing "运势" (fortune/horoscope).

## Architecture

### Plugin Structure

This is a LangBot plugin following the LangBot Plugin SDK architecture:

- **Plugin Entry Point**: `main.py` defines the `DailyHoroscope` class that inherits from `BasePlugin`
- **Event Listener Component**: Located in `components/event_listener/`, handles incoming messages
- **Component Registration**: Defined in `manifest.yaml` under `spec.components.EventListener`

### Key Components

1. **DefaultEventListener** (`components/event_listener/default.py`):
   - Listens for both `PersonMessageReceived` and `GroupMessageReceived` events
   - Filters messages containing "运势" keyword
   - Enforces once-per-day limit per user using plugin storage
   - Fetches horoscope image from external API and returns as base64-encoded image

2. **Data Persistence**:
   - Uses LangBot's plugin storage mechanism (`get_plugin_storage`/`set_plugin_storage`)
   - Stores user request timestamps with key pattern: `user_request_{user_id}`
   - Dates stored as UTF-8 encoded bytes in format `YYYY-MM-DD`

3. **Configuration** (`manifest.yaml`):
   - `time_zone`: Configurable timezone (default: `Asia/Shanghai`)
   - `api_token`: Required API token for external horoscope service

## Development Commands

### Install Dependencies
```bash
pip install -r requirements.txt
```

### Testing the Plugin

The plugin is designed to run within the LangBot framework. Refer to [LangBot Plugin Development Documentation](https://docs.langbot.app/en/plugin/dev/tutor.html) for testing and debugging instructions.

VSCode launch configuration is available at `.vscode/launch.json`.

## Important Implementation Details

### Event Handler Pattern

The event listener uses a decorator-based handler registration pattern within the `initialize()` method:

```python
@self.handler(events.PersonMessageReceived)
@self.handler(events.GroupMessageReceived)
async def handler(event_context: context.EventContext):
    # handler implementation
```

### Rate Limiting Mechanism

The once-per-day limit is timezone-aware:
1. Check if user has requested today using `has_requested_today(user_id)`
2. Compare stored date against current date in configured timezone
3. Record request timestamp after successful response with `record_request(user_id)`

### External API Integration

- API endpoint: `https://api.092399.xyz/api/yunshi/yunshi.php?type=today&token={api_token}`
- Returns image data (not JSON)
- Image is converted to base64 for LangBot message chain
- Uses `aiohttp` with `ssl=False` for async HTTP requests

## Message Chain Format

LangBot uses a message chain structure for sending/receiving messages:
- **Plain text**: `platform_message.Plain(text="...")`
- **Images**: `platform_message.Image(base64="...")`
- Wrap in `platform_message.MessageChain([...])`

## API Token Requirement

The plugin requires an API token from the service provider to prevent abuse. This should be configured via the plugin settings in LangBot, not hardcoded in the source.
