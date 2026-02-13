---
name: cloudwatch_logs
description: "Query and tail AWS CloudWatch logs for debugging and monitoring using the AWS CLI."
---

# CloudWatch Logs Skill

Use AWS CLI to query, search, and tail CloudWatch Logs.

## Finding Log Groups

Before querying, discover the right log group:

```bash
# List all log groups (most recently active first)
aws logs describe-log-groups --order-by LastEventTime --descending --query 'logGroups[*].[logGroupName,storedBytes]' --output table

# Search for log groups by name prefix
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/" --query 'logGroups[*].logGroupName' --output table

# Search for log groups matching a keyword
aws logs describe-log-groups --query 'logGroups[?contains(logGroupName, `my-service`)].logGroupName' --output table

# See recent log streams in a group (helps confirm it's the right one)
aws logs describe-log-streams --log-group-name "$LOG_GROUP" --order-by LastEventTimestamp --descending --limit 5 --query 'logStreams[*].[logStreamName,lastEventTimestamp]' --output table
```

In the examples below, replace `$LOG_GROUP` with the actual log group name and `$REGION` with the AWS region (e.g. `us-west-2`).

## Basic Queries

### Get recent logs (last 15 minutes)

```bash
aws logs filter-log-events \
  --log-group-name "$LOG_GROUP" \
  --start-time $(date -d '15 minutes ago' +%s000) \
  --region $REGION
```

### Search for specific text

```bash
aws logs filter-log-events \
  --log-group-name "$LOG_GROUP" \
  --filter-pattern "ERROR" \
  --start-time $(date -d '1 hour ago' +%s000) \
  --region $REGION
```

### Search for a JSON field value

```bash
aws logs filter-log-events \
  --log-group-name "$LOG_GROUP" \
  --filter-pattern '"user_id": "12345"' \
  --start-time $(date -d '1 hour ago' +%s000) \
  --region $REGION
```

### Custom time ranges

```bash
# Last N hours
--start-time $(date -d 'N hours ago' +%s000)

# Specific date/time range
--start-time $(date -d '2025-01-15 14:00:00' +%s000)
--end-time $(date -d '2025-01-15 15:00:00' +%s000)

# Last N days
--start-time $(date -d 'N days ago' +%s000)
```

## CloudWatch Logs Insights (Advanced Queries)

For complex queries, use CloudWatch Logs Insights:

```bash
aws logs start-query \
  --log-group-name "$LOG_GROUP" \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 50' \
  --region $REGION

# Get query results (use query-id from start-query response)
aws logs get-query-results --query-id "QUERY_ID" --region $REGION
```

### Useful Insights Queries

```
# Count errors by type
fields @timestamp, @message
| filter @message like /ERROR/
| stats count(*) by @message
| sort count desc
| limit 20

# Find slow requests (if timing is logged)
fields @timestamp, @message
| filter @message like /duration/
| sort @timestamp desc
| limit 100

# Group by log stream (e.g. Lambda instance)
fields @timestamp, @logStream, @message
| filter @message like /ERROR/
| stats count(*) by @logStream
| sort count desc
```

## Filter Pattern Syntax

- **Simple text**: `"ERROR"` or `ERROR`
- **Multiple terms (AND)**: `"ERROR" "timeout"`
- **Exclude term**: `"ERROR" -"DEBUG"`
- **JSON field match**: `{ $.level = "ERROR" }`
- **JSON field contains**: `{ $.message = "*timeout*" }`

## Tail Logs in Real-Time

```bash
# Requires aws-cli v2
aws logs tail "$LOG_GROUP" --follow --region $REGION

# With filter
aws logs tail "$LOG_GROUP" --filter-pattern "ERROR" --follow --region $REGION
```

## Formatted Output

```bash
# Extract just messages with JMESPath
aws logs filter-log-events \
  --log-group-name "$LOG_GROUP" \
  --filter-pattern "ERROR" \
  --start-time $(date -d '1 hour ago' +%s000) \
  --region $REGION \
  --query 'events[*].[timestamp,message]' \
  --output text
```

## Useful Options

- `--limit N` — max events returned (default 10000)
- `--output text` — plain text instead of JSON
- `--query 'events[*].message'` — JMESPath to extract fields
- `--next-token` — paginate through large result sets

## Tips

- Use millisecond timestamps for `filter-log-events` (`+%s000`)
- Use second timestamps for `start-query` (`+%s`)
- Logs Insights queries are more powerful but async — poll with `get-query-results`
- For large time ranges, results may be paginated — use `--next-token` to continue
- Always specify `--region` if the log group is not in your default region
