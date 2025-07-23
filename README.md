# Better Stack Telemetry API OpenAPI Specification

A comprehensive OpenAPI 3.0.3 specification for the Better Stack Telemetry API, providing complete documentation for log management, metrics, and data querying capabilities.

## 📋 Overview

Better Stack Telemetry (formerly Logtail) is an OpenTelemetry-native infrastructure monitoring platform that provides comprehensive log management, metrics, and tracing capabilities. This specification documents all available API endpoints for programmatic access to your telemetry data.

## 🚀 Features

### **Complete API Coverage**
- **Sources Management**: Create, list, get, update, and delete log sources
- **Source Groups Management**: Organize sources into logical groups
- **Metrics Management**: Create custom metrics and manage existing ones
- **Log Ingestion**: Real-time log data ingestion via HTTP
- **Data Querying**: Direct ClickHouse SQL queries for comprehensive analysis

### **Modern Architecture**
- **Multi-region Support**: Query endpoints available across multiple regions
- **Multiple Authentication Methods**: Bearer tokens, Basic auth, and Source tokens
- **Flexible Data Formats**: JSON, MessagePack, NDJSON, CSV, TSV support
- **High Performance**: Direct ClickHouse access for fast queries
- **Comprehensive Error Handling**: Detailed error responses with examples

## 📖 API Endpoints

### Sources (5 endpoints)
- `GET /sources` - List all sources with pagination
- `POST /sources` - Create a new log source
- `GET /sources/{source_id}` - Get detailed source information
- `PATCH /sources/{source_id}` - Update source configuration
- `DELETE /sources/{source_id}` - Delete a source

### Source Groups (5 endpoints)
- `GET /source-groups` - List all source groups with pagination
- `POST /source-groups` - Create a new source group
- `GET /source-groups/{source_group_id}` - Get detailed source group information
- `PATCH /source-groups/{source_group_id}` - Update source group
- `DELETE /source-groups/{source_group_id}` - Delete a source group

### Metrics (3 endpoints)
- `GET /sources/{source_id}/metrics` - List metrics for a source
- `POST /sources/{source_id}/metrics` - Create a new metric
- `DELETE /sources/{source_id}/metrics/{metric_id}` - Delete a metric

### Log Ingestion (1 endpoint)
- `POST /` - Ingest log data (single events or batches)

### Data Querying (1 endpoint)
- `POST /` - Connect Remotely HTTP API for direct ClickHouse SQL queries

## 🔐 Authentication

The API supports multiple authentication methods:

### 1. Bearer Authentication (API Operations)
```bash
Authorization: Bearer YOUR_API_TOKEN
```
- Used for managing sources, source groups, and metrics
- Get tokens from Better Stack → API tokens

### 2. Source Token Authentication (Log Ingestion)
```bash
Authorization: Bearer YOUR_SOURCE_TOKEN
```
- Used specifically for log ingestion
- Obtained when creating a source

### 3. Basic Authentication (Connect Remotely)
```bash
Authorization: Basic base64(username:password)
```
- Used for querying data via Connect Remotely HTTP API
- Create credentials in dashboard under "Connect remotely"

## 🌍 Multi-Region Support

The Connect Remotely HTTP API is available across multiple regions:

- **Europe**: `eu-nbg-2-connect.betterstackdata.com`
- **US Central**: `us-chi-1-connect.betterstackdata.com`  
- **Asia Pacific**: `ap-sin-1-connect.betterstackdata.com`

## 📚 Usage Examples

### Creating a Source
```bash
curl -X POST https://telemetry.betterstack.com/api/v1/sources \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production API Logs",
    "platform": "http",
    "data_region": "us_east"
  }'
```

### Ingesting Log Data
```bash
curl -X POST https://YOUR_INGESTING_HOST \
  -H "Authorization: Bearer YOUR_SOURCE_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "User login successful",
    "level": "info",
    "user_id": "12345"
  }'
```

### Querying Data with SQL
```bash
curl -X POST https://eu-nbg-2-connect.betterstackdata.com \
  -u "username:password" \
  -H "Content-Type: text/plain" \
  -d "SELECT dt, raw FROM remote(t123456_your_source_logs) 
      WHERE dt >= now() - INTERVAL 1 HOUR 
      ORDER BY dt DESC 
      LIMIT 100 
      FORMAT JSONEachRow"
```

