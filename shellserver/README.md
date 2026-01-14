# Shell Server - MCP Project

A Model Context Protocol (MCP) server project built with Python.

## Project Setup Steps

This document outlines the steps taken to create this project from scratch.

### 1. Initialize the Project with uv

Created a new Python project using `uv`:

```bash
uv init shellserver
cd shellserver
```

This command created the initial project structure with:
- `pyproject.toml` - Project configuration and dependencies
- `server.py` - Main Python script
- `.python-version` - Python version specification (3.13)

### 2. Install MCP Dependencies

Added the MCP library with CLI support:

```bash
uv add "mcp[cli]"
```

This installed:
- `mcp[cli]>=1.25.0` and its dependencies
- Created `.venv/` virtual environment
- Generated `uv.lock` for dependency locking

## Project Structure

```
shellserver/
├── .python-version    # Python 3.13
├── .venv/            # Virtual environment
├── pyproject.toml    # Project configuration
├── server.py         # Main server script
├── uv.lock          # Dependency lock file
└── README.md        # This file
```

## Development

### Prerequisites

- Python 3.13 or higher
- [uv](https://github.com/astral-sh/uv) package manager

### Running the Server

```bash
# Using uv
uv run server.py

# Or activate the virtual environment
source .venv/bin/activate
python server.py
```

### Installing Additional Dependencies

```bash
uv add <package-name>
```

### Syncing Dependencies

```bash
uv sync
```

## Next Steps

- Implement MCP server functionality in `server.py`
- Define tools and resources for the MCP server
- Add server configuration and endpoints
- Test with MCP Inspector

## Resources

- [Model Context Protocol Documentation](https://modelcontextprotocol.io/)
- [uv Documentation](https://github.com/astral-sh/uv)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
