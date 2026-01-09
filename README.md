# ReportDash Datastore MCP Server

Connect your ReportDash Datastore account to any MCP-compatible client and access your marketing analytics data through natural language queries.

## Overview

The Datastore Datastore MCP (Model Context Protocol) server provides a standardized interface for AI assistants to query your connected advertising and analytics platforms including Facebook Ads, Google Ads, Instagram, LinkedIn, TikTok, and more.

## Supported Platforms

- **Advertising**: Facebook Ads, Google Ads, Bing Ads, LinkedIn Ads, TikTok Ads, Snapchat Ads, Pinterest Ads, Amazon Ads
- **Analytics**: Google Analytics v4, Instagram Insights, Facebook Insights, YouTube Analytics, LinkedIn Analytics, Shopify Analytics
- **Marketing**: Mailchimp Insights, HubSpot Analytics, Klaviyo Analytics
- **Local**: Google Business Profile

## Installation

### Prerequisites

- MCP-compatible client (Claude Desktop, or any client supporting MCP protocol)
- Active ReportDash Datastore account at [datastore.reportdash.com](https://datastore.reportdash.com)
- API key from your ReportDash Datastore account

### Get Your API Key

1. Log in to [datastore.reportdash.com](https://datastore.reportdash.com)
2. Navigate to **Destinations → API Access**
3. Generate a new API key
4. Copy the key for configuration

### Configuration

The Datastore MCP server uses Server-Sent Events (SSE) transport over HTTPS.

**Connection Details:**
- **Base URL**: `https://datastore.reportdash.com/api/mcp/v1`
- **Transport**: SSE
- **Authentication**: API Key via custom header

**Configuration for MCP Clients:**

```json
{
  "mcpServers": {
    "datastore": {
      "url": "https://datastore.reportdash.com/api/mcp/v1",
      "transport": "sse",
      "headers": {
        "X-Api-Key": "your-api-key-here"
      }
    }
  }
}
```

**For Claude Desktop specifically:**

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`  
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

Note: Claude Desktop requires a Pro subscription to use MCP servers.

### Activation

After configuring your MCP client:
1. Save the configuration
2. Restart your MCP client
3. The Datastore tools will be available for use

## Available Tools

### Discovery Tools

#### `list_sources`
List all connected data sources in your Datastore account.

**Input Schema:**
```json
{
  "source_type": "string (optional)",
  "source_name_filter": "string (optional)",
  "limit": "integer (optional, default: 50, max: 200)",
  "cursor": "string (optional)"
}
```

**Output:**
- Array of sources with `source_id`, `source_name`, `source_type`, and `status`
- Pagination cursor for next page

---

#### `list_tables`
List available data tables across your sources.

**Input Schema:**
```json
{
  "source_id": "string (optional)",
  "source_type": "string (optional)",
  "table_name_filter": "string (optional)",
  "limit": "integer (optional, default: 50, max: 200)",
  "cursor": "string (optional)"
}
```

**Output:**
- Array of tables with `table_id`, `table_name`, `table_type`, `source_id`, `source_name`, `source_type`
- Pagination cursor for next page

---

#### `describe_table`
Get detailed schema information for a specific table.

**Input Schema:**
```json
{
  "table_id": "string (required)"
}
```

**Output:**
- Table metadata (`table_id`, `table_name`, `source_id`, `source_name`, `source_type`)
- Array of column definitions with:
  - `column_id`: Stable identifier
  - `column_name`: Human-readable name
  - `field_type`: dimension | time-dimension | metric
  - `value_type`: string | number | float | money | percentage | date | time | datetime
  - `description`: Optional column description
  - `formula`: Optional calculation formula
  - `aggregate`: Optional default aggregation function

---

### Query Tools

#### `query_table`
Run structured queries against individual source tables with filtering, grouping, and aggregation.

**Input Schema:**
```json
{
  "table_id": "string (required)",
  "query": {
    "select": ["column_id1", "column_id2"],
    "filters": [
      {
        "column_id": "string",
        "operator": "EQ | NEQ | GT | GTE | LT | LTE | IN | CONTAINS | IS_NULL | IS_NOT_NULL | BETWEEN | LAST_N_DAYS",
        "value": "any (depends on operator)"
      }
    ],
    "group_by": ["column_id1", "column_id2"],
    "aggregates": [
      {
        "fn": "SUM | AVG | MIN | MAX | COUNT | COUNT_DISTINCT",
        "column_id": "string (optional for COUNT)",
        "as": "alias_name"
      }
    ],
    "order_by": [
      {
        "column_id": "string (column_id or aggregate alias)",
        "direction": "ASC | DESC"
      }
    ],
    "limit": "integer (default: 50, max: 300)"
  }
}
```

**Output:**
- Array of column metadata
- Array of result rows (objects keyed by column IDs or aliases)
- Echo of `table_id`

**Filter Operators:**
- `EQ`, `NEQ`: Equal, not equal
- `GT`, `GTE`, `LT`, `LTE`: Comparison operators
- `IN`: Value in array
- `CONTAINS`: Substring match
- `BETWEEN`: Range [min, max]
- `IS_NULL`, `IS_NOT_NULL`: Null checks
- `LAST_N_DAYS`: Rolling date window (integer value)

**Aggregate Functions:**
- `SUM`: Sum of values
- `AVG`: Average of values
- `MIN`, `MAX`: Minimum/maximum values
- `COUNT`: Count of rows
- `COUNT_DISTINCT`: Count of unique values

---

### Unified View Tools

Views combine data from multiple sources for cross-platform or multi-account analysis.

#### `list_views`
List all available unified data views.

**Input Schema:**
```json
{
  "view_type": "source_category | source_group (optional)",
  "view_name_filter": "string (optional)",
  "limit": "integer (optional, default: 50, max: 200)",
  "cursor": "string (optional)"
}
```

**View Types:**
- `source_category`: Cross-platform views (e.g., all advertising platforms)
- `source_group`: Same-platform views (e.g., multiple Google Ads accounts)

**Output:**
- Array of views with `view_id`, `view_name`, `view_type`, `sources`, `source_types`
- Pagination cursor for next page

---

#### `describe_view`
Get schema information for a unified view.

**Input Schema:**
```json
{
  "view_id": "string (required)"
}
```

**Output:**
- View metadata (`view_id`, `view_name`, `view_type`)
- Array of sources included in the view
- Array of column definitions (similar to `describe_table`)

---

#### `query_view`
Query unified views across multiple sources or accounts.

**Input Schema:**
```json
{
  "view_id": "string (required)",
  "start_date": "string (required, YYYY-MM-DD)",
  "end_date": "string (required, YYYY-MM-DD)",
  "query": {
    "select": ["column_id1", "column_id2"],
    "filters": [
      {
        "column_id": "string",
        "operator": "EQ | NEQ | GT | GTE | LT | LTE | IN | CONTAINS | IS_NULL | IS_NOT_NULL | BETWEEN",
        "value": "any (depends on operator)"
      }
    ],
    "order_by": [
      {
        "column_id": "string",
        "direction": "ASC | DESC"
      }
    ],
    "limit": "integer (default: 50, max: 300)"
  }
}
```

**Note:** Views return row-level data and do not support grouping or aggregation.

**Output:**
- Array of column metadata
- Array of result rows
- Echo of `view_id`

---

## Data Types

### Field Types
- **dimension**: Categorical data (campaign name, country, device type)
- **time-dimension**: Date/time fields for temporal analysis
- **metric**: Quantitative measurements (spend, clicks, impressions, conversions)

### Value Types
- `string`: Text data
- `number`: Integer values
- `float`: Decimal numbers
- `money`: Currency amounts
- `percentage`: Percentage values
- `date`: Date only (YYYY-MM-DD)
- `time`: Time only
- `datetime`: Combined date and time

## Query Limits

- Maximum 300 rows per query
- Results are paginated for larger datasets
- Use specific filters and date ranges for optimal performance

## Usage Examples

### Example 1: List Facebook Ads Sources
```json
{
  "name": "list_sources",
  "arguments": {
    "source_type": "facebook_ads"
  }
}
```

### Example 2: Get Campaign Performance
```json
{
  "name": "query_table",
  "arguments": {
    "table_id": "table_12345",
    "query": {
      "select": ["campaign_name", "date"],
      "filters": [
        {
          "column_id": "date",
          "operator": "LAST_N_DAYS",
          "value": 30
        }
      ],
      "group_by": ["campaign_name"],
      "aggregates": [
        {
          "fn": "SUM",
          "column_id": "spend",
          "as": "total_spend"
        },
        {
          "fn": "SUM",
          "column_id": "clicks",
          "as": "total_clicks"
        }
      ],
      "order_by": [
        {
          "column_id": "total_spend",
          "direction": "DESC"
        }
      ],
      "limit": 10
    }
  }
}
```

### Example 3: Cross-Platform Analysis
```json
{
  "name": "query_view",
  "arguments": {
    "view_id": "view_all_ads",
    "start_date": "2026-01-01",
    "end_date": "2026-01-15",
    "query": {
      "select": ["source_type", "campaign_name", "spend", "clicks", "impressions"],
      "filters": [
        {
          "column_id": "spend",
          "operator": "GT",
          "value": 1000
        }
      ],
      "order_by": [
        {
          "column_id": "spend",
          "direction": "DESC"
        }
      ],
      "limit": 50
    }
  }
}
```

## Best Practices

1. **Discovery First**: Use `list_sources` and `list_tables` to explore available data
2. **Schema Inspection**: Always use `describe_table` or `describe_view` to understand column names and types before querying
3. **Efficient Filtering**: Apply filters early to reduce result set size
4. **Use Time Filters**: Leverage `LAST_N_DAYS` for rolling time windows
5. **Aggregate Wisely**: Use `group_by` with aggregates for summary statistics
6. **Pagination**: Handle pagination cursors for large result sets
7. **Cross-Platform**: Use views for multi-source analysis

## API Endpoints

The Datastore MCP server implements the following endpoints:

- `POST /api/mcp/v1/tools/list` - Returns list of available tools
- `POST /api/mcp/v1/tools/call` - Executes a tool with provided arguments

Both endpoints require the `X-Api-Key` header for authentication.

## Error Handling

The server returns standard MCP error responses:

```json
{
  "content": [
    {
      "type": "text",
      "text": "Error message describing what went wrong"
    }
  ],
  "isError": true
}
```

Common error scenarios:
- Invalid or missing API key
- Invalid table_id or view_id
- Malformed query structure
- Rate limit exceeded
- Insufficient permissions

## Security

- API keys are transmitted via HTTPS headers
- All queries are read-only - no data modification possible
- Data access is scoped to the authenticated account
- API keys can be revoked at any time in Datastore settings
- Keys should be stored securely and never committed to version control

## Rate Limits

- Standard rate limits apply per API key
- Contact ReportDash support for enterprise rate limits
- Use pagination and efficient queries to stay within limits

## Troubleshooting

### Authentication Issues
- Verify API key is correct and active
- Check that `X-Api-Key` header is being sent
- Regenerate key if necessary in Datastore settings

### Query Errors
- Use `describe_table` to verify column IDs
- Ensure column types match filter operators
- Check date formats are YYYY-MM-DD
- Verify table_id or view_id exists

### Connection Issues
- Confirm base URL is correct
- Check network connectivity
- Verify MCP client supports SSE transport
- Review MCP client logs for details

## Support

- **Documentation**: [datastore.reportdash.com/docs](https://datastore.reportdash.com/docs)
- **API Access**: [datastore.reportdash.com](https://datastore.reportdash.com) → Destinations → API Access
- **Support**: Contact ReportDash support team

## MCP Protocol Compliance

This server implements the Model Context Protocol specification:
- Protocol version: 2024-11-05
- Transport: Server-Sent Events (SSE)
- JSON-RPC 2.0 message format
- Standard MCP tool schema

## Contributing

For issues, feature requests, or contributions related to the Datastore MCP server, please contact ReportDash support or visit the documentation.

## License

MIT

---

**Getting Started:**
1. Sign up at [datastore.reportdash.com](https://datastore.reportdash.com)
2. Connect your advertising and analytics accounts
3. Generate an API key from Destinations → API Access
4. Configure your MCP client with the connection details above
5. Start querying your marketing data!