# Claude Code + SonarCloud: Automated Code Quality Analysis via MCP

A complete, battle-tested guide to setting up automated SonarCloud code analysis using **Claude Code** and the **Model Context Protocol (MCP)** ecosystem.

## What This Is

This guide was written from real experience — every challenge documented here was actually hit, debugged, and solved during setup. No theoretical fluff.

The goal: **Use Claude Code to set up, trigger, and read SonarCloud code quality reports — all from the terminal**, without constantly switching to browser UIs.

## The Stack

```
You write code → Claude commits & pushes → GitHub Actions runs sonar-scanner
→ SonarCloud analyzes → Claude reads the report via MCP → Shows you the issues
```

| Component | Role |
|-----------|------|
| **Claude Code** (CLI) | The orchestrator — commits, pushes, creates PRs, polls, reads reports |
| **SonarQube MCP Server** | Reads SonarCloud data (quality gates, issues, metrics) |
| **GitHub MCP Server** | Creates files, pushes commits, manages PRs from Claude |
| **GitHub Actions** | Runs the actual sonar-scanner on push/PR |
| **SonarCloud** | Hosts the analysis results (free tier compatible) |

## What's Inside

| File | Description |
|------|-------------|
| [`setup-guide.txt`](setup-guide.txt) | Full end-to-end setup guide (13 sections, all challenges documented) |
| [`automated-process.txt`](automated-process.txt) | The automated PR analysis flow + "Feed to Claude" instructions |
| [`examples/sonarcloud.yml`](examples/sonarcloud.yml) | GitHub Actions workflow file |
| [`examples/sonar-project.properties`](examples/sonar-project.properties) | SonarCloud project configuration |
| [`examples/settings.local.json`](examples/settings.local.json) | Claude Code permissions config for auto-approval |

## Quick Start

### Prerequisites
- Claude Code installed (`npm install -g @anthropic-ai/claude-code`)
- Docker installed and running
- GitHub account + repository
- SonarCloud account (free tier works)

### 1. Set Up MCP Servers

```bash
# SonarQube MCP (reads SonarCloud data)
docker pull mcp/sonarqube
claude mcp add sonarqube \
    --env SONARQUBE_TOKEN=<your-token> \
    --env SONARQUBE_ORG=<your-org> \
    -- docker run -i --rm -e SONARQUBE_TOKEN -e SONARQUBE_ORG mcp/sonarqube

# GitHub MCP (interacts with GitHub repos)
docker pull ghcr.io/github/github-mcp-server
claude mcp add github \
    -e GITHUB_PERSONAL_ACCESS_TOKEN=<your-pat> \
    -- docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server
```

### 2. Restart Claude Code

MCP servers only load on startup.

### 3. Ask Claude to Set Up the Pipeline

```
> Set up SonarCloud analysis for this repo
```

Claude will create the workflow file, sonar config, and push them via GitHub MCP.

### 4. Add SONAR_TOKEN Secret

Go to GitHub → Repo → Settings → Secrets → Actions → add `SONAR_TOKEN`.

(This is the only browser step — GitHub doesn't allow creating secrets via API for security reasons.)

### 5. Run Analysis

```
> Commit these changes and create a PR for sonar analysis
```

Claude will: commit → push → create PR → poll for CI completion → pull report → show issues.

## Key Findings

- **PR analysis works on free tier** (branch analysis does NOT)
- **SonarQube MCP is more reliable than GitHub status API** for detecting analysis completion
- **The full loop takes ~2.5 minutes** (mostly CI execution time)
- **9 challenges documented** with exact symptoms, causes, and fixes
- **"Feed to Claude" section** lets any Claude instance run the flow autonomously

## Free Tier Limitations

| Feature | Free Tier |
|---------|----------|
| Private repo lines | 50,000 max |
| Branch analysis | Main branch only |
| PR analysis | Yes (recommended!) |
| Automatic Analysis | JS/TS/Python/Java only (no Go/Rust/C++) |

## GitHub PAT Permissions Needed

| Permission | Why |
|------------|-----|
| Contents: Read/Write | Push files |
| Workflows: Read/Write | Create `.github/workflows/` files |
| Actions: Read/Write | Manage workflow runs |
| Pull requests: Read/Write | Create PRs |
| Commit statuses: Read | Poll CI status |
| Metadata: Read | Required |

## License

MIT — use this however you want.

---

*Built with Claude Code (Opus 4.6) — every line of this guide was generated from a real setup session.*
