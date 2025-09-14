# test_mcp_server.py
import json
import logging
import sys
from typing import Dict, Any, List
from test_api_client import TESTApiClient
from config import Config

# 配置日志
logging.basicConfig(
    level=getattr(logging, Config.LOG_LEVEL),
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

# 尝试导入 FastMCP，如果失败则使用 stdio 实现
try:
    from fastmcp import FastMCP
    USE_FASTMCP = True
    logger.info("Using FastMCP implementation")
except ImportError:
    USE_FASTMCP = False
    logger.info("FastMCP not available, using stdio implementation")

# TEST API 客户端
test_client = TESTApiClient()

if USE_FASTMCP:
    # FastMCP 实现
    app = FastMCP(Config.MCP_SERVER_NAME)
    
    @app.tool()
    def test_maintenance(
        mmb: str,
        operate: str,
        acctype: str,
        env: str,
        channel: str,
        mode: str,
        ccy: str = Config.DEFAULT_CCY,
        cdtr_name: str = Config.DEFAULT_CDTR_NAME
    ) -> str:
        """
        Execute TEST maintenance operations
        
        Args:
            mmb: MMB parameter
            operate: Operation type (CREATE, UPDATE, DELETE, etc.)
            acctype: Account type (SAVINGS, CURRENT, etc.)
            env: Environment (DEV, TEST, PROD, etc.)
            channel: Channel (WEB, API, MOBILE, etc.)
            mode: Mode (NORMAL, BATCH, etc.)
            ccy: Currency code (default: HKD)
            cdtr_name: Creditor name (default: test name)
        
        Returns:
            JSON response from TEST API
        """
        result = test_client.call_test_maintenance(
            mmb, operate, acctype, env, channel, mode, ccy, cdtr_name
        )
        return json.dumps(result, indent=2)
    
    @app.tool()
    def test_health_check() -> str:
        """
        Check TEST API health status
        
        Returns:
            Health check result
        """
        result = test_client.health_check()
        return json.dumps(result, indent=2)
    
    @app.tool()
    def test_quick_test(test_env: str = "DEV") -> str:
        """
        Quick test with predefined parameters
        
        Args:
            test_env: Environment for testing (DEV, TEST)
        
        Returns:
            Test result
        """
        result = test_client.call_test_maintenance(
            mmb="TEST",
            operate="CREATE",
            acctype="SAVINGS",
            env=test_env,
            channel="WEB",
            mode="NORMAL"
        )
        return json.dumps(result, indent=2)
    
    def run_fastmcp():
        logger.info(f"Starting {Config.MCP_SERVER_NAME} with FastMCP")
        app.run()

else:
    # Stdio 实现（无依赖版本）
    class StdioMCPServer:
        def __init__(self):
            self.tools = {
                "test_maintenance": {
                    "description": "Execute TEST maintenance operations",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "mmb": {"type": "string", "description": "MMB parameter"},
                            "operate": {"type": "string", "description": "Operation type"},
                            "acctype": {"type": "string", "description": "Account type"},
                            "env": {"type": "string", "description": "Environment"},
                            "channel": {"type": "string", "description": "Channel"},
                            "mode": {"type": "string", "description": "Mode"},
                            "ccy": {"type": "string", "description": "Currency", "default": Config.DEFAULT_CCY},
                            "cdtr_name": {"type": "string", "description": "Creditor name", "default": Config.DEFAULT_CDTR_NAME}
                        },
                        "required": ["mmb", "operate", "acctype", "env", "channel", "mode"]
                    }
                },
                "test_health_check": {
                    "description": "Check TEST API health status",
                    "parameters": {"type": "object", "properties": {}}
                },
                "test_quick_test": {
                    "description": "Quick test with predefined parameters",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "test_env": {"type": "string", "description": "Environment", "default": "DEV"}
                        }
                    }
                }
            }
        
        def handle_request(self, request: Dict[str, Any]) -> Dict[str, Any]:
            method = request.get("method")
            
            if method == "tools/list":
                return {
                    "tools": [
                        {
                            "name": name,
                            "description": tool["description"],
                            "inputSchema": tool["parameters"]
                        }
                        for name, tool in self.tools.items()
                    ]
                }
            
            elif method == "tools/call":
                params = request.get("params", {})
                tool_name = params.get("name")
                arguments = params.get("arguments", {})
                
                if tool_name == "test_maintenance":
                    result = test_client.call_test_maintenance(**arguments)
                elif tool_name == "test_health_check":
                    result = test_client.health_check()
                elif tool_name == "test_quick_test":
                    test_env = arguments.get("test_env", "DEV")
                    result = test_client.call_test_maintenance(
                        mmb="TEST", operate="CREATE", acctype="SAVINGS",
                        env=test_env, channel="WEB", mode="NORMAL"
                    )
                else:
                    return {"error": f"Unknown tool: {tool_name}"}
                
                return {
                    "content": [
                        {
                            "type": "text",
                            "text": json.dumps(result, indent=2)
                        }
                    ]
                }
            
            return {"error": f"Unknown method: {method}"}
        
        def run(self):
            logger.info(f"Starting {Config.MCP_SERVER_NAME} with stdio")
            for line in sys.stdin:
                try:
                    request = json.loads(line.strip())
                    response = self.handle_request(request)
                    print(json.dumps(response))
                    sys.stdout.flush()
                except Exception as e:
                    error_response = {"error": str(e)}
                    print(json.dumps(error_response))
                    sys.stdout.flush()
    
    def run_stdio():
        server = StdioMCPServer()
        server.run()

# 主入口
if __name__ == "__main__":
    if USE_FASTMCP:
        run_fastmcp()
    else:
        run_stdio()