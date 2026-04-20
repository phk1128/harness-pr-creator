# harness-pr-creator

**Automated PR Creation Harness** - A Claude Code Plugin

**English** | [한국어](README_KO.md)

A skill-based harness that analyzes your branch's commits and diffs to automatically generate standardized pull requests with optional Jira integration.

## Features

- **Jira Integration** - Automatically extracts ticket numbers from branch names and links them in the PR
- **Commit Analysis** - Analyzes commit types (feat, fix, refactor...) to determine PR title
- **Standardized Template** - Generates PRs with Related, Summary, Changes, Impact, Testing sections
- **Configurable** - Customize Jira URL, title format, base branch, language, and section visibility
- **User Confirmation** - Always previews before creating the PR
- **Multi-language** - Supports English, Korean, and Japanese summaries

## Installation

### Option 1: CLI (Recommended)

```bash
# 1. Add marketplace
/plugin marketplace add phk1128/harness-pr-creator

# 2. Install plugin
/plugin install harness-pr-creator@phk1128-harness-pr-creator
```

### Option 2: Manual

Add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "enabledPlugins": {
    "harness-pr-creator@phk1128-harness-pr-creator": true
  },
  "extraKnownMarketplaces": {
    "phk1128-harness-pr-creator": {
      "source": {
        "source": "github",
        "repo": "phk1128/harness-pr-creator"
      }
    }
  }
}
```

### Update

```bash
# 1. Refresh marketplace catalog
claude plugin marketplace update harness-pr-creator-marketplace

# 2. Update plugin
claude plugin update harness-pr-creator@harness-pr-creator-marketplace
```

## Usage

```
/create-pr              # Auto-extract ticket from branch name
/create-pr CSP-1234     # Specify Jira ticket directly
```

The skill will:

1. Load your project configuration (or use defaults)
2. Collect branch info, commits, and diffs
3. Extract Jira ticket from branch name
4. Generate PR title and body
5. Preview and ask for confirmation
6. Push and create the PR

## Configuration

Create `.claude/settings/pr-creator.json` in your project root:

```json
{
  "jira": {
    "enabled": true,
    "baseUrl": "https://your-org.atlassian.net",
    "ticketPattern": "[A-Z]+-[0-9]+"
  },
  "pr": {
    "titleFormat": "{type}({ticket}): {summary}",
    "titleMaxLength": 70,
    "baseBranch": "main",
    "language": "en"
  },
  "sections": {
    "related": true,
    "summary": true,
    "changes": true,
    "impact": true,
    "testing": true
  }
}
```

### Configuration Fields

| Field | Default | Description |
|-------|---------|-------------|
| `jira.enabled` | `true` | Enable Jira integration |
| `jira.baseUrl` | `""` | Jira URL. If empty, you'll be prompted on first run |
| `jira.ticketPattern` | `[A-Z]+-[0-9]+` | Regex to extract ticket from branch name |
| `pr.titleFormat` | `{type}({ticket}): {summary}` | Title template |
| `pr.titleMaxLength` | `70` | Max title length |
| `pr.baseBranch` | `main` | Base branch for comparison |
| `pr.language` | `en` | Summary language (`en`, `ko`, `ja`) |
| `sections.*` | `true` | Toggle individual PR sections |

### No Config File?

If no configuration file exists, the skill uses sensible defaults and prompts for required values (like Jira URL) interactively.

## Examples

### With Jira

Branch: `feature/CSP-1483-settlement-api`

```
feat(CSP-1483): Add settlement API endpoints

## Related
- Jira: [CSP-1483](https://your-org.atlassian.net/browse/CSP-1483)

## Summary
- Add settlement calculation and reconciliation API
- ...
```

### Without Jira

Branch: `refactor/excel-sax`

```
refactor: Migrate Excel parser to SAX streaming

## Summary
- Replace DOM-based parsing with SAX for memory efficiency
- ...
```

## Requirements

- [Claude Code](https://claude.ai/code) CLI
- [GitHub CLI](https://cli.github.com/) (`gh`) - authenticated
- Git repository with a remote

## License

[MIT](LICENSE)