# Update Guide

Step-by-step guide for updating the orchestrator prompt when MCP tools change.

## 🎯 When to Update

Update the prompt when:
- ✅ New MCP tool added to any service
- ✅ Existing tool renamed or removed
- ✅ Tool parameters changed (new params, renamed params)
- ✅ Workflow optimizations discovered
- ✅ Agent behavior needs adjustment

## ⚡ Quick Update Process

### 1. Preparation (5 minutes)

```bash
# Clone the repository
git clone https://github.com/YOUR_ORG/mcp-orchestrator-prompt.git
cd mcp-orchestrator-prompt

# Check current state
cat ORCHESTRATOR_PROMPT_YYYY-MM-DD.md | grep "### Tools (" 
# This shows tool counts per service
```

### 2. Identify Changes (10 minutes)

**Check MCP Server Code:**
```bash
# Example: Check ms365-teams tools
cd /path/to/ms365-teams
grep "@mcp.tool()" src/main.py
# Count tools: 1, 2, 3... 8 tools

# Check tool signatures
grep -A 10 "def teams_" src/teams_operations.py
```

**Document what changed:**
```markdown
## Changes Found
- teams_list_messages: NEW tool (was teams_get_chat_messages)
- teams_create_chat: MISSING from prompt
- teams_create_meeting: IN PROMPT but DOESN'T EXIST in code
- teams_list_channels: parameter changed (team_name → team_id)
```

### 3. Update Prompt (30 minutes)

**Location: Line 1379+ (MS365-Teams section)**

**Before:**
```markdown
### Tools (7)

**1. teams_send_message(...)**
**2. teams_list_chats(...)**
**3. teams_get_chat_messages(...)** ← Wrong name!
...
**7. teams_create_meeting(...)** ← Doesn't exist!
```

**After:**
```markdown
### Tools (8)

**1. teams_send_message(...)**
**2. teams_list_chats(...)**
**3. teams_create_chat(...)** ← Added!
**4. teams_get_chat(...)**
**5. teams_list_messages(...)** ← Correct name!
...
**8. teams_send_channel_message(...)** ← Correct count!
```

**Key Sections to Update:**

1. **Tools section** (line ~1381)
   - Tool count: `### Tools (7)` → `### Tools (8)`
   - Tool signatures with correct names/parameters

2. **Critical Notes** (line ~1422)
   - Add notes about new tools
   - Update usage patterns

3. **Workflows** (line ~538+)
   - Update workflows that use changed tools
   - Example: Workflow 7 uses teams_list_channels(team_id) now

4. **Quick Decision Table** (line ~1821)
   - Add new tools to table
   - Example: `| "Read Teams conversation" | MS365-Teams | teams_list_messages |`

5. **Examples** (line ~423+)
   - Update code examples if tools changed
   - Ensure consistency

### 4. Verify Changes (10 minutes)

**Checklist:**
```bash
# 1. Count tools in each service
grep "### Tools (" ORCHESTRATOR_PROMPT_NEW.md

# 2. Check for old tool names
grep "teams_get_chat_messages\|teams_create_meeting" ORCHESTRATOR_PROMPT_NEW.md
# Should return nothing if fixed!

# 3. Verify workflows use correct parameters
grep "teams_list_channels" ORCHESTRATOR_PROMPT_NEW.md
# All instances should use team_id, not team_name

# 4. Check consistency
grep -c "teams_list_messages" ORCHESTRATOR_PROMPT_NEW.md
# Should appear in: tools section, workflows, quick table, patterns
```

### 5. Document Changes (10 minutes)

**Update CHANGELOG.md:**
```markdown
## [1.2.0] - 2025-02-15

### Added - MS365-Teams
- `teams_search_messages` - Search across all chats by keyword
- `teams_pin_message` - Pin important messages in chat

### Changed
- `teams_list_messages` now supports `search_query` parameter
- `teams_send_message` now returns `message_id` in response

### Fixed
- Workflow 12: Corrected batch send to use new parameters
```

### 6. Create Dated Version (2 minutes)

```bash
# Get today's date
TODAY=$(date +%Y-%m-%d)

# Copy current version to archive
mkdir -p prompts/archive
mv ORCHESTRATOR_PROMPT_*.md prompts/archive/ 2>/dev/null || true

# Create new dated version
cp ORCHESTRATOR_PROMPT_NEW.md ORCHESTRATOR_PROMPT_${TODAY}.md

# Verify
ls -lh ORCHESTRATOR_PROMPT_*.md
```

### 7. Commit & Push (5 minutes)

```bash
# Stage changes
git add ORCHESTRATOR_PROMPT_*.md CHANGELOG.md prompts/

# Commit with descriptive message
git commit -m "feat: Update MS365-Teams tools (7→8) - Add teams_list_messages

- Added teams_list_messages for conversation history
- Added teams_create_chat and teams_get_chat (missing tools)
- Removed teams_create_meeting (doesn't exist in code)
- Updated teams_list_channels to use team_id parameter
- Updated Workflow 5.5, 7 with corrected parameters
- Updated quick decision table and cross-service patterns

Breaking Changes:
- teams_list_channels(team_name) → teams_list_channels(team_id)
- teams_send_channel_message parameters changed

Tools: 54 total (8 in MS365-Teams)"

# Push to GitHub
git push origin main
```