### Creating a Custom Metric
```bash
curl -X POST https://telemetry.betterstack.com/api/v2/sources/123/metrics \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "response_time",
    "sql_expression": "getJSON(raw, '\''message_json.duration_ms'\'')",
    "type": "float64_delta",
    "aggregations": ["avg", "p95", "min", "max"]
  }'
```

## 🛠️ Development Tools

### OpenAPI Specification File
- **File**: `better-stack-telemetry-api.yaml`
- **Format**: OpenAPI 3.0.3
- **Size**: 15 endpoints, comprehensive schemas and examples

### Using the Specification
1. **API Clients**: Generate client libraries using OpenAPI generators
2. **Documentation**: Use tools like Swagger UI or Redoc for interactive docs
3. **Testing**: Import into Postman, Insomnia, or similar tools
4. **Validation**: Validate requests/responses against the schema

### Validation Tools
```bash
# Using swagger-codegen
swagger-codegen validate -i better-stack-telemetry-api.yaml

# Using openapi-generator
openapi-generator validate -i better-stack-telemetry-api.yaml
```

## 📊 Query Capabilities

The Connect Remotely HTTP API provides powerful querying capabilities:

### Data Types Available
- **Logs**: `remote(t{source_id}_{source_name}_logs)`
- **Metrics**: `t{source_id}_{source_name}_metrics`  
- **Spans**: Tracing data tables
- **Archive Data**: S3-archived data via `s3Cluster()` function

### Query Best Practices
- Always use `LIMIT` to control result size
- Use `ORDER BY` for consistent results
- Apply time range filters for better performance
- Use shorter time ranges when possible
- Handle errors gracefully in your applications

### Output Formats
- **JSON**: Structured JSON response
- **JSONEachRow**: Newline-delimited JSON (recommended for large results)
- **CSV**: Comma-separated values
- **TSV**: Tab-separated values
- **Pretty**: Human-readable format

## 🔧 Configuration Options

### Source Configuration
Sources support various configuration options:
- **Platform Types**: http, nginx, docker, kubernetes, apache, mysql, postgresql, redis, mongodb, nodejs, python, ruby, php, go, java, dotnet
- **Data Regions**: us_east, eu_west, ap_southeast
- **Retention Periods**: Configurable for both logs and metrics (1-365 days)
- **VRL Transformations**: Vector Remap Language for data processing
- **Scraping**: URL scraping with customizable headers and authentication

### Metric Types
- **string_low_cardinality**: For categorical data
- **int64_delta**: For integer counters
- **float64_delta**: For decimal counters  
- **datetime64_delta**: For timestamp differences
- **boolean**: For true/false values

## 🐛 Error Handling

The API provides comprehensive error responses:

### HTTP Status Codes
- **200/201**: Success
- **204**: Success (empty response)
- **400**: Bad Request (invalid query syntax)
- **401**: Unauthorized (invalid/missing authentication)
- **403**: Forbidden (access denied)
- **404**: Not Found (resource doesn't exist)
- **406**: Not Acceptable (invalid format)
- **413**: Payload Too Large (request exceeds limits)
- **422**: Validation Error (invalid parameters)
- **504**: Gateway Timeout (query timeout)

### Error Response Format
```json
{
  "error": "Invalid request format",
  "details": "The request body contains invalid JSON"
}
```

## 📈 Rate Limits and Constraints

### Log Ingestion
- **Maximum request size**: 10 MiB
- **Recommended log record size**: Under 100 KiB
- **Supported formats**: JSON, MessagePack, NDJSON

### Pagination
- **Default page size**: 25 items
- **Maximum page size**: 50 items
- **Available for**: Sources, Source Groups, Metrics listing

### Query Limits
- Use `LIMIT` clause to control result size
- Query timeout: Varies by complexity and data size
- Best performance with time-based filtering

## 🤝 Contributing

This OpenAPI specification was created by analyzing the official Better Stack Telemetry documentation. If you find any discrepancies or have suggestions for improvements:

1. Check the [Better Stack Documentation](https://betterstack.com/docs/logs/)
2. Open an issue describing the problem
3. Submit a pull request with corrections

## 📄 License

This OpenAPI specification is provided for developer convenience. Better Stack Telemetry is subject to Better Stack's Terms of Service.

## 🆘 Support

For API support and questions:
- **Email**: hello@betterstack.com
- **Documentation**: https://betterstack.com/docs/logs/
- **Community**: Better Stack Community resources

---

**Generated with ❤️ for the developer community**