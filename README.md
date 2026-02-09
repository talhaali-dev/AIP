# Agent Interoperability Protocol (AIP)

**Version:** 0.1.0  
**Status:** Draft Specification  
**Author:** Turbo Launch

---

## 📋 Overview

The Agent Interoperability Protocol (AIP) standardizes how AI agents discover, install, configure, and understand external capabilities (MCP servers, CLI tools, APIs).

### What AIP Provides

1. **Manifest Format** - JSON/YAML files describing installation and configuration
2. **Skill Format** - Optional Markdown guides (like Anthropic Skills) that help agents understand capabilities
3. **Discovery Protocol** - Searchable registries of capabilities

### What AIP Does NOT Do

❌ Replace MCP, CLI, or HTTP protocols  
❌ Introduce new tool invocation mechanisms  
❌ Require specific agent frameworks

**AIP is metadata and automation on TOP of existing protocols.**

---

## 🎯 Core Concepts

### Manifest
A JSON/YAML file that describes:
- How to install the capability
- What configuration it needs
- What tools it provides
- What permissions it requires

Think: `package.json` + installation instructions

### Skill
An optional Markdown guide (like Anthropic Skills) that helps agents:
- Know when to use the capability
- Understand which tools to use for which tasks
- Follow best practices
- Learn from examples

Think: `README.md` or tutorial for agents

### The Key Difference

```
┌─────────────────────────────────────┐
│   Manifest (manifest.json)          │
│   - Installation instructions       │
│   - Configuration parameters        │
│   - Tool descriptions               │
│   - Machine-readable                │
└─────────────────────────────────────┘
                 +
┌─────────────────────────────────────┐
│   Skill (SKILL.md)                  │
│   - Usage guidelines                │
│   - Best practices                  │
│   - Example workflows               │
│   - Human/AI-readable               │
└─────────────────────────────────────┘
                 =
┌─────────────────────────────────────┐
│   Complete AIP Capability           │
│   - Automated installation          │
│   - Intelligent usage               │
└─────────────────────────────────────┘
```

---

## 🚀 Quick Start

### For Agent Developers

```python
from aip import CapabilityManager

# Initialize AIP
manager = CapabilityManager(
    capability_dir="~/.aip/capabilities",
    skill_dir="~/.aip/skills"
)

# Discover capabilities
caps = manager.discover(tags=["github"])

# Install
cap = manager.install("com.github.mcp-server")

# Configure
cap.configure({"github_token": "ghp_..."})

# Load skill (optional - provides context)
skill = manager.load_skill(cap.id)
# Skill is now in agent's context

# Use native protocol (MCP) to invoke tools
# AIP's job is done - now use MCP directly
mcp_client = cap.get_mcp_client()
result = mcp_client.call_tool("search_repositories", {
    "query": "python ML"
})
```

### For Capability Providers

**Step 1: Create manifest.json**
```json
{
  "aip_version": "0.1.0",
  "capability": {
    "id": "com.example.my-tool",
    "name": "My Tool",
    "version": "1.0.0",
    "type": "mcp_server"
  },
  "installation": {
    "steps": [{"type": "npm_install", "package": "@example/my-tool"}]
  },
  "tools": {
    "protocol": "mcp",
    "connection": {
      "type": "sse",
      "url": "http://localhost:3000/sse"
    }
  }
}
```

**Step 2: Create SKILL.md (optional)**
```markdown
# My Tool Skill

## When to Use
Use this when the user wants to...

## Available MCP Tools
- `tool_name` - Description and when to use it

## Best Practices
- Always check X before doing Y
- Use Z for better results
```

---

## 🏗️ Architecture

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

### Workflow

