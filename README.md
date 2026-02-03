# Petaverse GitHub Actions

A collection of reusable GitHub Actions for Petaverse projects.

## Available Actions

### Feishu Notification

Send build notifications to Feishu (Lark) with professional formatting.

**Location**: `feishu-notification/`

**Usage**:
```yaml
- uses: petaverse-cloud/pv-github-actions/feishu-notification@main
  with:
    webhook_url: ${{ secrets.FEISHU_WEBHOOK }}
    status: 'started'
    project_name: 'My Project'
```

[Read more](feishu-notification/README.md)

## Contributing

1. Create a new action directory under the repository root
2. Add `action.yml` with metadata and implementation
3. Create a `README.md` with usage instructions and examples
4. Update this main README with a link to the new action

## License

MIT
