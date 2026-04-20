# PR Creator Configuration

This file defines the user's project-specific settings for PR creation.
Copy this file to your project's `.claude/settings/pr-creator.json` and customize.

## Configuration Schema

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

## Full Example (with custom template)

```json
{
  "jira": {
    "enabled": false
  },
  "pr": {
    "titleFormat": "{type}: {summary}",
    "titleMaxLength": 70,
    "baseBranch": "main",
    "language": "en"
  },
  "customTemplate": "## What\n- 변경 내용\n\n## Why\n- 변경 이유\n\n## How to Test\n-"
}
```

## Field Descriptions

### jira
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | boolean | `true` | Enable Jira integration |
| `baseUrl` | string | `""` | Jira instance URL (e.g., `https://your-org.atlassian.net`) |
| `ticketPattern` | string | `[A-Z]+-[0-9]+` | Regex pattern to extract ticket number from branch name |

### pr
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `titleFormat` | string | `{type}({ticket}): {summary}` | PR title template. Available variables: `{type}`, `{ticket}`, `{summary}` |
| `titleMaxLength` | number | `70` | Maximum character length for PR title |
| `baseBranch` | string | `main` | Base branch to compare against |
| `language` | string | `en` | Summary language (`en`, `ko`, `ja`) |

### sections
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `related` | boolean | `true` | Include Related (Jira link) section |
| `summary` | boolean | `true` | Include Summary section |
| `changes` | boolean | `true` | Include Changes section |
| `impact` | boolean | `true` | Include Impact section |
| `testing` | boolean | `true` | Include Testing section |

### customTemplate
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `customTemplate` | string \| null | `null` | Custom PR body template. Use `##` headings to define sections. When set, `sections` config is ignored and this template is used instead. |