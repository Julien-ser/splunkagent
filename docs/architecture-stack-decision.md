# Technology Stack Decision Document

## Overview

This document records the technology choices for the Splunk MCP Agent project, including rationale, alternatives considered, and key implementation details.

## 1. Implementation Language: Python

### Decision
**Python 3.11+** as the primary implementation language.

### Rationale
- **Splunk SDK Availability**: Official Splunk SDK for Python is mature, well-documented, and actively maintained
- **Rapid Development**: Python's concise syntax and extensive standard library enable faster prototyping and development
- **MCP Ecosystem**: FastMCP (Model Context Protocol) library is Python-native, providing excellent integration
- **DevOps Friendly**: Python integrates well with CI/CD pipelines, containerization, and deployment tooling
- **Community & Support**: Large community, abundant documentation, and extensive third-party libraries
- **Type Safety**: Modern Python (3.11+) supports strong typing with mypy/pyright for better code quality

### Alternatives Considered
- **Go**: Excellent performance but Splunk SDK less mature, MCP libraries fewer
- **Node.js/TypeScript**: Good MCP libraries exist, but Splunk SDK is weaker than Python version
- **Rust**: Maximum performance but steeper learning curve and longer development time

### Trade-offs
- **Performance**: Python is slower than compiled languages, but Splunk operations are I/O-bound (network calls), so language performance less critical
- **Packaging**: Python packaging can be complex, but tools like `uv` and `pyproject.toml` simplify this

---

## 2. MCP Framework: FastMCP

