# start_server.py
#!/usr/bin/env python3
import sys
import os
import subprocess
import logging

def main():
    print("Starting TEST MCP Service...")
    
    # 设置环境变量（如果需要）
    os.environ.setdefault("TEST_BASE_URL", "http://aaabbbcc:8000/test")
    os.environ.setdefault("LOG_LEVEL", "INFO")
    
    script_path = os.path.join(os.path.dirname(__file__), "test_mcp_server.py")
    
    try:
        subprocess.run([sys.executable, script_path], check=True)
    except KeyboardInterrupt:
        print("\nMCP service stopped")
    except Exception as e:
        print(f"Error: {e}")
        sys.exit(1)

if __name__ == "__main__":
    main()