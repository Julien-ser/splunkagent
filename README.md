# Splunk MCP Agent

[![Build Status](https://github.com/anomalyco/splunkagent/workflows/Test%20%26%20Validate/badge.svg)](https://github.com/anomalyco/splunkagent/actions)
[![Coverage](https://img.shields.io/badge/coverage-0%25-red)](https://github.com/anomalyco/splunkagent/actions)
[![Version](https://img.shields.io/badge/version-0.1.0--dev-blue)](https://github.com/anomalyco/splunkagent/releases)

A Model Context Protocol (MCP) server that provides AI assistants with secure, structured access to Splunk instances. This agent bridges the gap between LLMs and Splunk's powerful search and analytics capabilities.

## Features

- Execute SPL queries with pagination and export options
- Retrieve Splunk instance information and licensing
- Discover and explore indexes with metadata
- Access knowledge objects (saved searches, dashboards, macros, data models)
- User permission introspection
- Token-based and credential-based authentication

## Quick Start

### Installation

```bash
pip install splunkagent
```

### Configuration

Create a `configuration.toml` file:

```toml
[splunk]
host = "splunk-instance.example.com"
port = 8089
auth_type = "token"  # or "credentials"
token = "your-splunk-token"  # if using token auth
# OR
username = "admin"
password = "your-password"   # if using credentials

[mcp]
transport = "stdio"  # or "sse"
host = "localhost"
port = 8080
```

### Running the Server

```bash
# Using stdio transport (for MCP clients)
splunkagent

# Using SSE transport (web-based)
splunkagent --transport sse --host localhost --port 8080
```

## Project Status

### Phase 1: Planning & Setup

- [x] **Task 1.1**: Research Splunk MCP specifications and REST API endpoints
- [ ] **Task 1.2**: Define system architecture and technology stack
- [ ] **Task 1.3**: Initialize project repository structure
- [ ] **Task 1.4**: Establish coding standards and documentation templates

### Phase 2: Core Infrastructure

- [ ] **Task 2.1**: Implement authentication handler
- [ ] **Task 2.2**: Build Splunk client wrapper
- [ ] **Task 2.3**: Create configuration management system
- [ ] **Task 2.4**: Setup logging and error handling

### Phase 3: MCP Function Implementation

- [ ] **Task 3.1**: Implement splunk_run_query
- [ ] **Task 3.2**: Implement splunk_get_info
- [ ] **Task 3.3**: Implement splunk_get_indexes and splunk_get_index_info
- [ ] **Task 3.4**: Implement splunk_get_metadata and splunk_get_knowledge_objects
- [ ] **Task 3.5**: Implement splunk_get_user_info
- [ ] **Task 3.6**: Create MCP server integration

### Phase 4: Quality & Documentation

- [ ] **Task 4.1**: Create comprehensive test suite
- [ ] **Task 4.2**: Generate architectural diagrams
- [ ] **Task 4.3**: Write complete documentation suite
- [ ] **Task 4.4**: Create README, CHANGELOG, and SECURITY files
- [ ] **Task 4.5**: Perform security audit and performance testing
- [ ] **Task 4.6**: Package and distribution preparation

## Development

This project follows a phase-based development approach. See [TASKS.md](TASKS.md) for the complete task list and current progress.

### Prerequisites

- Python 3.11+
- Splunk instance with REST API access
- Splunk user with appropriate permissions

### Setup Development Environment

```bash
# Clone repository
git clone https://github.com/anomalyco/splunkagent.git
cd splunkagent

# Install dependencies
pip install -e .
pip install -r requirements-dev.txt

# Run tests
pytest tests/ -v
```

## Documentation

Detailed documentation available in `docs/`:

- [Getting Started Guide](docs/getting-started.md)
- [API Reference](docs/api/)
- [Configuration Options](docs/configuration.md)
- [Troubleshooting](docs/troubleshooting.md)
- [Security Considerations](docs/security.md)

## Architecture

The project consists of:

```
splunkagent/
├── src/splunkagent/
│   ├── auth.py          # Authentication handling
│   ├── client.py        # Splunk client wrapper
│   ├── config.py        # Configuration management
│   ├── logging.py       # Logging setup
│   ├── exceptions.py    # Custom exceptions
│   ├── tools/           # MCP tool implementations
│   │   ├── query.py
│   │   ├── info.py
│   │   ├── indexes.py
│   │   ├── metadata.py
│   │   └── user.py
│   └── server.py        # MCP server integration
├── tests/               # Test suite
├── docs/                # Documentation
└── diagrams/            # Architecture diagrams
```

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.
