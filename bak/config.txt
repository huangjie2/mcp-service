# config.py
import os

class Config:
    # TEST API 配置
    TEST_BASE_URL = os.getenv("TEST_BASE_URL", "http://aaabbbcc:8000/test")
    TEST_TIMEOUT = int(os.getenv("TEST_TIMEOUT", "30"))
    
    # MCP 服务配置
    MCP_SERVER_NAME = "test-maintenance-mcp"
    MCP_VERSION = "1.0.0"
    
    # 日志配置
    LOG_LEVEL = os.getenv("LOG_LEVEL", "INFO")
    
    # 默认值
    DEFAULT_CCY = "HKD"
    DEFAULT_CDTR_NAME = "test name"