## 🔍 Finding What to Update

### Method 1: Compare with MCP Health Endpoint

```bash
# Get current tools from production
curl -s https://ms365-teams.brainaihub.tech/health | jq '.tools'
# Output: 8

# Compare with prompt
grep "### Tools (" ORCHESTRATOR_PROMPT_*.md | grep "Teams"
# Output: ### Tools (7)

# → UPDATE NEEDED!
```

### Method 2: Check MCP Server Source Code

```bash
# Navigate to MCP repo
cd /path/to/ms365-teams

# List all @mcp.tool() decorators
grep -n "@mcp.tool()" src/main.py

# Output:
# 123:@mcp.tool()
# 145:@mcp.tool()
# ...
# Count them manually or:
grep -c "@mcp.tool()" src/main.py
# Output: 8
```

### Method 3: Use MCP Inspector

```
1. Connect to MCP server via Inspector
2. Check tools list
3. Compare with prompt documentation
4. Note differences
```

## 📋 Update Template

Use this template for quick updates:

```markdown
# Update Checklist - [Date]

## MCP Server: [service-name]
- [ ] Current tools: ___ (from code)
- [ ] Prompt shows: ___ (from prompt)
- [ ] Match: YES / NO

## Changes Needed
- [ ] Tool added: [name]
- [ ] Tool removed: [name]
- [ ] Tool renamed: [old] → [new]
- [ ] Parameter changed: [tool].[param] ([old] → [new])

## Prompt Sections to Update
- [ ] Line ~XXXX: Tools list
- [ ] Line ~XXXX: Critical Notes
- [ ] Line ~XXXX: Workflow [N]
- [ ] Line ~XXXX: Quick Decision Table
- [ ] Line ~XXXX: Examples

## Verification
- [ ] Tool count correct
- [ ] No references to old tool names
- [ ] Workflows use correct parameters
- [ ] Examples updated
- [ ] CHANGELOG.md updated

## Files Changed
- [ ] ORCHESTRATOR_PROMPT_YYYY-MM-DD.md (new dated file)
- [ ] CHANGELOG.md
- [ ] Old prompt moved to prompts/archive/
```

## 🐛 Common Issues

### Issue: Can't find where tool is referenced

**Solution:**
```bash
# Search entire prompt for tool name
grep -n "teams_send_message" ORCHESTRATOR_PROMPT_*.md

# Output shows all line numbers:
# 534:   - teams_send_message(st, colleague_name, "See attached brief", document_url)
# 1383:**1. teams_send_message(session_token, recipient, message, attachment_url?)**
# 1564:3. teams_send_message(st, "Mario Rossi", "See attached", pdf_url)
# 1828:| "Send Teams message" | MS365-Teams | teams_send_message |
```

### Issue: Forgot to update workflow

**Solution:**
```bash
# Find all workflows that use the changed tool
grep -B2 -A10 "teams_list_channels" ORCHESTRATOR_PROMPT_*.md

# Review each workflow and update parameters
```

### Issue: Tool count doesn't match

**Solution:**
```bash
# List all tools in section
sed -n '/^## 💬 MS365-Teams/,/^---$/p' ORCHESTRATOR_PROMPT_*.md | grep "^\*\*[0-9]"

# Count them:
sed -n '/^## 💬 MS365-Teams/,/^---$/p' ORCHESTRATOR_PROMPT_*.md | grep -c "^\*\*[0-9]"
```

## 🎯 Best Practices

1. **Always check MCP server code** - Prompt should match reality
2. **Update all references** - Use grep to find every mention
3. **Test workflows** - Ensure examples use correct parameters
4. **Document breaking changes** - Note in CHANGELOG if parameters changed
5. **Keep history** - Archive old versions in prompts/archive/
6. **Date everything** - Use YYYY-MM-DD format in filenames

## 📚 Resources

- [MCP Specification](https://modelcontextprotocol.io/)
- [MS365-Teams MCP Server](https://github.com/ilvolodel/ms365-teams)
- [MCP Inspector Tool](https://inspector.modelcontextprotocol.io/)

## 🆘 Need Help?

If you're stuck:
1. Check `CHANGELOG.md` for examples of past updates
2. Look at git history: `git log --oneline ORCHESTRATOR_PROMPT_*.md`
3. Compare old vs new versions: `git diff prompts/archive/ORCHESTRATOR_PROMPT_2025-02-01.md ORCHESTRATOR_PROMPT_2025-02-04.md`

---

**Time estimate for typical update:** ~1 hour  
**Time estimate with this guide:** ~15 minutes  

Happy updating! 🚀
