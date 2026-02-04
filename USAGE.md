# Usage Guide

Complete guide for using the MCP Orchestrator Prompt in your AI agent.

## 📋 Prerequisites

Your agent needs:
1. **System Prompt**: Use `ORCHESTRATOR_PROMPT_YYYY-MM-DD.md` as system prompt
2. **MCP Servers**: Access to 9 MCP servers via Model Context Protocol
3. **TrustyVault**: Authentication endpoint for session_token management
4. **Environment**: API keys, endpoints configured

## 🔧 Configuration

### 1. Agent Setup

**Copy the latest prompt:**
```bash
cp ORCHESTRATOR_PROMPT_2025-02-04.md /path/to/your/agent/system-prompt.md
```

**Configure agent to:**
- Load prompt as system instructions
- Connect to MCP servers (SSE or HTTP endpoints)
- Handle TrustyVault authentication flow

### 2. MCP Server Endpoints

Example configuration:
```yaml
mcp_servers:
  ms365_email:
    url: "https://ms365-email.example.com/mcp/sse"
    auth: "Bearer ${API_KEY}"
  
  ms365_teams:
    url: "https://ms365-teams.brainaihub.tech/mcp/sse"
    auth: "Bearer ${API_KEY}"
  
  trustysign:
    url: "https://trustysign.example.com/mcp/sse"
    auth: "Bearer ${API_KEY}"
  
  # ... other 6 services
```

### 3. TrustyVault Authentication

Agent must handle this flow:
```
User → tool_trustyvault_authenticate(email)
     → Email sent to user
     → User clicks link, gets OTP from dashboard
     → tool_trustyvault_verify_otp(email, otp)
     → session_token received
     → Use session_token for all subsequent calls
```

## 🎯 Usage Patterns

### Pattern 1: Read Operations (Auto-execute)

**User:** "Read my email"

**Agent behavior:**
1. Check if authenticated (have session_token?)
2. If NO → Start authentication flow
3. If YES → Call `email_list_messages(session_token)` directly
4. Present results to user

**No confirmation needed!**

### Pattern 2: Write Operations (Internal - Auto-execute)

**User:** "Send email to Mario"

**Agent behavior:**
1. Authenticate if needed
2. Call `email_send_message(session_token, "Mario Rossi", subject, body)` directly
3. Confirm: "Email sent to Mario ✓"

**No confirmation needed!** (Mario is internal, name auto-resolves)

### Pattern 3: External Operations (Confirm Once)

**User:** "Send email to client@external.com"

**Agent behavior:**
1. Authenticate if needed
2. **ASK ONCE:** "Send to client@external.com? (external address)"
3. User: "Yes"
4. Call `email_send_message(...)`
5. Confirm: "Email sent ✓"

**Confirm external recipients only!**

### Pattern 4: Signature (Direct Execution)

**User:** "Sign contract"

**Agent behavior:**
1. Authenticate if needed
2. Find contract PDF
3. **Directly call** `signature_wizard_start(session_token, pdf_url, "analyze")`
4. Tell user: "SMS OTP arriving for signature..."
5. Wait for user to provide OTP
6. Call `signature_wizard_complete(session_token, transaction_id, otp)`
7. Confirm: "Contract signed ✓"

**No "Should I send OTP?" confirmation!**

### Pattern 5: Destructive Operations (Confirm Once)

**User:** "Delete last 10 emails"

**Agent behavior:**
1. **ASK ONCE:** "Delete 10 emails? (irreversible)"
2. User: "Yes"
3. Execute deletion
4. Confirm: "Deleted 10 emails ✓"

## 🔄 Cross-Service Workflows

### Email → Teams
```
User: "Forward last email to Mario via Teams"

Agent:
1. email_list_messages(st, max_results=1)
2. email_get_message(st, message_id)
3. teams_send_message(st, "Mario Rossi", f"FYI: {email.body}")
Done!
```

