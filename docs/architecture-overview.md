# GitHub MCP Server Architecture Overview

This repository implements a GitHub-aware Model Context Protocol (MCP) server that exposes GitHub capabilities to MCP clients such as VS Code, Copilot CLI, Claude, Cursor, and other AI hosts.

```mermaid
flowchart LR
    subgraph Clients["MCP Hosts / AI Clients"]
        IDE["VS Code\nCursor\nClaude\nCopilot CLI\nOther MCP hosts"]
    end

    subgraph Runtime["GitHub MCP Server"]
        CLI["cmd/github-mcp-server\nEntry point\nstdio / docker / local binary"]
        CFG["internal\nConfiguration, auth, runtime, server setup"]
        TOOLS["pkg/github\nGitHub tool implementations\nrepos, issues, PRs, actions, alerts, users"]
        AGENT["agent-plugin\nAgent-specific integration"]
        UI["ui\nUI / helper surfaces"]
    end

    subgraph External["GitHub Platform"]
        API["GitHub REST API\nGraphQL\nActions\nCode scanning\nDependabot"]
        AUTH["OAuth / PAT / GitHub App\nGHE / ghe.com support"]
    end

    DOCS["docs\ninstallation guides\nserver configuration"]
    E2E["e2e\nend-to-end validation"]

    IDE -->|MCP requests / tool calls| CLI
    CLI --> CFG
    CFG --> TOOLS
    CFG --> AGENT
    CFG --> UI

    AUTH -->|Authentication| CLI
    CLI -->|Authenticated API calls| API
    TOOLS -->|Structured results| IDE
    API -->|Repository, issue, PR, workflow, security data| TOOLS

    DOCS -->|Usage guidance| CLI
    E2E -->|Validation| CLI
```

## Architecture Notes

- The server is designed to be consumed by MCP-compatible clients over stdio or via a hosted remote GitHub MCP endpoint.
- `cmd/github-mcp-server` is the runtime entry point for local binaries and Docker-based deployments.
- `internal` contains the core server lifecycle, configuration, and authentication wiring.
- `pkg/github` exposes the actual GitHub tool implementations and adapters for repos, issues, pull requests, workflow actions, code security, and user context.
- `agent-plugin` and `ui` add integration points for agent experiences and supporting surfaces.
- External dependencies are GitHub APIs and authentication mechanisms such as OAuth tokens, PATs, and GitHub Enterprise deployments.
- `docs` and `e2e` provide installation, configuration, and validation support for operators and maintainers.

## Primary Flow

1. An MCP client issues a request to the GitHub MCP server.
2. The server authenticates using OAuth, a PAT, or GitHub App credentials.
3. Tool implementations in `pkg/github` call the appropriate GitHub APIs.
4. Structured results are returned to the MCP client as tool responses.
5. The client can then reason over repos, code, issues, pull requests, actions, and security signals.
