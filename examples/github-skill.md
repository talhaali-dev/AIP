---
name: github-integration
description: Use this capability when working with GitHub repositories, issues, pull requests, or code search
triggers:
  - User mentions GitHub, repos, issues, or PRs
  - User wants to create/search/manage GitHub resources
  - User asks about code in repositories
capability_id: com.github.mcp-server
protocol: mcp
---

# GitHub Integration Skill

## When to Use This Capability

Use the GitHub MCP server when the user wants to:
- Search for repositories by keywords, topics, or language
- Create, update, or search issues
- List or analyze pull requests
- Search for code across repositories
- Get repository information and statistics

## Available MCP Tools

The GitHub MCP server provides the following tools via the Model Context Protocol. Use MCP's native `tools/call` to invoke them.

### search_repositories

**When to use:** User asks to find repositories

**MCP Tool Schema:**
```json
{
  "name": "search_repositories",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Search query (supports GitHub search syntax)"
      },
      "sort": {
        "type": "string",
        "enum": ["stars", "forks", "updated", "created"]
      },
      "order": {
        "type": "string",
        "enum": ["asc", "desc"]
      },
      "per_page": {
        "type": "integer",
        "minimum": 1,
        "maximum": 100
      }
    },
    "required": ["query"]
  }
}
```

**Best practices:**
- Use GitHub search syntax for precise results
  - `language:python` - Filter by programming language
  - `stars:>1000` - Filter by star count
  - `topic:machine-learning` - Filter by topic
  - `in:name,description` - Search in specific fields
- Sort by `stars` for popular repositories
- Sort by `updated` for actively maintained projects
- Limit results to top 5-10 for user readability

**Example interaction:**

User: "Find popular TypeScript testing libraries"

Agent process:
1. Call MCP tool: `search_repositories`
2. Query: `"testing language:typescript"`
3. Sort: `"stars"`, Order: `"desc"`
4. Limit: 10
5. Format results with descriptions and star counts

### create_issue

**When to use:** User wants to report a bug, request a feature, or create a task

**MCP Tool Schema:**
```json
{
  "name": "create_issue",
  "inputSchema": {
    "type": "object",
    "properties": {
      "repo": {
        "type": "string",
        "description": "Repository in format 'owner/name'"
      },
      "title": {
        "type": "string",
        "description": "Issue title"
      },
      "body": {
        "type": "string",
        "description": "Issue description (supports Markdown)"
      },
      "labels": {
        "type": "array",
        "items": {"type": "string"},
        "description": "Label names"
      },
      "assignees": {
        "type": "array",
        "items": {"type": "string"},
        "description": "GitHub usernames"
      }
    },
    "required": ["repo", "title"]
  }
}
```

**Best practices:**
- **Always search for duplicates first** - Use `search_code` or check existing issues
- Include clear reproduction steps for bugs
- Use descriptive, actionable titles
- Add relevant labels: `bug`, `enhancement`, `documentation`, etc.
- For bugs, include:
  - What happened (actual behavior)
  - What should happen (expected behavior)
  - Steps to reproduce
  - Environment (OS, version, etc.)

**Example interaction:**

User: "The login form doesn't work on Safari mobile"

Agent process:
1. Identify repository from context (or ask user)
2. Search existing issues for duplicates
3. No duplicates found
4. Call MCP tool: `create_issue`
5. Params:
   ```json
   {
     "repo": "company/webapp",
     "title": "Login form non-functional on Safari mobile",
     "body": "## Description\nUsers report that the login form is unresponsive on Safari iOS.\n\n## Steps to Reproduce\n1. Open site on iPhone with Safari\n2. Navigate to login page\n3. Tap email field\n\n## Expected Behavior\nKeyboard should appear and field should focus\n\n## Actual Behavior\nNo response to taps\n\n## Environment\n- Device: iPhone 14\n- OS: iOS 17.2\n- Browser: Safari mobile",
     "labels": ["bug", "mobile", "priority:high"]
   }
   ```
