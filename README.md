# TEST MCP Service

A simple MCP service that wraps TEST maintenance API for VS Code Copilot integration.

## What it does

This service allows you to call TEST maintenance operations directly from VS Code Copilot chat.

## Quick Setup

### 1. Create project folder
```bash
mkdir mcp-service
cd mcp-service
```

### 2. Add the Python files
- Copy all the `.py` files to this folder
- Main files: `config.py`, `test_api_client.py`, `test_mcp_server.py`

### 3. Test connection
```bash
python test_connection.py
```

### 4. Configure VS Code

Add this to your VS Code `settings.json`:

```json
{
  "github.copilot.chat.experimental.mcp": {
    "enabled": true,
    "servers": {
      "test-maintenance": {
        "command": "python",
        "args": ["/full/path/to/mcp-service/test_mcp_server.py"],
        "cwd": "/full/path/to/mcp-service"
      }
    }
  }
}
```

Replace `/full/path/to/mcp-service` with your actual folder path.

### 5. Restart VS Code

## How to use

Open VS Code Copilot Chat and try these commands:

### Check if TEST API is working
```
@test-maintenance test_health_check
```

### Quick test
```
@test-maintenance test_quick_test
```

### Full TEST maintenance call
```
@test-maintenance test_maintenance mmb="TEST" operate="CREATE" acctype="SAVINGS" env="DEV" channel="WEB" mode="NORMAL"
```

## Available commands

- **test_health_check** - Check if TEST API is reachable
- **test_quick_test** - Run a test with default parameters  
- **test_maintenance** - Full TEST operation with custom parameters

## Troubleshooting

### "TEST API unreachable"
- Check if `http://aaabbbcc:8000/test/` is accessible
- Make sure the FastAPI service is running

### "FastMCP not available" 
- This is normal, the service will work anyway
- It automatically uses a backup method

### VS Code doesn't see the service
- Make sure you used the full absolute path in settings
- Restart VS Code completely
- Check if Python is in your system PATH

## Configuration

To change the TEST API URL, edit `config.py`:

```python
TEST_BASE_URL = "http://your-server:8000/test"
```

That's it! The service should now work with VS Code Copilot.
