# Changelog

All notable changes to the MCP Orchestrator Prompt will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2025-02-04

### Added - MS365-Teams

- **teams_list_messages** - NEW tool to read conversation history from chats
  - Parameters: `session_token`, `chat_id`, `max_results` (1-50, default 50)
  - Returns messages ordered newest → oldest
  - Use case: "What did Mario say about deadline?" → searches last 50 messages
  
- **teams_create_chat** - Create 1-on-1 or group chats
  - Parameters: `session_token`, `members` (array), `topic` (optional)
  - Auto-resolves member names (e.g., ["Mario Rossi", "Luigi Verdi"])
  - Returns: `{chat_id, chatType: "oneOnOne"/"group"}`
  
- **teams_get_chat** - Get chat details
  - Parameters: `session_token`, `chat_id`
  - Returns: chat metadata, members list, creation date

- **Workflow 5.5** - "Search conversation history with colleague"
  - Demonstrates teams_list_messages + keyword search pattern
  - Example: Search Mario's messages for "deadline" keyword

### Changed - MS365-Teams

- **teams_list_channels** - Parameter change (BREAKING)
  - Before: `teams_list_channels(session_token, team_name)`
  - After: `teams_list_channels(session_token, team_id)`
  - Must now get team_id from teams_list_teams first

- **teams_send_channel_message** - Parameters change (BREAKING)
  - Before: `teams_send_channel_message(st, team_name, channel_name, message)`
  - After: `teams_send_channel_message(st, team_id, channel_id, message, subject?)`
  - Added optional `subject` parameter for message title

- **Tool count** - Updated from 7 to 8 tools
  - Documentation now correctly reflects actual codebase

- **Critical Notes section** - Enhanced with:
  - Read messages capability (up to 50 per call)
  - Group chat creation patterns
  - Message ordering clarification (newest → oldest)

### Removed - MS365-Teams

- **teams_create_meeting** - Removed from documentation
  - Tool does not exist in MS365-Teams MCP server codebase
  - Meeting creation handled by MS365-Calendar service instead
  - Prevents confusion and incorrect tool calls

### Fixed - Workflows

- **Workflow 7** - "Forward email with attachments to team channel"
  - Corrected to use teams_list_teams → team_id → teams_list_channels → channel_id
  - Fixed attachment handling to use download_url directly
  - No longer uses non-existent team_name parameter

- **Workflow 3** - "Sign PDF from PEC and send to client"
  - Removed unnecessary "Should I sign it?" confirmation
  - Simplified to: signature_wizard_start → user provides OTP → complete
  - Kept external recipient confirmation ("Send to client@company.com?")

### Changed - Agent Behavior

- **Signature operations** - Now execute directly without confirmation
  - Before: Agent asks "Should I send SMS OTP?" then "Should I sign?"
  - After: Agent executes signature_wizard_start and says "SMS OTP arriving..."
  - Rationale: User saying "sign" IS the confirmation

- **Removed from confirmation requirements:**
  - Line 52: "BEFORE starting signature" removed from "When to Ask"
  - Line 84: "Starting signature process" removed from confirmation list
  - Line 233: "Starting signature (triggers SMS)" removed from Ask ONCE list

- **Added explicit rule (lines 57-61):**
  ```
  When user says "sign document":
  - Proceed DIRECTLY with signature_wizard_start
  - SMS OTP will arrive automatically
  - User will provide OTP when ready
  - NO need to ask "Should I send OTP?"
  ```

- **Updated DO NOT section (line 128):**
  - Clarified: Execute WRITE operations when user directly requests ("sign", "send", "create")
  - Only confirm for destructive/external operations

### Added - Documentation

- **Quick Decision Table entries:**
  - "Read Teams conversation" → teams_list_messages
  - "Create group chat" → teams_create_chat

- **Cross-Service Patterns:**
  - Pattern #5: Teams (list_messages) → Search keywords → Reply
  - Pattern #6: Email/PEC → Teams (forward to channel with attachments)

- **Example interaction (line 423-431):**
  - Shows signature without confirmation
  - Demonstrates faster, more direct workflow

### Statistics

