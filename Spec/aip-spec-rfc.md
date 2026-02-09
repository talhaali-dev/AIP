# Agent Interoperability Protocol (AIP)
## RFC Draft v0.1.0

**Status:** Draft  
**Authors:** Turbo Launch  
**Created:** February 2026  
**Last Modified:** February 2026

---

## Abstract

The Agent Interoperability Protocol (AIP) defines a standardized way for AI agents to discover, install, configure, and utilize external capabilities at runtime. AIP provides:

1. **Manifest format** - Machine-readable installation and configuration instructions
2. **Skill format** - Optional Markdown guides (similar to Anthropic Skills) that help agents understand capabilities

**AIP does NOT introduce new tool invocation mechanisms.** Agents use native protocols (MCP, CLI, HTTP) to invoke tools. AIP focuses on discovery, installation, configuration, and providing intelligence/context to agents.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Terminology](#2-terminology)
3. [Architecture Overview](#3-architecture-overview)
4. [Capability Manifest Specification](#4-capability-manifest-specification)
5. [Installation Protocol](#5-installation-protocol)
6. [Configuration Protocol](#6-configuration-protocol)
7. [Skill Format](#7-skill-format)
8. [Tool Invocation](#8-tool-invocation)
9. [Permission Model](#9-permission-model)
10. [Discovery Protocol](#10-discovery-protocol)
11. [Lifecycle Management](#11-lifecycle-management)
12. [Platform Integration](#12-platform-integration)
13. [Security Considerations](#13-security-considerations)
14. [References](#14-references)

---

## 1. Introduction

### 1.1 Motivation

AI agents need to integrate with external tools (MCP servers, CLI utilities, APIs). Today this requires:

- Manual installation of each tool
- Hardcoded configuration
- No standardized discovery
- Agents lack context on how to use tools effectively

**AIP solves this by providing:**
- Automated installation from manifests
- Standardized configuration management
- Searchable capability registries
- Optional skills that teach agents how to use capabilities

### 1.2 Design Philosophy

**What AIP Does:**
✅ Automates installation and configuration  
✅ Enables discovery of capabilities  
✅ Provides context via skills (optional)  
✅ Manages permissions and security  

**What AIP Does NOT Do:**
❌ Replace MCP, CLI, or HTTP protocols  
❌ Introduce new tool invocation mechanisms  
❌ Require agents to use a specific framework  

**Analogy:** AIP is like package.json (metadata, dependencies) + README.md (how to use) for AI agent capabilities.

### 1.3 Relationship to MCP

AIP **complements** Model Context Protocol:

- **MCP:** Defines how agents invoke tools (protocol, transport, tool schemas)
- **AIP:** Defines how agents discover, install, and understand MCP servers

```
┌─────────────────────┐
│   AIP Manifest      │  ← Installation & configuration
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   MCP Server        │  ← Tool invocation protocol
│  (running server)   │
└─────────────────────┘
           │
           ▼
┌─────────────────────┐
│  Agent calls        │  ← Uses MCP protocol
│  MCP tools          │
└─────────────────────┘
```

---

## 2. Terminology

### Core Concepts

**Agent**  
An AI system capable of reasoning and executing tasks (e.g., Claude Code, Claude Cowork, custom agents).

**Capability**  
An external tool, service, or resource that extends agent functionality.  
Examples: GitHub MCP server, file search CLI tool, weather API.

**Manifest**  
A JSON/YAML file that describes:
- How to install the capability
- What configuration it needs  
- What tools it provides
- What permissions it requires

Think of it like `package.json` + installation instructions.

**Skill**  
An optional Markdown guide (similar to Anthropic Skills in `/mnt/skills/`) that helps agents understand:
- When to use the capability
- How to use the tools effectively
- Best practices and common patterns
- Example workflows

Think of it like a `README.md` or tutorial for agents.

**Tool**  
An individual function/operation that a capability provides:
- For MCP servers: MCP tools (defined by MCP protocol)
- For CLI: Command-line commands
- For APIs: HTTP endpoints

**Provider**  
The developer or organization offering a capability.

**Registry**  
A searchable index of capabilities (optional central directory).

### Key Distinctions

**Manifest vs Skill:**
| Manifest | Skill |
|----------|-------|
| JSON/YAML format | Markdown format |
| Installation instructions | Usage guide |
| Configuration parameters | Best practices |
| Tool descriptions | Example workflows |
| Machine-readable | Human/AI-readable |
| Required | Optional |

**AIP Skill vs Anthropic Skill:**
- Both are Markdown guides for agents
- Both help agents understand how to do something
- AIP skills are for external capabilities
- Anthropic skills are built into Claude (docx, pptx, etc.)
- Same concept, different scope

---

## 3. Architecture Overview

### 3.1 System Components

```
┌─────────────────────────────────────────────────────────────┐
│                         Agent                                │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Discovery   │  │ Installation │  │    Skill     │      │
│  │   Engine     │  │    Engine    │  │   Loader     │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                  │                  │              │
│         └──────────────────┴──────────────────┘              │
│                            │                                 │
│                    AIP Protocol Layer                        │
└────────────────────────────┼─────────────────────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ▼                   ▼                   ▼
  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
  │   MCP       │    │     CLI     │    │   HTTP      │
  │   Server    │    │     Tool    │    │   API       │
  └─────────────┘    └─────────────┘    └─────────────┘
         │                   │                   │
         │                   │                   │
         └───────────────────┴───────────────────┘
                             │
                    Native Protocols
              (MCP, shell commands, HTTP)
```

### 3.2 Workflow

```
1. DISCOVER
   Agent: "I need GitHub integration"
   → Searches registry for "github" capabilities
   → Finds "GitHub MCP Server"

2. INSTALL
   → Downloads manifest
   → Executes: npm install -g @github/mcp-server
   → Validates installation

3. CONFIGURE
   → Prompts user: "Enter GitHub token"
   → Stores securely in keychain
   → Writes config file

4. LOAD SKILL (optional)
   → Downloads skill.md
   → Reads: "Use search_repositories when user asks to find repos"
   → Agent now understands when/how to use tools

5. INVOKE TOOLS
   → Agent uses MCP protocol
   → Calls: search_repositories(query="python ML")
   → No AIP involvement - pure MCP
```

### 3.3 Capability Types

AIP supports three types, each using native protocols:

| Type | Protocol | Example |
|------|----------|---------|
| MCP Server | Model Context Protocol | GitHub integration, Slack bot |
| CLI Tool | Shell commands | File search, git wrapper |
| HTTP API | REST/GraphQL | Weather API, translation service |

---

## 4. Capability Manifest Specification

### 4.1 Complete Example

```json
{
  "aip_version": "0.1.0",
  
  "capability": {
    "id": "com.github.mcp-server",
    "name": "GitHub MCP Server",
    "version": "2.1.0",
    "description": "MCP server providing GitHub API integration for repositories, issues, and pull requests",
    "type": "mcp_server",
    "provider": {
      "name": "GitHub, Inc.",
      "url": "https://github.com",
      "contact": "support@github.com"
    },
    "homepage": "https://github.com/github/mcp-server",
    "repository": "https://github.com/github/mcp-server",
    "license": "MIT"
  },
  
  "installation": {
    "platforms": ["linux", "macos", "windows"],
    "requirements": {
      "node": ">=20.0.0"
    },
    "steps": [
      {
        "type": "npm_install",
        "description": "Install GitHub MCP server package",
        "package": "@github/mcp-server",
        "version": "^2.1.0",
        "global": true,
        "validation": {
          "command": "github-mcp-server --version",
          "expected_exit_code": 0
        }
      }
    ],
    "post_install": {
      "message": "GitHub MCP server installed. Configure your GitHub token to get started.",
      "configuration_required": true
    }
  },
  
  "configuration": {
    "parameters": [
      {
        "name": "github_token",
        "type": "secret",
        "description": "GitHub personal access token with repo and user scope",
        "required": true,
        "validation": {
          "pattern": "^(ghp_[a-zA-Z0-9]{36}|github_pat_[a-zA-Z0-9]{82})$"
        }
      },
      {
        "name": "server_port",
        "type": "number",
        "description": "Port for SSE server",
        "required": false,
        "default": 3100,
        "validation": {
          "min": 1024,
          "max": 65535
        }
      }
    ],
    "config_file": {
      "path": "~/.github-mcp/config.json",
      "format": "json",
      "template": {
        "token": "${github_token}",
        "port": "${server_port}"
      }
    }
  },
  
  "tools": {
    "protocol": "mcp",
    "connection": {
      "type": "sse",
      "url": "http://localhost:${server_port}/sse",
      "start_command": "github-mcp-server --port ${server_port}"
    },
    "available_tools": [
      {
        "name": "search_repositories",
        "description": "Search for GitHub repositories using keywords or advanced syntax"
      },
      {
        "name": "create_issue",
        "description": "Create a new issue in a repository"
      },
      {
        "name": "get_pull_requests",
        "description": "List pull requests for a repository"
      }
    ]
  },
  
  "permissions": {
    "required": ["network:http", "process:execute"],
    "scope": {
      "network": {
        "domains": ["api.github.com", "github.com"],
        "protocols": ["https"],
        "ports": [443]
      }
    }
  },
  
  "skill": {
    "included": true,
    "location": "https://github.com/github/mcp-server/blob/main/SKILL.md",
    "local_path": "~/.aip/skills/github-integration.md"
  },
  
  "metadata": {
    "tags": ["github", "version-control", "collaboration", "mcp"],
    "category": "developer-tools",
    "keywords": ["git", "repository", "issues", "pull-requests"],
    "use_cases": [
      "Create issues from bug reports",
      "Search repositories",
      "Manage pull requests"
    ]
  }
}
```

### 4.2 Manifest Sections

#### capability
Basic identification and metadata.

#### installation
How to install the capability on different platforms.

#### configuration
What parameters need to be configured (API keys, paths, etc.).

#### tools
**Important:** This section lists available tools for **documentation purposes only**. Agents use the native protocol (MCP/CLI/HTTP) to discover and invoke tools.

For MCP servers, agents will call MCP's `tools/list` to get the actual tool schemas. The manifest just provides a high-level overview.

#### permissions
What access the capability needs (filesystem, network, etc.).

#### skill
Optional link to a skill file (Markdown guide).

#### metadata
Tags, categories, keywords for discovery.

---

## 5. Installation Protocol

### 5.1 Installation Step Types

**npm_install** - Install Node.js package
```json
{
  "type": "npm_install",
  "package": "@github/mcp-server",
  "version": "^2.1.0",
  "global": true
}
```

**pip_install** - Install Python package
```json
{
  "type": "pip_install",
  "package": "code-analyzer",
  "version": ">=1.0.0"
}
```

**docker_pull** - Pull Docker image
```json
{
  "type": "docker_pull",
  "image": "example/tool:latest"
}
```

**command** - Execute arbitrary shell command
```json
{
  "type": "command",
  "command": "curl -fsSL https://example.com/install.sh | bash",
  "validation": {
    "command": "which example-tool",
    "expected_exit_code": 0
  }
}
```

**download** - Download and install binary
```json
{
  "type": "download",
  "url": "https://example.com/releases/v1.0/tool-linux-amd64",
  "destination": "/usr/local/bin/tool",
  "checksum": {
    "algorithm": "sha256",
    "value": "abc123..."
  },
  "permissions": "755"
}
```

### 5.2 Installation Flow

```
1. Load manifest from URL or registry
2. Validate manifest schema
3. Check platform compatibility
4. Verify system requirements (Node.js, Python, etc.)
5. Execute installation steps sequentially
6. Validate each step
7. Mark capability as installed
8. Proceed to configuration
```

### 5.3 Validation

Each installation step should include validation:

```json
{
  "type": "npm_install",
  "package": "@github/mcp-server",
  "validation": {
    "command": "github-mcp-server --version",
    "expected_exit_code": 0,
    "expected_output": "^2\\."
  }
}
```

Agents MUST abort installation if validation fails.

---

## 6. Configuration Protocol

### 6.1 Parameter Types

| Type | Description | Storage | Example |
|------|-------------|---------|---------|
| `secret` | API keys, tokens | Secure keychain | GitHub token |
| `string` | Text values | Config file | Base URL |
| `number` | Numeric values | Config file | Port number |
| `boolean` | True/false | Config file | Enable cache |
| `path` | File/directory paths | Config file | Log directory |
| `url` | Valid URLs | Config file | API endpoint |

### 6.2 Configuration Example

```json
{
  "configuration": {
    "parameters": [
      {
        "name": "api_key",
        "type": "secret",
        "description": "API key for authentication",
        "required": true,
        "validation": {
          "pattern": "^sk-[a-zA-Z0-9]{32}$"
        }
      },
      {
        "name": "timeout_ms",
        "type": "number",
        "description": "Request timeout in milliseconds",
        "default": 5000,
        "validation": {
          "min": 1000,
          "max": 60000
        }
      }
    ],
    "config_file": {
      "path": "~/.my-tool/config.json",
      "format": "json",
      "template": {
        "apiKey": "${api_key}",
        "timeout": "${timeout_ms}"
      }
    }
  }
}
```

### 6.3 Secure Storage

**Secrets MUST be stored securely:**
- macOS: Keychain
- Windows: Credential Manager
- Linux: Secret Service API / gnome-keyring
- Containers: Environment variables with restricted permissions

**Regular config can be stored in:**
- JSON/YAML files with appropriate permissions
- Agent's configuration database

### 6.4 Variable Substitution

Configuration values can be referenced elsewhere in the manifest:

```json
{
  "configuration": {
    "parameters": [
      {"name": "port", "type": "number", "default": 3000}
    ]
  },
  "tools": {
    "connection": {
      "url": "http://localhost:${port}/sse"
    }
  }
}
```

---

## 7. Skill Format

### 7.1 Purpose

Skills are **optional Markdown files** that help agents understand capabilities. They serve the same purpose as Anthropic Skills (in `/mnt/skills/`) but for external capabilities.

**Skills provide:**
- When to use the capability
- Which tools to use for which tasks
- Best practices
- Common workflows
- Example interactions

### 7.2 Skill Structure

```markdown
---
name: github-integration
description: Use this capability when working with GitHub
triggers:
  - User mentions GitHub, repos, issues, or PRs
  - User wants to create/search/manage GitHub resources
capability_id: com.github.mcp-server
---

# GitHub Integration Skill

## When to Use This Capability

Use this when the user wants to:
- Search for repositories
- Create or manage issues
- Work with pull requests
- Access GitHub API

## Available MCP Tools

### search_repositories
**When to use:** User asks to find repositories  
**Input:** Search query (supports GitHub search syntax)  
**Output:** List of repositories with metadata  
**Example:** "Find popular Python ML libraries"

**Best practices:**
- Use specific search terms
- Sort by stars for popular repos
- Include language filter when relevant

### create_issue
**When to use:** User wants to report a bug or create a task  
**Input:** 
- `repo` - Repository in format "owner/name"
- `title` - Concise issue title
- `body` - Detailed description
- `labels` - Array of label names (optional)

**Best practices:**
- Search for duplicates first using search_repositories
- Include reproduction steps for bugs
- Add relevant labels for organization

### get_pull_requests
**When to use:** User wants to review or list PRs  
**Input:**
- `repo` - Repository
- `state` - "open", "closed", or "all"

**Output:** List of PRs with status

## Common Workflows

### Workflow: Creating a Bug Report

1. Verify repository exists
2. Search for duplicate issues
3. Create issue with:
   - Descriptive title
   - Reproduction steps
   - Expected vs actual behavior
   - Environment details
4. Add "bug" label
5. Return issue URL to user

**Example interaction:**

User: "There's a bug in the login form - it doesn't work on mobile"

Agent thought process:
1. Identify repo from context (or ask user)
2. Search existing issues for "login mobile"
3. No duplicates found
4. Create issue:
   - Title: "Login form non-functional on mobile devices"
   - Body: Include user's description + ask for details
   - Labels: ["bug", "mobile"]
5. Respond: "Created issue #142: [link]"

### Workflow: Finding Relevant Repositories

User: "Show me popular TypeScript testing libraries"

Agent thought process:
1. Use search_repositories tool
2. Query: "language:typescript testing"
3. Sort by stars
4. Limit to top 5
5. Format results with descriptions

## Tips and Best Practices

- **Always search before creating** - Avoid duplicate issues
- **Use labels consistently** - Helps with organization
- **Include context** - Link related issues/PRs
- **Batch operations** - When reviewing multiple PRs, fetch all at once

## Error Handling

- If authentication fails, check that github_token is valid
- If rate limited, wait before retrying
- If repo not found, verify repo name format is "owner/name"

## Advanced Usage

For complex workflows, consider:
- Combining search with filters for precise results
- Using GitHub's advanced search syntax
- Creating issues with templates
- Batch processing for multiple operations
```

### 7.3 Skill Location

Skills can be:

1. **Bundled with capability** - Included in package
2. **Hosted remotely** - URL in manifest
3. **User-created** - Custom skills for specific use cases

```json
{
  "skill": {
    "included": true,
    "location": "https://github.com/github/mcp-server/SKILL.md",
    "local_path": "~/.aip/skills/github-integration.md"
  }
}
```

### 7.4 Skill Loading

Agents load skills into context when using a capability:

```python
# Agent pseudocode
capability = aip.load_capability("com.github.mcp-server")
skill = aip.load_skill(capability.id)

# Skill is now in agent's context
# Agent understands when/how to use GitHub tools
```

---

## 8. Tool Invocation

### 8.1 Critical: AIP Does NOT Handle Invocation

**AIP's role ends after installation and configuration.**

Agents use **native protocols** to invoke tools:

```
┌─────────────────────────────────────┐
│         AIP Responsibilities        │
│  ✓ Discovery                        │
│  ✓ Installation                     │
│  ✓ Configuration                    │
│  ✓ Skill loading (optional)         │
└─────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│      Native Protocol Layer          │
│  • MCP for MCP servers              │
│  • Shell for CLI tools              │
│  • HTTP for APIs                    │
└─────────────────────────────────────┘
```

### 8.2 MCP Server Invocation

**Manifest provides connection info:**
```json
{
  "tools": {
    "protocol": "mcp",
    "connection": {
      "type": "sse",
      "url": "http://localhost:3100/sse"
    }
  }
}
```

**Agent uses MCP protocol:**
```
1. Connect to SSE endpoint
2. Send MCP initialize request
3. List available tools via MCP
4. Invoke tools using MCP protocol:
   {
     "method": "tools/call",
     "params": {
       "name": "search_repositories",
       "arguments": {"query": "python ML"}
     }
   }
5. Receive response via MCP
```

**No AIP involvement in invocation.**

### 8.3 CLI Tool Invocation

**Manifest provides command info:**
```json
{
  "tools": {
    "protocol": "cli",
    "connection": {
      "command": "filesearch"
    }
  }
}
```

**Agent uses shell:**
```bash
filesearch search --query "machine learning" --format json
```

### 8.4 HTTP API Invocation

**Manifest provides API info:**
```json
{
  "tools": {
    "protocol": "http",
    "connection": {
      "base_url": "https://api.weather.example.com",
      "authentication": {
        "type": "header",
        "header_name": "X-API-Key",
        "value": "${api_key}"
      }
    }
  }
}
```

**Agent uses HTTP:**
```http
GET https://api.weather.example.com/v1/current?location=Tokyo
X-API-Key: <from_config>
```

---

## 9. Permission Model

### 9.1 Permission Types

```json
{
  "permissions": {
    "required": [
      "filesystem:read",
      "filesystem:write",
      "network:http",
      "process:execute"
    ]
  }
}
```

**Available permissions:**
- `filesystem:read` - Read files
- `filesystem:write` - Write/modify files
- `filesystem:delete` - Delete files
- `filesystem:execute` - Execute binaries
- `network:http` - HTTP/HTTPS requests
- `network:socket` - Raw socket connections
- `network:dns` - DNS resolution
- `process:execute` - Spawn processes
- `process:signal` - Send signals
- `system:env` - Read environment variables
- `system:info` - Read system info

### 9.2 Permission Scopes

```json
{
  "permissions": {
    "required": ["filesystem:read", "network:http"],
    "scope": {
      "filesystem": {
        "paths": ["~/Documents/*", "~/Projects/*"],
        "operations": ["read"]
      },
      "network": {
        "domains": ["api.github.com"],
        "protocols": ["https"],
        "ports": [443]
      }
    }
  }
}
```

### 9.3 Permission Workflow

```
1. Agent reads manifest
2. Agent reviews requested permissions
3. Agent prompts user (in interactive mode):
   "GitHub MCP Server requires:
    - Network access to api.github.com
    - Execute process permission
    Allow? (y/n)"
4. If approved, agent installs
5. Agent enforces permissions at runtime
```

---

## 10. Discovery Protocol

### 10.1 Registry API

```http
GET /api/v1/capabilities?tags=github&category=developer-tools
```

Response:
```json
{
  "capabilities": [
    {
      "id": "com.github.mcp-server",
      "name": "GitHub MCP Server",
      "version": "2.1.0",
      "description": "MCP server for GitHub API",
      "manifest_url": "https://github.com/github/mcp-server/manifest.json",
      "tags": ["github", "version-control", "mcp"],
      "category": "developer-tools",
      "rating": 4.8,
      "downloads": 15420,
      "has_skill": true
    }
  ],
  "total": 1,
  "page": 1
}
```

### 10.2 Search by Use Case

```http
GET /api/v1/capabilities?use_case=create+github+issues
```

### 10.3 Direct Manifest URL

```
https://github.com/github/mcp-server/manifest.json
```

### 10.4 Local Filesystem

```
~/.aip/capabilities/*/manifest.json
~/.claude-code/capabilities/*/manifest.json
```

---

## 11. Lifecycle Management

### 11.1 States

```
Discovered
    ↓
Installing
    ↓
Installed
    ↓
Configured
    ↓
Ready ←→ Disabled
    ↓
Uninstalled
```

### 11.2 Version Management

```json
{
  "capability": {
    "version": "2.1.0"
  }
}
```

Agents can:
- Pin to specific versions
- Auto-update (with user permission)
- Handle breaking changes via changelog

### 11.3 Uninstallation

```json
{
  "uninstall": {
    "steps": [
      {
        "type": "command",
        "command": "npm uninstall -g @github/mcp-server"
      },
      {
        "type": "delete",
        "paths": ["~/.github-mcp"]
      }
    ]
  }
}
```

---

## 12. Platform Integration

### 12.1 Claude Code

```json
{
  "aip": {
    "enabled": true,
    "capability_directory": "~/.claude-code/capabilities",
    "skill_directory": "~/.claude-code/skills",
    "registry_url": "https://registry.claudecode.com/api/v1",
    "auto_install": false,
    "sandboxing": "docker"
  }
}
```

### 12.2 Claude Cowork

```json
{
  "aip": {
    "enabled": true,
    "capability_directory": "~/.claude-cowork/capabilities",
    "skill_directory": "~/.claude-cowork/skills",
    "ui_prompts": true,
    "auto_update": true
  }
}
```

### 12.3 Custom Agents

```python
from aip import CapabilityManager

manager = CapabilityManager(
    capability_dir="~/.aip/capabilities",
    skill_dir="~/.aip/skills",
    registry_url="https://aip-registry.example.com"
)

# Discover
caps = manager.discover(tags=["github"])

# Install
cap = manager.install("com.github.mcp-server")

# Configure
cap.configure({"github_token": "ghp_..."})

# Load skill (optional)
skill = manager.load_skill(cap.id)
# Skill is now in agent's context

# Use native protocol (MCP) to invoke tools
mcp_client = cap.get_mcp_client()
result = mcp_client.call_tool("search_repositories", {
    "query": "python ML"
})
```

---

## 13. Security Considerations

### 13.1 Manifest Verification

- HTTPS-only for manifest URLs
- Optional cryptographic signatures
- Checksum verification for downloads

```json
{
  "security": {
    "signature": {
      "algorithm": "ed25519",
      "public_key": "base64...",
      "signature": "base64..."
    }
  }
}
```

### 13.2 Sandboxing

Agents SHOULD run capabilities in isolated environments:
- Docker containers
- Virtual machines
- OS-level sandboxes (firejail, bubblewrap)

### 13.3 Audit Logging

```json
{
  "timestamp": "2026-02-08T10:30:00Z",
  "capability_id": "com.github.mcp-server",
  "action": "tool_invocation",
  "tool": "create_issue",
  "user": "user@example.com"
}
```

---

## 14. References

- **Model Context Protocol:** https://modelcontextprotocol.io
- **JSON Schema:** https://json-schema.org
- **Semantic Versioning:** https://semver.org

---

## Appendices

### Appendix A: Complete Examples
See `examples/` directory

### Appendix B: Skill Template
See `skill-template.md`

### Appendix C: JSON Schema
See `aip-manifest-schema.json`

---

**Version:** 0.1.0  
**Status:** Draft for Community Review  
**Last Updated:** February 8, 2026
