# Prerequisites — What You Need Before Starting

## Required Software

| Software | Min Version | Why | Install |
|----------|------------|-----|---------|
| **VS Code** | 1.100+ | Agent framework, chat modes, instructions, skills all require recent builds | https://code.visualstudio.com |
| **GitHub Copilot** | Pro+ plan | Pro+ required for custom agents, chat modes, model routing, MCP servers | $39/mo in VS Code Marketplace |
| **Node.js** | 20+ | MCP servers run via `npx`, project tooling is Node-based | https://nodejs.org |
| **npm** | 9+ | Comes with Node.js | (included) |
| **Git** | 2.30+ | Version control, GitHub integration | https://git-scm.com |

## Optional Software

| Software | Why | When Needed |
|----------|-----|-------------|
| **PostgreSQL** | Database MCP server, project database | If your project needs a database |
| **Docker** | Containerization, dev environment | If you want Docker-based dev |
| **Python** | Python instruction files, syntax hooks | If you write Python code |
| **PowerShell 7** | Better terminal experience on Windows | Windows only |

## Required Accounts

| Account | Why | Sign Up |
|---------|-----|---------|
| **GitHub** | Repository hosting, Copilot subscription, GitHub MCP server | https://github.com |
| **Tavily** | AI-powered web search MCP server (free tier: 1000 searches/mo) | https://tavily.com |

## API Keys You'll Need

### 1. GitHub Personal Access Token (PAT)

Generate at: https://github.com/settings/tokens

Required scopes:
- `repo` — Full repository access
- `read:org` — Read org membership
- `read:user` — Read user profile
- `user:email` — Read user email

This powers the GitHub MCP server (search code, create repos, manage issues/PRs).

### 2. Tavily API Key

Sign up at: https://tavily.com → Dashboard → API Keys

Free tier gives 1000 searches/month. Paid plans available for heavy use.

This powers the Tavily MCP server (deep web search, content extraction, crawling).

### 3. PostgreSQL Credentials (optional)

If you have PostgreSQL installed, you'll need:
- Host (default: `localhost`)
- Port (default: `5432`)
- Username (default: `postgres`)
- Password
- Database name

This powers the PostgreSQL MCP server (direct database queries from Copilot).

## VS Code Settings Access

You'll need to know where your VS Code settings are:

| OS | Location |
|----|----------|
| Windows | `%APPDATA%\Code\User\settings.json` |
| macOS | `~/Library/Application Support/Code/User/settings.json` |
| Linux | `~/.config/Code/User/settings.json` |

And the user-level MCP config:

| OS | Location |
|----|----------|
| Windows | `%APPDATA%\Code\User\mcp.json` |
| macOS | `~/Library/Application Support/Code/User/mcp.json` |
| Linux | `~/.config/Code/User/mcp.json` |

## Pre-Flight Checklist

Before running the setup, verify:

- [ ] VS Code is installed and updated to latest
- [ ] GitHub Copilot extension is installed and Pro+ subscription active
- [ ] Node.js 20+ is installed (`node --version`)
- [ ] Git is installed (`git --version`)
- [ ] GitHub PAT generated with correct scopes
- [ ] Tavily API key obtained
- [ ] PostgreSQL running (if you want the DB server)
- [ ] You know your VS Code settings.json location