```
1. DISCOVER
   Agent searches registry: "I need GitHub integration"
   → Finds: "GitHub MCP Server"

2. INSTALL
   Agent reads manifest
   → Executes: npm install -g @github/mcp-server
   → Validates installation

3. CONFIGURE
   Agent prompts: "Enter GitHub token"
   → Stores securely in keychain
   → Writes config file

4. LOAD SKILL (optional)
   Agent downloads SKILL.md
   → Reads: "Use search_repositories when user asks to find repos"
   → Agent now understands when/how to use tools

5. INVOKE TOOLS
   Agent uses MCP protocol (not AIP)
   → Calls: search_repositories via MCP
   → AIP is not involved in invocation
```

---

## 📚 Documentation

### Core Documents

- **[AIP Specification (RFC)](Spec/aip-spec-rfc.md)** - Complete technical specification
- **[Manifest Schema (JSON)](Spec/aip-manifest-schema.json)** - JSON Schema for validation
- **[AIP Introduction](Spec/AIP0.md)** - Introduction to AIP

### Examples

- **[GitHub MCP Server](examples/github-mcp-manifest.json)** - MCP server manifest
- **[GitHub Skill](examples/github-skill.md)** - Example skill guide
- **[File Search CLI](examples/filesearch-cli-manifest.json)** - CLI tool manifest
- **[Weather API](examples/weather-api-manifest.json)** - HTTP API manifest

---

## 🔑 Key Principles

### 1. AIP Uses Native Protocols

```
✅ CORRECT:
- Agent installs MCP server using AIP manifest
- Agent invokes tools using MCP protocol

❌ WRONG:
- Agent installs MCP server using AIP manifest
- Agent invokes tools using "AIP protocol" (doesn't exist!)
```

### 2. Skills Are Optional But Valuable

Without skill:
```
Agent: "I have a GitHub MCP server installed"
Agent: "It has a tool called 'search_repositories'"
Agent: "Not sure when to use it..." 🤔
```

With skill:
```
Agent: "I have a GitHub MCP server installed"
Agent: "The skill says to use search_repositories when user asks to find repos"
Agent: "User said 'find Python ML libraries' → I should use this tool!" ✅
```

### 3. Manifests Are Machine-Readable, Skills Are AI-Readable

| Manifest | Skill |
|----------|-------|
| JSON format | Markdown format |
| Installation steps | Usage guidelines |
| Config parameters | Best practices |
| Tool list | When to use each tool |
| For automation | For intelligence |

---

## 🔐 Security Model

### Permissions

```json
{
  "permissions": {
    "required": ["filesystem:read", "network:http"],
    "scope": {
      "filesystem": {
        "paths": ["~/Documents/*"],
        "operations": ["read"]
      },
      "network": {
        "domains": ["api.github.com"],
        "protocols": ["https"]
      }
    }
  }
}
```

### Secure Configuration

- **Secrets** (API keys, tokens) → Stored in system keychain
- **Regular config** → JSON/YAML files with appropriate permissions

### Sandboxing

Agents SHOULD run capabilities in isolated environments:
- Docker containers
- Virtual machines
- OS-level sandboxes

---

## 🌐 Platform Support

### Claude Code

Developer-focused agent for coding tasks.

```json
{
  "aip": {
    "capability_directory": "~/.claude-code/capabilities",
    "skill_directory": "~/.claude-code/skills",
    "sandboxing": "docker"
  }
}
```

### Claude Cowork

File and task automation agent.

```json
{
  "aip": {
    "capability_directory": "~/.claude-cowork/capabilities",
    "skill_directory": "~/.claude-cowork/skills",
    "ui_prompts": true
  }
}
```

### Custom Agents

Works with any agent framework - just implement the AIP SDK.

---

## 📦 Capability Types

### MCP Server

```json
{
  "type": "mcp_server",
  "tools": {
    "protocol": "mcp",
    "connection": {
      "type": "sse",
      "url": "http://localhost:3100/sse"
    }
  }
}
```

Agent uses **MCP protocol** to invoke tools.

### CLI Tool

```json
{
  "type": "cli_tool",
  "tools": {
    "protocol": "cli",
    "connection": {
      "command": "filesearch"
    }
  }
}
```