- **Lines modified:** ~45 sections
- **Workflows added:** 1 (Workflow 5.5)
- **Workflows optimized:** 2 (Workflow 3, 7)
- **Tools documented:** 8 in MS365-Teams (was 7)
- **Confirmations removed:** 6 locations
- **Examples updated:** 3 major examples

## [1.0.0] - 2025-01-29 (Baseline)

### Initial Release

Complete orchestrator prompt supporting:

- **MS365-Email** (11 tools) - Email operations, search, attachments
- **MS365-Calendar** (9 tools) - Calendar management, meetings, scheduling
- **MS365-Drive** (9 tools) - File storage, sharing, permissions
- **MS365-Teams** (7 tools) - Teams messaging, channels [UPDATED to 8 in v1.1.0]
- **MS365-Users** (3 tools) - Directory lookups, user profiles
- **PEC** (4 tools) - Italian certified email (PEC)
- **TrustySign** (2 tools) - Digital signature wizard
- **TrustyVault** (2 tools) - Authentication and session management
- **Documents** (4 tools) - PDF/DOCX reading, conversion, verification

**Total:** 54 tools across 9 services (as of v1.1.0)

### Features

- **Auto-resolve names:** Email, Calendar, Teams auto-resolve "Mario Rossi" to email
- **Smart defaults:** Reasonable choices instead of unnecessary questions
- **Minimal confirmations:** Only for external/destructive operations
- **Cross-service workflows:** 15+ documented multi-tool patterns
- **Error handling:** Graceful error messages without technical jargon

### Anti-Patterns Documented

- Pattern 1: Asking to confirm what user already said
- Pattern 2: Multiple confirmations for same action
- Pattern 3: Asking "Should I?" for obvious actions
- Pattern 4: Showing raw JSON instead of human-friendly output

## Breaking Changes Between Versions

### v1.0.0 → v1.1.0

**MS365-Teams:**
1. `teams_list_channels(team_name)` → `teams_list_channels(team_id)` ⚠️
2. `teams_send_channel_message(team_name, channel_name, ...)` → `teams_send_channel_message(team_id, channel_id, ...)` ⚠️
3. `teams_create_meeting` removed entirely (use MS365-Calendar instead) ⚠️

**Agent Behavior:**
1. Signature operations no longer ask for confirmation before sending OTP
2. Direct execution when user says "sign" (breaking for agents expecting confirmation)

**Mitigation:**
- Update any hardcoded tool calls to use new parameters
- Review agent configuration if signature confirmation was expected
- Test Teams channel operations with team_id/channel_id resolution

## Migration Guide

### From v1.0.0 to v1.1.0

**If using Teams channels:**
```python
# OLD (v1.0.0)
teams_send_channel_message(st, "Marketing", "general", "Hello")

# NEW (v1.1.0)
teams_list_teams(st) → find "Marketing" → team_id
teams_list_channels(st, team_id) → find "general" → channel_id  
teams_send_channel_message(st, team_id, channel_id, "Hello")
```

**If expecting signature confirmation:**
```
# OLD behavior (v1.0.0)
User: "Sign contract"
Agent: "Should I send SMS OTP?"
User: "Yes"
Agent: [proceeds]

# NEW behavior (v1.1.0)
User: "Sign contract"
Agent: "SMS OTP arriving..." [proceeds directly]
```

## Unreleased

### Planned

- Integration with MS365-SharePoint (when MCP server available)
- Batch operation optimization patterns
- Advanced calendar scheduling algorithms
- Multi-language support examples

---

## Version History Summary

| Version | Date | MS365-Teams Tools | Total Tools | Key Changes |
|---------|------|-------------------|-------------|-------------|
| 1.1.0 | 2025-02-04 | 8 | 54 | +teams_list_messages, signature auto-execute |
| 1.0.0 | 2025-01-29 | 7 | 51 | Initial release |

## Contributing

When updating this changelog:
1. Follow [Keep a Changelog](https://keepachangelog.com/) format
2. Use sections: Added, Changed, Deprecated, Removed, Fixed, Security
3. Note breaking changes clearly with ⚠️
4. Include migration guide for breaking changes
5. Update version history summary table

---

**For detailed update instructions, see [UPDATE_GUIDE.md](UPDATE_GUIDE.md)**