### PEC → Sign → Email
```
User: "Sign contract from PEC and send to client"

Agent:
1. pec_list_messages(st, search_query='SUBJECT "contract"')
2. pec_get_message(st, msg_id, attachment_index=0) → download_url
3. signature_wizard_start(st, download_url, "analyze")
4. Tell user: "SMS OTP arriving..."
5. User provides OTP
6. signature_wizard_complete(st, transaction_id, otp) → signed_pdf_url
7. ASK: "Send to client@company.com?"
8. email_send_message(st, "client@company.com", "Signed", attachments=[signed_pdf_url])
Done!
```

### Teams Conversation Search
```
User: "What did Mario say about the deadline?"

Agent:
1. teams_list_chats(st) → Find Mario's chat → chat_id
2. teams_list_messages(st, chat_id, max_results=50)
3. Search messages for "deadline"
4. Present: "Mario said on Jan 15: 'Deadline is Feb 1st'"

User: "Reply I'm on track"
5. teams_send_message(st, chat_id, "Thanks! I'm on track")
Done!
```

## 🚨 Common Mistakes to Avoid

### ❌ DON'T: Ask unnecessary confirmations
```
User: "Read email"
Agent: "Should I read your email?" ← WRONG!
```

### ❌ DON'T: Use users_search when auto-resolve works
```
User: "Send email to Mario"
Agent: users_search("Mario") ← WRONG!
Agent: email_send_message(st, "Mario Rossi", ...) ← CORRECT!
```

### ❌ DON'T: Download then upload for attachments
```
files_download(file_id) → get_url
files_upload(file_content) → new_url ← WRONG!

Use download_url directly:
email_send_message(..., attachments=[download_url]) ← CORRECT!
```

### ❌ DON'T: Ask multiple times for same action
```
User: "Sign contract"
Agent: "Should I send OTP?" ← WRONG!
User: "Yes"
Agent: "Should I sign?" ← WRONG AGAIN!
```

### ✅ DO: Execute directly when user requests
```
User: "Sign contract"
Agent: "SMS OTP arriving..." ← CORRECT!
[waits for OTP]
Agent: "Signed! ✓" ← CORRECT!
```

## 🎯 When to Ask Confirmation

**ONLY ask for:**
1. ⚠️ External recipients (new email addresses)
2. ⚠️ Destructive operations (delete, remove)
3. ⚠️ Batch operations (process 50 files)

**DON'T ask for:**
- ✅ Read operations
- ✅ Internal messages/emails
- ✅ File uploads
- ✅ Signatures (user already said "sign")
- ✅ Creating meetings/events

## 📊 Service Selection Quick Reference

| User Says | Service | Tool |
|-----------|---------|------|
| "Read email" | MS365-Email | email_list_messages |
| "Send Teams message" | MS365-Teams | teams_send_message |
| "What did X say?" | MS365-Teams | teams_list_messages |
| "Create group chat" | MS365-Teams | teams_create_chat |
| "Create meeting" | MS365-Calendar | calendar_create_meeting_* |
| "Sign document" | TrustySign | signature_wizard_start |
| "Verify signature" | Documents | pdf_verify_signature |
| "Read PDF" | Documents | pdf_read |
| "Send PEC" | PEC | pec_send_message |

## 🐛 Troubleshooting

### "User not found"
- Check if auto-resolve is being used ("Mario Rossi" works!)
- Don't use users_search unless explicitly asked

### "Chat not found" for Teams
- Use teams_list_chats first to get chat_id
- Or send by name: teams_send_message(st, "Mario Rossi", ...)

### "Multiple users found"
- If users_search returns 2+, ASK USER which one
- Never pick first automatically

### Authentication issues
- Verify session_token is being passed to all tools
- Check if token expired (re-authenticate)

## 📚 Additional Resources

- See `ORCHESTRATOR_PROMPT_YYYY-MM-DD.md` for complete tool reference
- See `CHANGELOG.md` for version history
- See `UPDATE_GUIDE.md` for maintaining the prompt

---

**Questions?** Check the full prompt for detailed tool signatures and workflow examples.
