# GitHub PR Sync with Notion

> Automatically sync GitHub Pull Requests with Notion Database using GitHub Actions

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Note**: If you have a Notion Business plan, we recommend using the [official Notion GitHub integration](https://www.notion.so/integrations/github) which provides more comprehensive features. This action is designed for solo developers and small teams without a Business plan who need minimal PR synchronization.

## Features

- 🔗 **Automatic PR Linking**: Automatically add PR links to Notion when creating a PR
- 🔢 **Multiple Task Support**: Link a single PR to multiple Notion tasks (e.g., `[TASK-123][TASK-456]`)
- ✏️ **Title Change Detection**: Automatically update Notion when PR title (task ID) changes
- ✅ **Status Update**: Automatically update Notion status to "Done" when PR is merged
- 💬 **PR Comments**: Post Notion page links as PR comments

## How It Works

### 1. PR Created
```
Create PR: [TASK-123] Add new feature
↓
✅ Add PR link to Notion
✅ Post comment on PR with Notion page link
```

### 2. Title Edited (Task ID Changed)
```
Change title: [TASK-123] → [TASK-456]
↓
✅ Remove PR link from TASK-123
✅ Add PR link to TASK-456
✅ Update PR comment to show TASK-456 only
```

### 3. PR Merged
```
Merge PR
↓
✅ Update Notion status to "Done"
```

## Quick Start

### 1. Create Notion Integration

1. Go to [Notion Integrations](https://www.notion.so/my-integrations)
2. Click "New integration"
3. Give it a name (e.g., "GitHub PR Sync")
4. Copy the **Internal Integration Token**

### 2. Setup Notion Database

Your Notion Database needs these properties:

| Property Name | Type | Description |
|--------------|------|-------------|
| `ID` | Unique ID | Task ID with prefix (e.g., TASK-123) |
| `Status` | Status or Select | Task status |
| `GitHub PR` | Rich text | PR links |

**Important**:
- Use **Unique ID** type for the `ID` property
- Configure the Unique ID with a **prefix** (e.g., "TASK") to match your PR title format `[TASK-123]`
- Add "Done" option to `Status` property
- Connect your Integration to the Database (... menu → Add connections)

### 3. Get Database ID

1. Open your Notion Database
2. Copy the URL
3. Extract the 32-character ID before `?v=`
   ```
   https://www.notion.so/abc123def456... → abc123def456...
   ```

### 4. Add GitHub Secrets

In your repository: Settings → Secrets and variables → Actions

Add these secrets:
- `NOTION_TOKEN`: Your Integration Token (required for authentication)
- `NOTION_DATABASE_ID`: Your Database ID (not sensitive, but storing as secret keeps configuration organized)

**Note**: The Database ID is not sensitive information. You can also use it as a workflow variable instead of a secret if preferred.

### 5. Add Workflow File

Create `.github/workflows/github-pr-sync-with-notion.yml` in your repository:

```yaml
name: GitHub PR Sync with Notion

on:
  pull_request:
    types: [opened, edited, closed]

jobs:
  sync:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      issues: write
    steps:
      - name: Sync to Notion
        uses: isoppp/github-pr-sync-with-notion@v0.1.0
        with:
          # Customize these based on your Notion Database setup
          notion_token: ${{ secrets.NOTION_TOKEN }}
          notion_database_id: ${{ secrets.NOTION_DATABASE_ID }}
          notion_id_property: 'ID'              # Property name for task ID (Unique ID type)
          notion_status_property: 'Status'      # Status property name (Status type, not Select)
          notion_status_value_done: 'Done'      # Status value when PR is merged
          notion_pr_property: 'GitHub PR'       # Property name for PR links (Rich text type)
          pr_title_prefix: 'TASK'               # Prefix in PR title (e.g., [TASK-123])
```

**Note**: Customize the inputs to match your Notion Database property names and values.

### 6. Use It!

Create a PR with a title like:
```
[TASK-123] Add authentication feature
```

Or link multiple tasks:
```
[TASK-123][TASK-456] Fix authentication and update docs
```

The workflow will:
- Add PR link to the specified Notion task(s)
- Post a comment on the PR with link(s) to Notion page(s)
- Update status to "Done" when merged (for all linked tasks)

## Multiple Task Support

You can link a single PR to multiple Notion tasks by including multiple task IDs in the PR title:

```
[TASK-123][TASK-456][TASK-789] Implement feature across multiple components
```

### How it works:

**When PR is created:**
- PR link is added to all specified tasks (TASK-123, TASK-456, TASK-789)
- PR comment shows links to all Notion pages

**When title is edited:**
- Removing `[TASK-123]` removes the PR link from that task
- Adding `[TASK-999]` adds the PR link to that task
- Existing links remain unchanged

**When PR is merged:**
- All tasks in the title are updated to "Done" status

**Example workflow:**
```
1. Create PR: [TASK-123][TASK-456] Feature implementation
   → Links added to both tasks

2. Edit title: [TASK-123][TASK-789] Feature implementation
   → Link removed from TASK-456
   → Link added to TASK-789
   → TASK-123 link unchanged

3. Merge PR
   → Both TASK-123 and TASK-789 marked as "Done"
```

## Configuration

### Action Inputs

All settings can be customized via action inputs:

| Input | Required | Description |
|-------|----------|-------------|
| `notion_token` | Yes | Notion Integration Token |
| `notion_database_id` | Yes | Notion Database ID |
| `notion_id_property` | Yes | Property name for task ID (Unique ID type) |
| `notion_status_property` | Yes | Status property name (Status type, not Select) |
| `notion_status_value_done` | No | Status value to set when PR is merged (leave empty to disable status update) |
| `notion_pr_property` | Yes | Property name for PR links (Rich text type) |
| `pr_title_prefix` | Yes | Prefix in PR title (e.g., `TASK` for `[TASK-123]`) |

Example with custom configuration:

```yaml
- uses: isoppp/github-pr-sync-with-notion@v0.1.0
  with:
    notion_token: ${{ secrets.NOTION_TOKEN }}
    notion_database_id: ${{ secrets.NOTION_DATABASE_ID }}
    notion_id_property: 'TaskID'
    notion_status_property: 'State'
    notion_status_value_done: 'Completed'
    notion_pr_property: 'Pull Request'
    pr_title_prefix: 'PROJECT'
```

To disable status updates when PR is merged:

```yaml
- uses: isoppp/github-pr-sync-with-notion@v0.1.0
  with:
    notion_token: ${{ secrets.NOTION_TOKEN }}
    notion_database_id: ${{ secrets.NOTION_DATABASE_ID }}
    notion_id_property: 'ID'
    notion_status_property: 'Status'
    notion_status_value_done: ''  # Empty value disables status update
    notion_pr_property: 'GitHub PR'
    pr_title_prefix: 'TASK'
```

### Multiple Projects

You can use the same Notion Integration for multiple projects:
- Use the same `notion_token` secret
- Use different `notion_database_id` for each project
- Use different `pr_title_prefix` to distinguish projects

## Troubleshooting

### Error: `validation_error: Could not find property with name or id: unique_id`

**Solution**: Update `NOTION_ID_PROPERTY` to match your actual property name in Notion Database.

### Error: `Resource not accessible by integration`

**Solution**: Make sure your Integration is connected to the Database (... menu → Add connections)

## License

MIT License - see the [LICENSE](LICENSE) file for details

---

**Made with ❤️ by developers who love Notion and GitHub**