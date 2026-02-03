# Feishu Notification Action

Send professional build notifications to Feishu (Lark) with rich formatting.

## Features

- 🚀 Build start notifications
- ✅ Build success notifications with build duration and image info
- ❌ Build failure notifications with error logs link
- 🔗 Direct links to commits and workflow runs
- 📊 Rich post format with structured information

## Usage

### Basic Example

```yaml
- name: Notify build start
  uses: petaverse-cloud/pv-github-actions/feishu-notification@main
  with:
    webhook_url: ${{ secrets.FEISHU_WEBHOOK }}
    status: 'started'
    project_name: 'My Project'
```

### Complete Build Workflow Example

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Notify build start
        uses: petaverse-cloud/pv-github-actions/feishu-notification@main
        with:
          webhook_url: ${{ secrets.FEISHU_WEBHOOK }}
          status: 'started'
          project_name: 'My Project'
          commit_sha: ${{ github.sha }}

      - name: Build Docker image
        run: |
          docker build -t my-image:latest .

      - name: Notify build success
        if: success()
        uses: petaverse-cloud/pv-github-actions/feishu-notification@main
        with:
          webhook_url: ${{ secrets.FEISHU_WEBHOOK }}
          status: 'success'
          project_name: 'My Project'
          commit_sha: ${{ github.sha }}
          commit_message: ${{ github.event.head_commit.message }}
          start_time: ${{ steps.start_time.outputs.time }}
          build_duration: '120s'
          image_info: 'my-image:latest'

      - name: Notify build failure
        if: failure()
        uses: petaverse-cloud/pv-github-actions/feishu-notification@main
        with:
          webhook_url: ${{ secrets.FEISHU_WEBHOOK }}
          status: 'failure'
          project_name: 'My Project'
          commit_sha: ${{ github.sha }}
          start_time: ${{ steps.start_time.outputs.time }}
```

### Advanced Example with Build Tracking

```yaml
name: Build and Push to ACR

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Record start time
        id: start_time
        run: echo "time=$(date -u +'%Y-%m-%d %H:%M:%S')" >> $GITHUB_OUTPUT

      - name: Get short SHA
        id: sha
        run: echo "short=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

      - name: Notify build start
        uses: petaverse-cloud/pv-github-actions/feishu-notification@main
        with:
          webhook_url: ${{ secrets.FEISHU_WEBHOOK }}
          status: 'started'
          project_name: 'My API Backend'
          commit_sha: ${{ steps.sha.outputs.short }}

      - name: Login to Azure Container Registry
        uses: docker/login-action@v3
        with:
          registry: myregistry.azurecr.io
          username: ${{ secrets.ACR_USERNAME }}
          password: ${{ secrets.ACR_PASSWORD }}

      - name: Build and push
        id: build
        run: |
          BUILD_START=$(date +%s)

          docker build -t myregistry.azurecr.io/my-image:${{ steps.sha.outputs.short }} .
          docker tag myregistry.azurecr.io/my-image:${{ steps.sha.outputs.short }} \
                     myregistry.azurecr.io/my-image:latest
          docker push myregistry.azurecr.io/my-image:${{ steps.sha.outputs.short }}
          docker push myregistry.azurecr.io/my-image:latest

          BUILD_END=$(date +%s)
          BUILD_DURATION=$((BUILD_END - BUILD_START))
          echo "duration=${BUILD_DURATION}s" >> $GITHUB_OUTPUT
          echo "image=myregistry.azurecr.io/my-image:${{ steps.sha.outputs.short }}" >> $GITHUB_OUTPUT

      - name: Notify build success
        if: success()
        uses: petaverse-cloud/pv-github-actions/feishu-notification@main
        with:
          webhook_url: ${{ secrets.FEISHU_WEBHOOK }}
          status: 'success'
          project_name: 'My API Backend'
          commit_sha: ${{ steps.sha.outputs.short }}
          commit_message: ${{ github.event.head_commit.message }}
          start_time: ${{ steps.start_time.outputs.time }}
          build_duration: ${{ steps.build.outputs.duration }}
          image_info: ${{ steps.build.outputs.image }}

      - name: Notify build failure
        if: failure()
        uses: petaverse-cloud/pv-github-actions/feishu-notification@main
        with:
          webhook_url: ${{ secrets.FEISHU_WEBHOOK }}
          status: 'failure'
          project_name: 'My API Backend'
          commit_sha: ${{ steps.sha.outputs.short }}
          start_time: ${{ steps.start_time.outputs.time }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `webhook_url` | ✅ | - | Feishu webhook URL |
| `status` | ✅ | - | Build status: `started`, `success`, or `failure` |
| `project_name` | ✅ | - | Project name to display in notification |
| `run_number` | ❌ | `${{ github.run_number }}` | GitHub Actions run number |
| `actor` | ❌ | `${{ github.actor }}` | User who triggered the workflow |
| `branch` | ❌ | `${{ github.ref_name }}` | Git branch name |
| `commit_sha` | ❌ | - | Git commit SHA (short) |
| `commit_message` | ❌ | - | Git commit message |
| `start_time` | ❌ | - | Build start time |
| `build_duration` | ❌ | - | Build duration (e.g., 120s) |
| `image_info` | ❌ | - | Docker image information |
| `workflow_url` | ❌ | Auto-generated | GitHub Actions workflow URL |
| `commit_url` | ❌ | Auto-generated | Git commit URL |

## Notification Formats

### Build Started
```
🚀 My Project Build #123 Started

User: wezchow
Project: My Project
Branch: main
Start Time: 2026-02-03 14:30:00

Code: main abc1234 [link]
Status: Building...
```

### Build Succeeded
```
✅ My Project Build #123 Succeeded

User: wezchow
Project: My Project
Branch: main
Start Time: 2026-02-03 14:30:00
Completion Time: 2026-02-03 14:32:00
Duration: 120s

Code: main abc1234 [link]
Commit Message: feat: Add new feature

Image: myregistry.azurecr.io/my-image:abc1234

Status: ✅ Pushed to ACR

[View Workflow]
```

### Build Failed
```
❌ My Project Build #123 Failed

User: wezchow
Project: My Project
Branch: main
Start Time: 2026-02-03 14:30:00
Failure Time: 2026-02-03 14:31:00

Code: main abc1234 [link]

Status: ❌ Build or push failed

[View Error Logs]
```

## Setup

1. Create a Feishu bot webhook:
   - Open your Feishu group
   - Click "..." → "Settings" → "Bots" → "Add Bot"
   - Select "Custom Bot" and configure
   - Copy the webhook URL

2. Add the webhook to GitHub Secrets:
   - Go to your repository → Settings → Secrets and variables → Actions
   - Click "New repository secret"
   - Name: `FEISHU_WEBHOOK`
   - Value: Your webhook URL (e.g., `https://open.feishu.cn/open-apis/bot/v2/hook/xxx`)

3. Use the action in your workflow (see examples above)

## Tips

- **Track build duration**: Record start time in a step output and calculate duration
- **Use short SHA**: `git rev-parse --short HEAD` for cleaner notifications
- **Always notify on failure**: Use `if: failure()` to ensure error notifications are sent
- **Use descriptive project names**: Help team members quickly identify which project is building

## Troubleshooting

### Notification not received

1. Check webhook URL is correct in GitHub Secrets
2. Verify the Feishu bot is still in the group
3. Check GitHub Actions logs for curl errors

### Notification format broken

Ensure all input values are properly escaped, especially `commit_message` which may contain special characters.

## License

MIT
