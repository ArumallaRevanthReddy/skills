---
name: add-mcp-tool
description: Intelligently adds new tools to existing MCP servers. Discovers available MCP servers, analyzes the tool requirement, selects the most appropriate server, implements the tool, and commits changes. Use when the user asks to add a tool, create a new MCP function, or extend MCP server capabilities.
allowed-tools: Bash, Read, Write, Edit, Grep, Glob, AskUserQuestion
---

# Add MCP Tool Skill

Automatically adds new tools to the most appropriate existing MCP server by analyzing requirements and implementing best practices.

## Workflow Overview

When a user requests a new MCP tool, this skill:

1. **Discovers** available MCP servers using `claude mcp list`
2. **Analyzes** the tool requirement to understand its purpose
3. **Selects** the most appropriate MCP server for the tool
4. **Implements** the tool with proper error handling and validation
5. **Tests** the implementation (if requested)
6. **Commits** and pushes the changes to git

## Step-by-Step Instructions

### Step 1: Discover Available MCP Servers

Run the command to list all configured MCP servers:

```bash
claude mcp list
```

Parse the output to identify:
- Server names
- Server purposes/descriptions (if available)
- Transport types (stdio, http, sse)
- Command paths or URLs

**Expected output format:**
```
MCP Servers:
- weather: Weather data and forecasts (stdio)
- database: PostgreSQL query tool (stdio)
- github: GitHub API integration (http)
```

### Step 2: Analyze Tool Requirement

When the user describes what tool they want, extract:

- **Tool purpose**: What the tool does
- **Input parameters**: What data it needs
- **Output format**: What it returns
- **External dependencies**: APIs, databases, services it uses
- **Domain category**: Weather, database, files, communication, etc.

**Example:**
```
User: "Add a tool to fetch the latest commits from a repository"

Analysis:
- Purpose: Fetch git commits
- Inputs: repository name/URL, limit (optional)
- Output: List of commits with metadata
- Dependencies: Git/GitHub API
- Domain: Version control / code repository
```

### Step 3: Select Appropriate MCP Server

Match the tool to a server using these criteria (in order of priority):

1. **Domain Match**: Does the tool's domain match the server's purpose?
   - Git tool → github server
   - Database query → database server
   - Weather data → weather server

2. **Technology Stack**: Does the tool use the same technologies?
   - Python tool → Python-based server
   - TypeScript tool → TypeScript-based server

3. **Dependency Overlap**: Does it use similar APIs or libraries?
   - GitHub API tool → server already using GitHub API

4. **Server Size**: Prefer servers with related functionality over generic servers

**Decision Logic:**
```
IF exact domain match EXISTS:
    SELECT that server
ELSE IF similar domain EXISTS:
    ASK USER to confirm or create new server
ELSE:
    ASK USER which server to use or create new one
```

**Use AskUserQuestion if:**
- Multiple servers could work equally well
- No clear domain match exists
- The tool represents a new domain category

### Step 4: Access Server Codebase

Once a server is selected, determine its location:

```bash
# Get server configuration details
claude mcp list | grep -A 5 "server_name"
```

The configuration will show the command path for stdio servers:
```
command: python /path/to/server.py
```

**For git repositories:**

1. Extract the repository path from the command
2. Check if it's already cloned locally
3. If not, ask user for the git repository URL
4. Clone if needed: `git clone <repo_url>`
5. Navigate to the server directory

**Example:**
```bash
# Check if server directory exists
ls /path/to/server

# If it's a git repo, check status
cd /path/to/server && git status

# If not cloned, ask user for repo URL and clone
git clone https://github.com/user/mcp-server.git
```

### Step 5: Analyze Server Structure

Read the server files to understand:

1. **Language**: Python or TypeScript?
2. **Framework**: FastMCP, MCP SDK, custom?
3. **Existing tools**: What tools are already defined?
4. **File structure**: Single file or multiple modules?
5. **Import patterns**: How are dependencies imported?
6. **Coding style**: Naming conventions, formatting

**Commands to run:**
```bash
# Find main server file
ls *.py *.ts src/index.ts

# Read main file
cat server.py  # or src/index.ts

# Check for existing tools
grep -n "@mcp.tool\|registerTool" server.py
```

### Step 6: Implement the Tool

Based on the server's language and framework, implement the tool following existing patterns.