6. Respond with issue URL

### get_pull_requests

**When to use:** User wants to review, list, or analyze pull requests

**MCP Tool Schema:**
```json
{
  "name": "get_pull_requests",
  "inputSchema": {
    "type": "object",
    "properties": {
      "repo": {
        "type": "string",
        "description": "Repository in format 'owner/name'"
      },
      "state": {
        "type": "string",
        "enum": ["open", "closed", "all"]
      },
      "per_page": {
        "type": "integer"
      }
    },
    "required": ["repo"]
  }
}
```

**Best practices:**
- Default to `state: "open"` unless user specifies otherwise
- For code review workflows, fetch PR details including:
  - Review status
  - CI/CD status
  - Number of comments
  - Files changed
- Batch multiple PR checks when possible
- Highlight PRs that need attention (failed CI, pending reviews)

**Example interaction:**

User: "What PRs need my review?"

Agent process:
1. Call MCP tool: `get_pull_requests`
2. Params: `{"repo": "company/webapp", "state": "open"}`
3. Filter results for PRs requesting user's review
4. Format with status indicators

### search_code

**When to use:** User wants to find specific code patterns or functions

**MCP Tool Schema:**
```json
{
  "name": "search_code",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Code search query"
      },
      "repo": {
        "type": "string",
        "description": "Optional: limit to specific repo"
      },
      "language": {
        "type": "string",
        "description": "Programming language filter"
      }
    },
    "required": ["query"]
  }
}
```

**Best practices:**
- Use specific search terms (function names, class names)
- Include language filter when known
- Limit to specific repo when appropriate
- Search supports regex and exact phrases (in quotes)

**Example interaction:**

User: "Find all usages of the `authenticate()` function"

Agent process:
1. Call MCP tool: `search_code`
2. Query: `"authenticate() language:javascript"`
3. Repo: Current repo from context
4. Display results with file paths and line numbers

### get_repository

**When to use:** User asks about a specific repository's details

**MCP Tool Schema:**
```json
{
  "name": "get_repository",
  "inputSchema": {
    "type": "object",
    "properties": {
      "repo": {
        "type": "string",
        "description": "Repository in format 'owner/name'"
      }
    },
    "required": ["repo"]
  }
}
```

**Returns:** Repository metadata including stars, forks, language, description, topics, license

**Example interaction:**

User: "Tell me about the React repository"

Agent process:
1. Call MCP tool: `get_repository`
2. Params: `{"repo": "facebook/react"}`
3. Summarize key info for user

## Common Workflows

### Workflow 1: Creating a Bug Report

```
User: "There's a bug - the export button crashes the app"

Steps:
1. Identify repository from context
2. Call search_code to find export button code (optional)
3. Call search_repositories or check issues for duplicates
4. Call create_issue with:
   - Clear title: "Export button causes application crash"
   - Detailed body with reproduction steps
   - Labels: ["bug", "priority:high"]
5. Return issue URL and number to user
```

### Workflow 2: Finding Libraries

```
User: "What are some good Python data visualization libraries?"

Steps:
1. Call search_repositories
2. Query: "data visualization language:python"
3. Sort by stars (descending)
4. Limit to top 5
5. Present results with:
   - Repository name and description
   - Star count
   - Last updated date
   - Primary use cases
```

### Workflow 3: Code Review Status

```
User: "What's the status of open PRs?"

Steps:
1. Call get_pull_requests with state="open"
2. For each PR, note:
   - Review approvals
   - CI/CD status
   - Number of comments
   - Days open
3. Categorize:
   - Ready to merge (approved + passing CI)
   - Needs review
   - Needs changes (requested changes)
   - Failing CI
4. Present summary
```

### Workflow 4: Code Search and Analysis

```
User: "How is the authentication function implemented?"

Steps:
1. Call search_code
2. Query: "function authenticate" or "def authenticate"
3. Find relevant matches
4. Extract code snippets
5. Explain implementation to user
```

