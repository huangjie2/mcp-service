# test_connection.py
import json
from test_api_client import TESTApiClient
from config import Config

def test_all():
    print("=" * 50)
    print("TEST MCP Service Connection Test")
    print("=" * 50)
    
    client = TESTApiClient()
    
    # 1. 健康检查
    print("1. Testing TEST API health...")
    health_result = client.health_check()
    print(f"   Result: {'✓' if health_result['success'] else '✗'}")
    if not health_result['success']:
        print(f"   Error: {health_result['message']}")
    
    # 2. 测试完整调用
    print("\n2. Testing TEST maintenance call...")
    test_result = client.call_test_maintenance(
        mmb="TEST",
        operate="CREATE", 
        acctype="SAVINGS",
        env="DEV",
        channel="WEB",
        mode="NORMAL"
    )
    print(f"   Result: {'✓' if test_result['success'] else '✗'}")
    if test_result['success']:
        print(f"   Status: {test_result['status_code']}")
    else:
        print(f"   Error: {test_result['error_message']}")
    
    # 3. 显示配置
    print(f"\n3. Configuration:")
    print(f"   Base URL: {Config.TEST_BASE_URL}")
    print(f"   Timeout: {Config.TEST_TIMEOUT}s")
    
    print("=" * 50)

if __name__ == "__main__":
    test_all()