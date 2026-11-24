# Calling the server with curl

The server speaks JSON-RPC 2.0 over HTTP POST to `https://www.norli.se/mcp`. It is stateless:
no session header is needed, and each request stands on its own.

```sh
MCP=https://www.norli.se/mcp
H=(-H 'Content-Type: application/json' -H 'Accept: application/json, text/event-stream')
```

## Initialize

```sh
curl -s "$MCP" "${H[@]}" -d '{
  "jsonrpc": "2.0", "id": 1, "method": "initialize",
  "params": {"protocolVersion": "2025-06-18", "capabilities": {},
             "clientInfo": {"name": "curl", "version": "1"}}
}'
```

## List the tools

```sh
curl -s "$MCP" "${H[@]}" -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'
```

## Rental income estimate

```sh
curl -s "$MCP" "${H[@]}" -d '{
  "jsonrpc": "2.0", "id": 3, "method": "tools/call",
  "params": {"name": "hyresuppskattning",
             "arguments": {"bostadstyp": "lagenhet", "sovrum": 2, "postnummer": "118 20",
                           "tillval": ["balkong_uteplats"], "sprak": "en"}}
}'
```

## Search the guides

```sh
curl -s "$MCP" "${H[@]}" -d '{
  "jsonrpc": "2.0", "id": 4, "method": "tools/call",
  "params": {"name": "sok_guider", "arguments": {"fraga": "skatt på uthyrning", "max": 3}}
}'
```

## Areas

```sh
curl -s "$MCP" "${H[@]}" -d '{
  "jsonrpc": "2.0", "id": 5, "method": "tools/call",
  "params": {"name": "omraden", "arguments": {}}
}'
```