### Decision
**FastMCP** (https://github.com/modelcontextprotocol/fastmcp) as the MCP server implementation.

### Rationale
- **Python Native**: Written in Python, perfect for our language choice
- **Simple API**: Decorator-based tool registration, intuitive interface
- **Multiple Transports**: Supports stdio (for local AI assistants) and SSE (for web-based clients)
- **Schema Validation**: Automatic JSON Schema generation from type hints
- **Active Development**: Part of the official MCP reference implementation, well-maintained
- **Extensible**: Easy to customize server behavior and middleware

### Example Usage Pattern
```python
from mcp import FastMCP

mcp = FastMCP("splunkagent")

@mcp.tool()
def splunk_run_query(query: str, params: dict = None) -> dict:
    """Execute SPL query and return results"""
    # Implementation
    pass

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### Alternatives Considered
- **MCP Go SDK**: Would require Go implementation, incompatible with Python Splunk SDK
- **JavaScript/TypeScript MCP**: Would require Node.js, less optimal Splunk integration
- **Custom Implementation**: Too much work, reinventing the wheel

---

## 3. Splunk Interface: Splunk SDK for Python

### Decision
**splunk-sdk** (https://github.com/splunk/splunk-sdk-python) as the primary Splunk interface.

### Rationale
- **Official SDK**: Maintained by Splunk, ensures compatibility and timely updates
- **Complete Coverage**: Implements all major Splunk REST API endpoints
- **Authentication**: Built-in support for token and credential-based auth
- **Job Management**: High-level abstractions for search jobs, including polling and result retrieval
- **Error Handling**: Consistent error types and messages matching Splunk API
- **Connection Pooling**: SDK handles connection reuse efficiently

### Usage Pattern
```python
import splunklib.client as splunk_client

service = splunk_client.connect(
    host="splunk.example.com",
    port=8089,
    token="splunk-token"
)

# Execute search
job = service.jobs.create("search index=_internal | head 10")
job.wait()
results = job.results()
```

### Alternatives Considered
- **Direct REST API calls**: More control but requires implementing all endpoints manually
- **splunklib.client only**: We're using this, but could wrap it further (which we are doing)
- **splunk-sdk for Node.js**: Python SDK is more mature and feature-complete

---

## 4. Configuration Management: Pydantic

### Decision
**Pydantic v2** for configuration validation and management.

### Rationale
- **Type Safety**: Strong runtime type checking catches config errors early
- **Environment Variables**: First-class support for env var overrides
- **Nested Configs**: Complex config hierarchies with clear definitions
- **Validation**: Automatic validation with helpful error messages
- **Settings Management**: `pydantic-settings` provides dotenv and environment loading
- **JSON/YAML Support**: Easy serialization for config files

### Configuration Schema Example
```python
from pydantic import BaseSettings, Field, HttpUrl

class SplunkConfig(BaseSettings):
    host: str = Field(..., env="SPLUNK_HOST")
    port: int = Field(8089, env="SPLUNK_PORT")
    auth_type: str = Field("token", env="SPLUNK_AUTH_TYPE")
    token: str | None = Field(None, env="SPLUNK_TOKEN")
    username: str | None = Field(None, env="SPLUNK_USERNAME")
    password: str | None = Field(None, env="SPLUNK_PASSWORD")

    class Settings:
        env_nested_delimiter = "__"
```

### Alternatives Considered
- **Dynaconf**: More feature-rich but heavier, Pydantic sufficient
- **python-decouple**: Simpler but less powerful validation
- **configparser**: Basic, no type safety
- **dotted dict**: No validation

---

## 5. Type Checking: Pyright

### Decision
**Pyright** (via VS Code or command line) for static type checking.

### Rationale
- **Microsoft-backed**: Actively developed, widely used
- **Fast**: Incremental checking, good performance
- **Strict Mode**: Enables strict type checking for code quality
- **VS Code Integration**: Excellent editor support out of the box
- **Compatible**: Works well with Pydantic and modern Python typing

### Configuration
```json
{
  "venv": ".venv",
  "reportMissingImports": true,
  "reportMissingTypeStubs": false,
  "strict": true,
  "pythonVersion": "3.11"
}
```

### Alternatives Considered
- **mypy**: Industry standard but slower, some syntax differences
- **pytype**: Less strict, different approach
- **ruff with typing**: Limited type checking

---

## 6. Linting: Ruff

### Decision
**Ruff** for linting and formatting.

### Rationale
- **Speed**: Written in Rust, extremely fast
- **Comprehensive**: Implements flake8, isort, pydocstyle, and more
- **Auto-fixing**: Can fix many issues automatically
- **Modern**: Supports latest Python style guides
- **Formatting**: Includes Black-compatible formatting

### Configuration (pyproject.toml)
```toml
[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "W", "I", "UP", "N", "B", "A"]
ignore = []
```

### Alternatives Considered
- **flake8 + plugins**: Slower, requires multiple tools
- **pylint**: Too strict, verbose
- **Black + isort + flake8**: Three separate tools, Ruff combines them
- **autopep8**: Less configurable

---

## 7. Testing: pytest + pytest-cov

### Decision
**pytest** as the test framework with **pytest-cov** for coverage.

### Rationale
- **Popular**: Most widely used Python test framework
- **Fixtures**: Powerful fixture system for test setup/teardown
- **Parametrization**: Easy to write parameterized tests
- **Plugins**: Extensive plugin ecosystem (mocking, coverage, etc.)
- **Mocking**: Built-in `unittest.mock` sufficient, or `pytest-mock` for convenience
- **Coverage**: pytest-cov integrates seamlessly with coverage.py

### Test Structure
```
tests/
├── unit/
│   ├── test_auth.py
│   ├── test_config.py
│   └── test_client.py
├── integration/
│   ├── test_query.py
│   ├── test_info.py
│   └── ...
└── e2e/
    └── test_full_workflow.py
```

### Alternatives Considered
- **unittest**: Standard library but more verbose, less flexible
- **tox**: Good for multi-version testing but pytest sufficient
- **behave**: BDD style but adds complexity
- **hypothesis**: Property-based testing could be added later

---

## 8. Mocking Splunk: unittest.mock + pytest fixtures

### Decision
**unittest.mock** with custom fixtures for mocking Splunk API responses.

### Rationale
- **Standard Library**: Part of Python stdlib, no extra dependency
- **Flexible**: Can mock any level - SDK methods, HTTP requests, or entire services
- **Fixtures**: pytest fixtures allow reusable mock setups per test module
- **Testcontainers**: For integration tests, use **testcontainers-splunk** (if available) or manual Splunk container

### Mocking Strategy
- **Unit tests**: Mock `splunklib.client` methods directly
- **Integration tests**: Mock HTTP responses using `requests-mock` or similar
- **E2E tests**: Use actual Splunk Docker container (if feasible)

### Example Mock
```python
from unittest.mock import Mock, patch
import pytest

@pytest.fixture
def mock_splunk_service():
    with patch("splunkagent.client.splunklib.client.connect") as mock:
        service = Mock()
        mock.return_value = service
        yield service

def test_run_query(mock_splunk_service):
    # Setup mock job
    job = Mock()
    job.is_done.return_value = True
    job.results.return_value = [{"_time": "..."}]
    mock_splunk_service.jobs.create.return_value = job
    
    # Call function and assert
    ...
```

### Alternatives Considered
- **responses**: Good for HTTP mocking but Splunk SDK abstracts HTTP
- **requests-mock**: Lower-level, but we're using SDK
- **VCR.py**: Record/replay real API interactions - good for integration tests

---

## 9. Dependency Management: uv + pyproject.toml

### Decision
**uv** for fast dependency installation and **pyproject.toml** for project metadata (PEP 621).

### Rationale
- **Speed**: uv is dramatically faster than pip (Rust-based)
- **Modern**: Uses `pyproject.toml` standard (PEP 517/518/621)
- **Locking**: Built-in lock file generation for reproducible environments
- **Cross-platform**: Works on Linux, macOS, Windows
- **pip replacement**: Can replace pip, pip-tools, pipenv, and virtualenv

### pyproject.toml Structure
```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "splunkagent"
version = "0.1.0"
description = "Splunk MCP Agent"
requires-python = ">=3.11"
dependencies = [
    "mcp>=0.1.0",
    "splunk-sdk>=1.7.0",
    "pydantic>=2.0.0",
    "pydantic-settings>=2.0.0",
]

[project.scripts]
splunkagent = "splunkagent.server:main"

[tool.setuptools.packages.find]
where = ["src"]
```

### Alternatives Considered
- **pip + requirements.txt**: Traditional but less modern features
- **poetry**: Good but slower, lock file format not standard
- **pipenv**: Slow, abandoned by maintainers
- **hatch**: Good but uv is faster

---

## 10. CI/CD: GitHub Actions

### Decision
**GitHub Actions** for continuous integration.

### Rationale
- **Integrated**: Native to GitHub where repository lives
- **Free**: No cost for public repos and generous limits for private
- **Matrix Testing**: Easy to test across Python versions
- **YAML Config**: Human-readable and version-controlled
- **Artifacts**: Built-in support for test results and coverage
- **Secrets**: Secure secret storage for Splunk credentials in testing

### Workflow Strategy
- **test.yml**: Run on every push/PR to main/develop
  - Python 3.11 and 3.12 matrix
  - Install dependencies with uv
  - Run ruff (lint), pyright (typecheck), pytest (tests)
  - Upload coverage to Codecov or similar
- **deploy-staging.yml**: Deploy to staging environment on merge to develop
- **deploy-production.yml**: Deploy to production on tagged releases
- **security-scan**: Run trufflehog, bandit, safety checks

### Alternatives Considered
- **Jenkins**: Powerful but requires self-hosting
- **GitLab CI**: Good but redundant if using GitHub
- **CircleCI**: Good but costs money
- **Travis CI**: Dated, slower

---

## 11. Documentation: MkDocs + Material Theme

### Decision
**MkDocs** with **Material** theme for documentation site.

### Rationale
- **Simple**: Markdown-based, easy to write and maintain
- **Fast**: Instant preview with live-reload
- **Material Theme**: Professional, feature-rich, responsive
- **Search**: Built-in full-text search
- **Plugins**: Extensible (diagrams, code snippets, etc.)
- **GitHub Pages**: Easy deployment to GitHub Pages

### Structure
```
docs/
├── index.md           # Home page
├── getting-started.md
├── configuration.md
├── api/
│   ├── overview.md
│   ├── auth.md
│   ├── client.md
│   └── tools.md
├── troubleshooting.md
├── security.md
└── mkdocs.yml
```

### mkdocs.yml
```yaml
site_name: Splunk MCP Agent
theme:
  name: material
  palette:
    - scheme: default
      primary: indigo
      accent: indigo
plugins:
  - search
  - mkdocs-mermaid2
  - mkdocstrings
```

### Alternatives Considered
- **Sphinx**: More powerful but steeper learning curve, reStructuredText
- **Docusaurus**: React-based, JavaScript stack, heavier
- **Docsify**: Client-side rendering, SEO issues
- **Hugo**: Static site generator, Go-based, template complexity

---

## 12. Containerization: Docker (Optional)

### Decision
**Docker** with multi-stage build for optional containerized deployment.

### Rationale
- **Consistency**: Guarantees same environment across deployments
- **Isolation**: No dependency conflicts with host system
- **Distribution**: Easy to share and deploy via container registry
- **Multi-stage**: Minimize final image size (~100MB vs 500MB+)

### Dockerfile Structure
```dockerfile
# Build stage
FROM python:3.12-slim as builder
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv pip install -e . --system

# Runtime stage
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY src/ /app/src/
COPY pyproject.toml .
ENTRYPOINT ["splunkagent"]
```

### Alternatives Considered
- **No container**: Simpler but less portable
- **Docker Compose**: For local development with Splunk dependency (useful for e2e tests)

---

## 13. Logging: structlog + standard logging

### Decision
**structlog** on top of Python's standard `logging` module.

### Rationale
- **Structured Logging**: JSON output for machine parsing (important for distributed systems)
- **Flexible**: Works with standard logging, easy to configure
- **Performance**: C-accelerated, minimal overhead
- **Rich**: Context binding, processors, formatters

### Configuration Pattern
```python
import structlog

structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer()
    ],
    wrapper_class=structlog.stdlib.BoundLogger,
    logger_factory=structlog.stdlib.LoggerFactory(),
)

