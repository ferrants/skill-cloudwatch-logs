# skill-cloudwatch-logs

A [Claude Code plugin](https://code.claude.com/docs/en/plugins) that teaches Claude how to query and tail AWS CloudWatch logs — discovering log groups, filtering events, running Insights queries, and debugging production issues.

## Install

```bash
claude plugin install cloudwatch-logs --url https://github.com/ferrants/skill-cloudwatch-logs
```

Or for a specific project only:

```bash
claude plugin install cloudwatch-logs --url https://github.com/ferrants/skill-cloudwatch-logs --scope project
```

## What it does

Once installed, Claude gains the `/cloudwatch_logs` skill and can automatically:

- **Discover log groups** — find the right log group by prefix, keyword, or recent activity
- **Search and filter** — query log events by text, JSON fields, or time range
- **Run Insights queries** — use CloudWatch Logs Insights for aggregations and complex analysis
- **Tail logs** — follow logs in real-time with optional filters

This is useful when you need Claude to debug production issues, investigate errors, or monitor services running in AWS.

## Usage

Invoke the skill directly:

```
/cloudwatch_logs
```

Or Claude will invoke it automatically when the task involves CloudWatch logs or production debugging.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed and configured with appropriate permissions

## License

MIT