#### Python (FastMCP) Implementation

Add the tool using the `@mcp.tool()` decorator:

```python
@mcp.tool()
async def tool_name(param1: str, param2: int = 0) -> str:
    """
    Brief description of what the tool does.

    Args:
        param1: Description of first parameter
        param2: Description of second parameter (optional, default: 0)

    Returns:
        Description of return value
    """
    import logging
    logger = logging.getLogger(__name__)

    try:
        # Input validation
        if not param1:
            return "Error: param1 is required"

        if param2 < 0:
            return "Error: param2 must be non-negative"

        # Tool logic here
        logger.info(f"Processing {param1} with {param2}")

        # Example: Make API call
        async with httpx.AsyncClient(timeout=10.0) as client:
            response = await client.get(f"https://api.example.com/{param1}")

            if response.status_code != 200:
                return f"Error: API returned status {response.status_code}"

            data = response.json()

        # Format and return result
        return f"Result: {data}"

    except asyncio.TimeoutError:
        return "Error: Request timed out"
    except Exception as e:
        logger.error(f"Error in tool_name: {e}")
        return f"Error: Unable to process request"
```

#### TypeScript (MCP SDK) Implementation

Add the tool using `server.registerTool`:

```typescript
server.registerTool(
  "tool_name",
  {
    description: "Brief description of what the tool does",
    inputSchema: {
      param1: z.string().describe("Description of first parameter"),
      param2: z.number().int().nonnegative().default(0).describe("Description of second parameter"),
    },
  },
  async ({ param1, param2 }) => {
    try {
      // Input validation
      if (!param1) {
        return {
          content: [{
            type: "text",
            text: "Error: param1 is required",
          }],
          isError: true,
        };
      }

      // Tool logic here
      console.error(`Processing ${param1} with ${param2}`);

      // Example: Make API call
      const response = await axios.get(
        `https://api.example.com/${param1}`,
        { timeout: 10000 }
      );

      if (response.status !== 200) {
        return {
          content: [{
            type: "text",
            text: `Error: API returned status ${response.status}`,
          }],
          isError: true,
        };
      }

      const data = response.data;

      // Format and return result
      return {
        content: [{
          type: "text",
          text: `Result: ${JSON.stringify(data)}`,
        }],
      };
    } catch (error) {
      console.error(`Error in tool_name: ${error}`);
      return {
        content: [{
          type: "text",
          text: "Error: Unable to process request",
        }],
        isError: true,
      };
    }
  },
);
```

### Step 7: Add Required Dependencies

Check if new dependencies are needed:

**Python:**
```bash
# Check existing dependencies
cat pyproject.toml

# Add new dependency if needed
echo 'httpx = "^0.24.0"' >> pyproject.toml
```

**TypeScript:**
```bash
# Check existing dependencies
cat package.json

# Add new dependency if needed
npm install axios
# or
yarn add axios
```

### Step 8: Validate Implementation

Before committing, verify:

- [ ] Tool name follows naming conventions (snake_case for Python, camelCase for TypeScript)
- [ ] All parameters have type hints and descriptions
- [ ] Required parameters are clearly marked
- [ ] Input validation is present
- [ ] Error handling returns proper error messages
- [ ] No logging to stdout (use stderr or logging library)
- [ ] Timeouts are set for external API calls
- [ ] Tool follows existing code style in the server
- [ ] Imports are added if needed

### Step 9: Test the Implementation (Optional)

If the user requests testing:

**Python:**
```bash
# Test import
python -c "from server import mcp; print(mcp.tools.keys())"

# Run server to check for syntax errors
python server.py --help
```

**TypeScript:**
```bash
# Build the project
npm run build

# Check for compilation errors
tsc --noEmit
```

**Use MCP Inspector (if available):**
```bash
mcp-inspector -- python server.py
# or
mcp-inspector -- node dist/index.js
```

### Step 10: Commit and Push Changes

Create a meaningful commit message:

```bash
cd /path/to/server

# Check changes
git status
git diff

# Add files
git add server.py  # or appropriate files

# Commit with descriptive message
git commit -m "Add <tool_name> tool

Implement <tool_name> to <brief description of what it does>.

Features:
- <feature 1>
- <feature 2>

Parameters:
- param1: <description>
- param2: <description>"