logger = structlog.get_logger()
logger.info("query_executed", query_hash=hash, duration=1.5)
```

### Alternatives Considered
- **logging only**: Built-in but no structured output out of the box
- **loguru**: Easier API but less standard, doesn't integrate as well with libraries
- **python-json-logger**: Only JSON formatting, less feature-rich

---

## 14. Error Handling: Custom Exception Hierarchy

### Decision
Define custom exception classes inheriting from a base `SplunkAgentError`.

### Rationale
- **Clarity**: Distinguish agent errors from library/network errors
- **Catchable**: Consumers can catch specific error types
- **Informative**: Attach context (Splunk response, operation, etc.)
- **Serializable**: Can be converted to MCP error responses cleanly

### Exception Hierarchy
```python
class SplunkAgentError(Exception):
    """Base exception for all agent errors"""
    pass

class AuthenticationError(SplunkAgentError):
    """Raised when Splunk authentication fails"""
    pass

class QueryError(SplunkAgentError):
    """Raised when SPL query execution fails"""
    pass

class ConfigurationError(SplunkAgentError):
    """Raised on invalid configuration"""
    pass

class SplunkAPIError(SplunkAgentError):
    """Raised when Splunk API returns an error"""
    def __init__(self, status, message, details):
        self.status = status
        self.details = details