Agent uses **shell commands** to invoke tools.

### HTTP API

```json
{
  "type": "api_service",
  "tools": {
    "protocol": "http",
    "connection": {
      "base_url": "https://api.example.com",
      "authentication": {
        "type": "header",
        "header_name": "X-API-Key",
        "value": "${api_key}"
      }
    }
  }
}
```

Agent uses **HTTP requests** to invoke tools.

---

## 🔍 Discovery

### Registry Search

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
      "manifest_url": "https://github.com/github/mcp-server/manifest.json",
      "has_skill": true
    }
  ]
}
```

### Direct Manifest URL

```
https://example.com/capabilities/my-tool/manifest.json
```

### Local Filesystem

```
~/.aip/capabilities/*/manifest.json
```

---

## 💡 Use Cases

### Automated Development Workflows

```
User: "Set up ESLint for this TypeScript project"

Claude Code:
1. Discovers ESLint MCP server in registry
2. Installs via manifest instructions
3. Loads skill to understand ESLint configuration
4. Uses MCP to invoke setup_eslint tool
5. Creates .eslintrc.json
```

### File Operations

```
User: "Find all Python files with machine learning code"

Claude Cowork:
1. Discovers file search CLI tool
2. Installs via npm
3. Loads skill to understand search syntax
4. Runs: filesearch search --query "machine learning" --type python
5. Presents results
```

### API Integration

```
User: "What's the weather in Tokyo?"

Custom Agent:
1. Discovers weather API capability
2. No installation needed (cloud API)
3. Configures with API key
4. Makes HTTP request: GET /v3/current?location=Tokyo
5. Returns current weather
```

---

## 🔄 Relationship to MCP

**MCP (Model Context Protocol):**
- Defines tool invocation protocol
- Specifies tool schemas
- Handles communication between agent and server

**AIP:**
- Defines how to discover MCP servers
- Automates installation
- Manages configuration
- Provides usage intelligence via skills

```
AIP helps you GET the MCP server
MCP helps you USE the MCP server
```

---

## 🧪 Validation

Validate manifests:

```bash
# Using ajv-cli
npm install -g ajv-cli
ajv validate -s Spec/aip-manifest-schema.json -d my-manifest.json

# Using Python jsonschema
pip install jsonschema
python -m jsonschema -i my-manifest.json Spec/aip-manifest-schema.json
```

---

## 🤝 Contributing

We welcome contributions!

- **Feedback** - Open issues with suggestions
- **Examples** - Share capability manifests and skills
- **Implementations** - Build AIP support into your agent
- **Documentation** - Improve guides

---

## 📄 License

AIP specification: MIT License

Individual capabilities may have different licenses.

---

## 🔗 Related Projects

- **Model Context Protocol (MCP):** https://modelcontextprotocol.io
- **JSON Schema:** https://json-schema.org
- **Semantic Versioning:** https://semver.org

---

## 📞 Contact

**Turbo Launch**  
Helping founders launch MVPs in 15 days

- Website: https://turbolaunch.com
- Email: support@turbolaunch.com

---

## ✅ Getting Started Checklist

### For Agent Developers

- [ ] Read the [AIP Specification](Spec/aip-spec-rfc.md)
- [ ] Review [example manifests](examples/)
- [ ] Understand manifest vs skill distinction
- [ ] Implement AIP manifest parser
- [ ] Implement skill loader (optional)
- [ ] Test with example capabilities

### For Capability Providers

- [ ] Read the [AIP Specification](Spec/aip-spec-rfc.md)
- [ ] Create `manifest.json` for your capability
- [ ] Create `SKILL.md` guide (optional but recommended)
- [ ] Validate manifest against [schema](Spec/aip-manifest-schema.json)
- [ ] Test installation on target platforms
- [ ] Submit to capability registry

---

**Built with ❤️ by Turbo Launch**

**Version:** 0.1.0  
**Last Updated:** February 8, 2026  
**Status:** Draft for Community Review
