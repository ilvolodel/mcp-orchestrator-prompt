# MCP Orchestrator Prompt

Complete system prompt for a multi-MCP orchestrator agent that coordinates 9 different MCP servers.

## 📋 Overview

This prompt enables an AI agent to intelligently orchestrate multiple specialized MCP (Model Context Protocol) services:

- **MS365-Email** - Email operations
- **MS365-Calendar** - Calendar and meeting management  
- **MS365-Drive** - File storage and sharing
- **MS365-Teams** - Teams messaging (8 tools)
- **MS365-Users** - Directory lookups
- **PEC** - Italian certified email
- **TrustySign** - Digital signature
- **TrustyVault** - Authentication
- **Documents** - PDF/DOCX operations

**Total: 54 tools across 9 services**

## 🎯 Key Features

### Autonomous Behavior
- **Direct execution**: When user says "sign", agent signs without asking "Should I send OTP?"
- **Smart defaults**: Makes reasonable choices instead of asking unnecessary questions
- **Minimal confirmations**: Only asks for external/destructive operations

### MS365-Teams (8 Tools)
- ✅ teams_list_messages - Read conversation history (NEW!)
- ✅ teams_create_chat - Create group chats
- ✅ teams_get_chat - Get chat details
- ✅ teams_send_message - Send messages
- ✅ teams_list_chats - List recent chats
- ✅ teams_list_teams - List all Teams
- ✅ teams_list_channels - List channels
- ✅ teams_send_channel_message - Post to channels

### Signature Workflow
**Before:**
```
User: "Sign contract"
Agent: "Should I send SMS OTP?" ❌
User: "Yes"
Agent: "Should I sign contract.pdf?" ❌
User: "YES!"
```

**Now:**
```
User: "Sign contract"
Agent: "SMS OTP arriving for signature..." ✅
[User provides OTP]
Agent: "Signed! ✓" ✅
```

## 📁 Repository Structure

```
mcp-orchestrator-prompt/
├── ORCHESTRATOR_PROMPT_2025-02-04.md  ← Latest prompt (use this!)
├── README.md                           ← This file
├── USAGE.md                            ← How to use the prompt
├── UPDATE_GUIDE.md                     ← How to update in the future
├── CHANGELOG.md                        ← Version history
└── prompts/                            ← Historical versions
    └── archive/
```

## 🚀 Quick Start

### 1. Copy the Prompt
```bash
# Latest version
cp ORCHESTRATOR_PROMPT_2025-02-04.md your-agent-config/
```

### 2. Configure Your Agent
Use the prompt as your agent's system prompt. The agent needs access to:
- TrustyVault authentication endpoint
- 9 MCP servers (MS365-Email, Calendar, Drive, Teams, Users, PEC, TrustySign, TrustyVault, Documents)

### 3. Test Basic Flow
```
User: "Read my email"
Agent: Authenticates → Lists emails → Shows results
```

## 📖 Documentation

- **[USAGE.md](USAGE.md)** - Detailed usage instructions
- **[UPDATE_GUIDE.md](UPDATE_GUIDE.md)** - How to update the prompt when MCP tools change
- **[CHANGELOG.md](CHANGELOG.md)** - Version history with detailed changes

## 🔄 Recent Updates (2025-02-04)

### MS365-Teams: 7 → 8 Tools
- ✅ Added `teams_list_messages` (read conversation history)
- ✅ Added `teams_create_chat` (missing tool)
- ✅ Added `teams_get_chat` (missing tool)
- ❌ Removed `teams_create_meeting` (doesn't exist in code)
- 🔧 Fixed `teams_list_channels` to use `team_id` instead of `team_name`
- 🔧 Fixed `teams_send_channel_message` to use `team_id + channel_id`

### Autonomous Behavior
- 🚀 Sign documents directly without confirmation
- 🚀 Fewer unnecessary questions
- 🚀 Confirmation only for external/destructive operations

### Optimized Workflows
- ✅ Workflow 5.5: Search Teams conversation history
- ✅ Workflow 7: Corrected channel forwarding (team_id/channel_id)
- ✅ Workflow 3: Simplified sign & send (no signature confirmation)

See [CHANGELOG.md](CHANGELOG.md) for complete details.

## 🤝 Contributing

When MCP tools change:
1. Read [UPDATE_GUIDE.md](UPDATE_GUIDE.md) for step-by-step instructions
2. Update the prompt file
3. Update CHANGELOG.md
4. Create dated copy: `ORCHESTRATOR_PROMPT_YYYY-MM-DD.md`
5. Move old version to `prompts/archive/`
6. Commit with descriptive message

## 📝 License

Proprietary - Internal use only

## 🔗 Related Projects

- [ms365-teams](https://github.com/ilvolodel/ms365-teams) - MS365-Teams MCP server
- TrustyVault - Authentication service
- TrustySign - Digital signature service

## 📧 Contact

For questions or issues, contact the development team.

---

**Last Updated:** February 4, 2025  
**Version:** 1.1.0  
**Tools:** 54 total (8 in MS365-Teams)