## Tips and Best Practices

### Search Queries
- Use GitHub's search syntax for precision
- Combine filters: `language:python stars:>1000 topic:machine-learning`
- Quote exact phrases: `"neural network"`
- Use wildcards for function searches: `authenticate*`

### Issue Creation
- **Always search first** - Avoid duplicates
- Use descriptive titles (not "Bug" but "Login form broken on mobile Safari")
- Include reproduction steps numbered 1, 2, 3
- Add labels for categorization and priority
- Link related issues/PRs when relevant

### Pull Request Management
- Check both "open" and "draft" states
- Look for CI/CD status before reviewing
- Highlight PRs that are blocked or stale
- Suggest batching reviews for efficiency

### Rate Limiting
- GitHub API has rate limits (5000/hour for authenticated)
- Batch operations when possible
- Cache results for repeated queries
- If rate limited, inform user and suggest waiting

## Error Handling

### Common Errors

**Authentication Failed**
- Check that github_token is valid and not expired
- Verify token has required scopes (repo, user)
- Suggest regenerating token if necessary

**Repository Not Found**
- Verify repo name format: "owner/name"
- Check user has access to private repos
- Suggest searching public repos if unsure

**Rate Limited**
- Inform user of rate limit
- Show reset time
- Suggest reducing query scope or waiting

**Permission Denied**
- User lacks permissions for operation
- Suggest contacting repo maintainer
- Explain required permission level

## Integration with Other Tools

The GitHub capability works well with:
- **Code editors** - Create issues from error messages
- **CI/CD tools** - Monitor PR status
- **Project management** - Sync issues with other systems
- **Documentation** - Link to relevant code examples

## Security Considerations

- GitHub token is stored securely in system keychain
- Token should have minimum required scopes
- Never log or display token in plain text
- Recommend using fine-grained personal access tokens (PATs)

## Example Conversations

### Example 1: Repository Discovery

**User:** "I need a good Python web framework"

**Agent:** I'll search for popular Python web frameworks.

[Calls MCP: search_repositories("web framework language:python", sort="stars")]

Here are the top Python web frameworks:

1. **Django** (70k stars) - High-level framework for rapid development
2. **Flask** (65k stars) - Lightweight and flexible micro-framework
3. **FastAPI** (60k stars) - Modern, fast framework with automatic API docs

Would you like details about any of these?

### Example 2: Bug Report

**User:** "Create a bug report - the dashboard doesn't load on Firefox"

**Agent:** I'll create an issue in the repository.

[Calls MCP: search_code to check if similar issue exists]
[No duplicates found]
[Calls MCP: create_issue]

Created issue #247: "Dashboard fails to load on Firefox"
https://github.com/company/webapp/issues/247

I've added the "bug" and "browser-compatibility" labels. Would you like me to add any additional information to the issue?

### Example 3: PR Review

**User:** "What PRs are waiting for review?"

**Agent:** Let me check the open pull requests.

[Calls MCP: get_pull_requests(state="open")]

You have 3 PRs awaiting review:

1. **PR #145**: Add dark mode (2 days old)
   - Status: ✅ CI passing
   - Needs: 1 approval

2. **PR #142**: Fix memory leak (5 days old)
   - Status: ⚠️ CI failing
   - Needs: Changes requested

3. **PR #138**: Update dependencies (1 week old)
   - Status: ✅ Ready to merge (2 approvals)

Would you like me to provide details on any of these?

---

## Quick Reference

| User Intent | MCP Tool | Key Parameters |
|-------------|----------|----------------|
| Find repositories | search_repositories | query, sort, order |
| Create issue | create_issue | repo, title, body, labels |
| List PRs | get_pull_requests | repo, state |
| Search code | search_code | query, repo, language |
| Get repo info | get_repository | repo |

Remember: Always use the native MCP protocol to invoke these tools. This skill provides context on WHEN and HOW to use them, but the actual invocation happens via MCP.
