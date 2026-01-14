# MCP Inspector - Containerized Setup

## Running the MCP Inspector with Docker

This guide shows how to run the MCP Inspector using Docker with the container name `gohar-mcp`.

### Quick Start

Run the following command to start the MCP Inspector container:

```bash
docker run --rm \
  --name gohar-mcp \
  -p 127.0.0.1:6274:6274 \
  -p 127.0.0.1:6277:6277 \
  -e HOST=0.0.0.0 \
  -e MCP_AUTO_OPEN_ENABLED=false \
  ghcr.io/modelcontextprotocol/inspector:latest
```

### Command Breakdown

- `--rm`: Automatically remove the container when it stops
- `--name gohar-mcp`: Names the container "gohar-mcp"
- `-p 127.0.0.1:6274:6274`: Maps port 6274 for the inspector UI
- `-p 127.0.0.1:6277:6277`: Maps port 6277 for the MCP server
- `-e HOST=0.0.0.0`: Sets the host to listen on all interfaces inside the container
- `-e MCP_AUTO_OPEN_ENABLED=false`: Disables automatic browser opening

### Accessing the Inspector

Once the container is running, access the MCP Inspector at:

```
http://127.0.0.1:6274
```

### Stopping the Container

To stop the container, press `Ctrl+C` in the terminal, or run:

```bash
docker stop gohar-mcp
```

### Running in Detached Mode

To run the container in the background:

```bash
docker run -d \
  --name gohar-mcp \
  -p 127.0.0.1:6274:6274 \
  -p 127.0.0.1:6277:6277 \
  -e HOST=0.0.0.0 \
  -e MCP_AUTO_OPEN_ENABLED=false \
  ghcr.io/modelcontextprotocol/inspector:latest
```

View logs with:

```bash
docker logs -f gohar-mcp
```

## Troubleshooting

### Port Already in Use

If you get an error that ports 6274 or 6277 are already in use, you need to kill the process using those ports.

#### Find and Kill Process on Port 6274

```bash
# Find the process ID (PID) using port 6274
lsof -i :6274

# Kill the process (replace PID with the actual process ID)
kill -9 PID
```

#### Find and Kill Process on Port 6277

```bash
# Find the process ID (PID) using port 6277
lsof -i :6277

# Kill the process (replace PID with the actual process ID)
kill -9 PID
```

#### Kill Both Ports at Once

```bash
# Find and kill all processes on both ports
lsof -ti :6274 | xargs kill -9
lsof -ti :6277 | xargs kill -9
```

After killing the processes, try running the Docker container again.
