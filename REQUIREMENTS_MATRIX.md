# Splunk MCP Requirements Matrix

## Overview

This document maps the planned MCP (Model Context Protocol) functions to their corresponding Splunk REST API endpoints, based on Splunk's REST API documentation and MCP specification.

## MCP Functions vs Splunk REST API Mapping

### Function 1: splunk_run_query
**Purpose**: Execute SPL queries and return results

**Splunk REST API Endpoints**:
- Primary: `/services/search/jobs` (POST)
  - Create and dispatch a search job
  - Parameters: `search` (SPL query), `exec_mode`, `earliest_time`, `latest_time`, etc.
- Secondary: `/services/search/jobs/{sid}` (GET)
  - Get job status and metadata
- Tertiary: `/services/search/jobs/{sid}/results` (GET)
  - Retrieve search results (supports `output_mode=json`, `csv`, `xml`)
  - Parameters: `offset`, `count` (pagination)

**Response Types**:
- Job creation: job SID, required poll interval
- Results: Data rows, field names, result count

---

### Function 2: splunk_get_info
**Purpose**: Retrieve Splunk instance information

**Splunk REST API Endpoints**:
- Primary: `/services/server/info` (GET)
  - Returns server version, build, OS, server name
- Secondary: `/services/licenser/license` (GET)
  - Returns licensing information
- Alternative: `/services/licenser/slaves` (GET)
  - For distributed deployments

**Response Types**:
- Server info: version, build, processor, os, server_name
- License: type, expiration, quota, usage

---

### Function 3: splunk_get_indexes
**Purpose**: List all available indexes

**Splunk REST API Endpoints**:
- Primary: `/services/data/indexes` (GET)
  - List all indexes with basic metadata
  - Returns: name, total event count, size, frozen path, etc.
- Secondary: `/services/data/indexes/{index_name}` (GET)
  - Get detailed information about a specific index

**Response Types**:
- Index summary: name, totalEventCount, currentDBSizeMB, maxTotalDataSizeMB
- Index details: retention policy, bucket paths, REPLICATION settings

---

### Function 4: splunk_get_index_info
**Purpose**: Get detailed information about a specific index

**Splunk REST API Endpoints**:
- Primary: `/services/data/indexes/{index_name}` (GET)
  - Comprehensive index configuration
- Secondary: `/services/data/indexes/{index_name}/status` (GET)
  - Current index status and statistics
- Tertiary: `/services/data/indexes/{index_name}/ buckets` (GET)
  - Index buckets information

**Response Types**:
- Configuration: retention days, max data size, home/ cold/frozen paths
- Status: event count, size on disk, bucket count

---

### Function 5: splunk_get_metadata
**Purpose**: Retrieve metadata about hosts, sources, and sourcetypes

**Splunk REST API Endpoints**:
- **For hosts**:
  - `/services/data/indexes/{index}/hosts` (GET)
  - `/services/data/indexes/{index}/hosts/{host}` (GET)
- **For sources**:
  - `/services/data/indexes/{index}/sources` (GET)
  - `/services/data/indexes/{index}/sources/{source}` (GET)
- **For sourcetypes**:
  - `/services/data/indexes/{index}/sourcetypes` (GET)
  - `/services/data/indexes/{index}/sourcetypes/{sourcetype}` (GET)

**Alternative approach via SPL**:
- Use `| metadata index=<index> type=hosts|sources|sourcetypes` via `/services/search/jobs`

**Response Types**:
- Hosts/sources/sourcetypes: name, count, lastTime, firstTime

---

### Function 6: splunk_get_knowledge_objects
**Purpose**: Retrieve knowledge objects (saved searches, dashboards, macros, data models)

**Splunk REST API Endpoints**:
- **Saved Searches**:
  - `/servicesNS/{user}/{app}/saved/searches` (GET)
  - `/servicesNS/{user}/{app}/saved/searches/{name}` (GET)
- **Dashboards**:
  - `/servicesNS/{user}/{app}/data/ui/views` (GET)
  - Includes dashboards and forms
- **Macros**:
  - `/servicesNS/{user}/{app}/admin/macros` (GET)
- **Data Models**:
  - `/servicesNS/{user}/{app}/datamodel` (GET)
  - `/servicesNS/{user}/{app}/datamodel/{model_name}` (GET)

**Response Types**:
- Saved searches: name, search query, schedule, actions
- Dashboards: name, dashboard XML/JSON, layout
- Macros: name, definition, isEval
- Data models: name, description, fields, constraints

---

### Function 7: splunk_get_user_info
**Purpose**: Get current user's roles, capabilities, and permissions

**Splunk REST API Endpoints**:
- Primary: `/services/authentication/current-context` (GET)
  - Returns current user, roles, and capabilities
- Secondary: `/services/authentication/users/{username}` (GET)
  - Detailed user information (requires admin)
- Tertiary: `/services/authorization/roles/{role}` (GET)
  - Role-specific capabilities (multiple calls for all roles)

**Response Types**:
- Current context: username, roles[], capabilities[]
- User info: realname, email, type, roles
- Role info: capabilities[], imported_roles[], concrete capabilities

---

## MCP Protocol Considerations

### MCP Tool Schema Requirements
Each MCP tool will need:
- `name`: Unique identifier (e.g., "splunk_run_query")
- `description`: Clear explanation of function
- `inputSchema`: JSON Schema for parameters
- `outputSchema`: Structured response format

### Authentication Methods
- **Token-based**: Splunk authentication token in `Authorization: Bearer <token>` header
- **Credential-based**: Username/password via `/services/auth/login` to obtain session token

### Rate Limiting Considerations
- Splunk REST API may impose rate limits
- Implement exponential backoff for 429/503 responses
- Consider Splunk's `max_concurrent` settings for search jobs

### Error Handling
- HTTP status codes: 400 (bad request), 401 (unauthorized), 403 (forbidden), 404 (not found), 409 (conflict), 500 (server error)
- Splunk-specific errors in response body with `messages` array
- Include error codes: `ERROR`, `WARN`, `INFO`

---

## Implementation Notes

### Authentication Flow
1. Obtain Splunk authentication token (either provided or via login)
2. Store token securely, refresh if needed
3. Include token in all API requests: `Authorization: Bearer <token>`

### Request Format
- Base URL: `https://<splunk-host>:8089`
- Content-Type: `application/json` or `application/x-www-form-urlencoded`
- Accept header: `Accept: application/json`

### Data Pagination
- Search results: Use `offset` and `count` parameters
- Index lists: Use `count` and `offset` for large deployments
- Implement cursor-based pagination for large result sets

### Timeouts
- Search jobs: Consider `execution_timeout` (default 60s)
- API requests: Recommended 30s timeout for most calls
- Job polling: Interval based on `sid` status (usually 1-5 seconds)

---

## References

- Splunk REST API Developer Guide: https://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTprolog
- Splunk REST API Reference: https://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTsearch
- Model Context Protocol Specification: https://modelcontextprotocol.io/specification
- FastMCP Library: https://github.com/modelcontextprotocol/fastmcp

---

*Generated: 2026-04-05*
*Status: Ready for review*
