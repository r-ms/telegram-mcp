# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Telegram MCP Server - A Model Context Protocol (MCP) server that provides Telegram integration via the Telethon library. Exposes 70+ Telegram tools for chat management, messaging, contacts, groups, channels, and more.

## Development Commands

```bash
# Install dependencies
uv sync

# Run the MCP server
uv run main.py

# Generate Telegram session string (interactive)
uv run session_string_generator.py

# Lint and format
black .
flake8 .

# Run with Docker
docker compose up --build
```

## Architecture

**Single-file MCP server** (`main.py`):
- Uses FastMCP from `mcp.server.fastmcp` to register tools via `@mcp.tool()` decorators
- Telethon `TelegramClient` handles all Telegram API interactions
- Supports both string-based sessions (`TELEGRAM_SESSION_STRING`) and file-based sessions (`TELEGRAM_SESSION_NAME`)
- All tools are async functions decorated with `@mcp.tool(annotations=ToolAnnotations(...))`

**Key patterns**:
- `validate_id()` decorator validates `chat_id`/`user_id` parameters (handles int, string, @username formats)
- `log_and_format_error()` centralizes error handling with error codes logged to `mcp_errors.log`
- `format_entity()` / `format_message()` helpers convert Telethon objects to JSON-serializable dicts

**Environment variables** (`.env`):
- `TELEGRAM_API_ID` - from my.telegram.org/apps
- `TELEGRAM_API_HASH` - from my.telegram.org/apps
- `TELEGRAM_SESSION_STRING` - generated via `session_string_generator.py` (preferred)
- `TELEGRAM_SESSION_NAME` - file-based session name (fallback)

## Code Style

- Black formatter with line-length 99
- Flake8 linting (ignores E203, E501, W503)
- Target Python 3.11

## Global Search Tools

- `search_global_messages(query, min_date, max_date, chat_type, limit)` - search messages across all chats
- `get_active_chats(from_date, to_date, chat_type, limit)` - get chats with activity in date range
- `list_messages()` now supports `from_user` parameter to filter by sender

**API Limitation**: `from_user` filter only works for specific chat searches, not global search (Telegram API constraint)

## Important Notes

- File-based tools (send_file, download_media, etc.) have been intentionally removed due to MCP environment limitations
- GIF tools removed due to reliability issues with Telethon/Telegram API
- Bot-only functions detect account type and return clear messages when used with user accounts