```

---

## Summary Table

| **Aspect**               | **Choice**              | **Rationale**                                  |
|--------------------------|-------------------------|-----------------------------------------------|
| Language                 | Python 3.11+            | Splunk SDK availability, rapid development   |
| MCP Framework            | FastMCP                 | Python-native, simple, multiple transports   |
| Splunk Interface         | splunk-sdk (Python)     | Official, complete, mature                   |
| Config Management        | Pydantic v2             | Type safety, validation, env var support     |
| Type Checking            | Pyright                 | Fast, strict, excellent VS Code integration  |
| Linting                  | Ruff                    | Fast, comprehensive, auto-fixing             |
| Testing Framework        | pytest                  | Popular, flexible, extensive plugins         |
| Mocking                  | unittest.mock           | Stdlib, sufficient for our needs             |
| Dependency Management    | uv + pyproject.toml     | Fast, modern, PEP-compliant                  |
| CI/CD                    | GitHub Actions          | Integrated with GitHub, free, matrix testing |
| Documentation            | MkDocs + Material       | Markdown-based, fast, professional theme     |
| Containerization         | Docker (optional)       | Consistency, isolation, distribution         |
| Logging                  | structlog + logging     | Structured, flexible, performant             |
| Error Handling           | Custom exceptions       | Clear typing, informative, catchable         |

---

## Future Considerations

### Potential Additions
- **OpenTelemetry**: For distributed tracing if agent becomes distributed
- **Prometheus metrics**: For production monitoring
- **Redis caching**: For query result caching at scale
- **Async support**: Python asyncio for concurrent Splunk operations
- **gRPC transport**: Alternative MCP transport for performance
- **Secret management**: Integration with HashiCorp Vault or AWS Secrets Manager

### Scalability Path
- Current design is single-process, single-threaded (except Splunk SDK's internal concurrency)
- If scaling needed: Kubernetes deployment with autoscaling, load balancer in front of multiple agent instances
- State management would need Redis or similar for clustering

---

*Document Version: 1.0*
*Last Updated: 2026-04-05*