# Push to remote
git push
```

## Best Practices

### Tool Naming
- Use descriptive, action-oriented names: `get_weather`, `search_files`, `validate_email`
- Follow language conventions: snake_case (Python) or camelCase (TypeScript)
- Avoid generic names: `process`, `handle`, `data`

### Documentation
- Write clear docstrings/descriptions
- Document all parameters
- Specify return value format
- Include example usage if complex

### Error Handling
- Validate all inputs before processing
- Return user-friendly error messages
- Never expose internal errors or stack traces
- Set `isError: true` for TypeScript error responses

### Security
- Sanitize all inputs
- Never log sensitive data
- Validate external API responses
- Use timeouts for all network calls
- Avoid SQL injection, XSS, command injection

### Performance
- Set reasonable timeouts (5-30 seconds)
- Implement rate limiting if needed
- Cache results when appropriate
- Avoid blocking operations

## Example Workflows

### Example 1: Add Weather Forecast Tool

**User Request:** "Add a tool to get a 7-day weather forecast"

**Workflow:**
1. Run `claude mcp list` → Find "weather" server
2. Analyze: forecast tool, needs location input, returns 7-day data
3. Select: weather server (exact domain match)
4. Access: Check server at `/path/to/weather-server`
5. Read: Python FastMCP server with existing `get_weather` tool
6. Implement: Add `get_forecast` tool following existing pattern
7. Add dependency: httpx already present
8. Validate: Check syntax, error handling
9. Commit: "Add get_forecast tool for 7-day weather predictions"
10. Push: `git push`

### Example 2: Add Database Export Tool

**User Request:** "Add a tool to export query results to CSV"

**Workflow:**
1. Run `claude mcp list` → Find "database" server
2. Analyze: export tool, needs query + filename, returns CSV file
3. Select: database server (exact domain match)
4. Access: Check server at `/path/to/db-server`
5. Read: Python server with database connection pool
6. Implement: Add `export_to_csv` tool with csv module
7. Add dependency: csv (standard library, no install needed)
8. Validate: Test CSV generation logic
9. Commit: "Add export_to_csv tool for query result export"
10. Push: `git push`

### Example 3: Ambiguous Tool Placement

**User Request:** "Add a tool to send Slack notifications when database errors occur"

**Analysis:**
- Could fit: "slack" server OR "database" server
- Involves: Both Slack API and database monitoring

**Resolution:**
1. List both options to user via AskUserQuestion:
   - Option 1: Add to slack server (notification-focused)
   - Option 2: Add to database server (database-focused)
   - Option 3: Create new "monitoring" server
2. User selects: slack server
3. Proceed with implementation in slack server
4. Add database connection as dependency

## Troubleshooting

### Server Not Found
- Check `claude mcp list` output
- Verify server is configured correctly
- Ask user for server name or create new one

### Cannot Access Repository
- Ask user for git repository URL
- Check if repository is private (needs authentication)
- Verify user has write permissions

### Tool Conflicts
- Check if tool name already exists
- Rename to avoid conflicts: `get_weather_v2` or `get_detailed_weather`

### Build/Syntax Errors
- Read error messages carefully
- Check import statements
- Verify type hints (Python) or Zod schemas (TypeScript)
- Follow existing code patterns in the file

### Commit Fails
- Check if git credentials are configured
- Verify user has push permissions
- Check if branch is protected

## Quick Reference

### Command Checklist
```bash
# 1. Discover servers
claude mcp list

# 2. Navigate to server
cd /path/to/server

# 3. Check git status
git status

# 4. Read main file
cat server.py  # or src/index.ts

# 5. After implementing, check changes
git diff

# 6. Add and commit
git add .
git commit -m "Add <tool_name> tool"

# 7. Push
git push
```

### Decision Matrix

| Tool Domain | Best Server Match |
|-------------|------------------|
| Weather data | weather |
| Database queries | database |
| File operations | filesystem |
| Git/GitHub | github |
| Communication | slack, email |
| Cloud services | aws, gcp, azure |
| Generic utility | utils, tools |

## Summary

This skill automates the process of adding tools to MCP servers by:
- Intelligently selecting the right server
- Following the server's existing code patterns
- Implementing best practices for error handling and validation
- Managing git workflow automatically
- Ensuring consistency across tools

Use this skill whenever you need to extend MCP server capabilities with new tools.
