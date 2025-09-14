# TEST_api_client.py
import json
import urllib.request
import urllib.error
import urllib.parse
from typing import Dict, Any, Optional
import logging
from config import Config

logger = logging.getLogger(__name__)

class TESTApiClient:
    def __init__(self):
        self.base_url = Config.TEST_BASE_URL
        self.timeout = Config.TEST_TIMEOUT
    
    def call_test_maintenance(
        self,
        mmb: str,
        operate: str,
        acctype: str,
        env: str,
        channel: str,
        mode: str,
        ccy: str = None,
        cdtr_name: str = None
    ) -> Dict[str, Any]:
        """
        调用 TEST 维护 API
        """
        # 使用默认值
        ccy = ccy or Config.DEFAULT_CCY
        cdtr_name = cdtr_name or Config.DEFAULT_CDTR_NAME
        
        # 构建 URL
        url_path = f"{mmb}/{operate}/{acctype}/{env}/{channel}/{mode}"
        full_url = f"{self.base_url}/{url_path}"
        
        # 构建请求体
        payload = {
            "ccy": ccy,
            "cdtr": {
                "nm": cdtr_name
            }
        }
        
        try:
            logger.info(f"Calling TEST API: {full_url}")
            logger.debug(f"Payload: {json.dumps(payload)}")
            
            # 发送请求
            data = json.dumps(payload).encode('utf-8')
            headers = {
                'Content-Type': 'application/json',
                'Accept': 'application/json',
                'User-Agent': 'TEST-MCP-Client/1.0'
            }
            
            req = urllib.request.Request(full_url, data=data, headers=headers)
            
            with urllib.request.urlopen(req, timeout=self.timeout) as response:
                response_data = response.read().decode('utf-8')
                
                result = {
                    "success": True,
                    "status_code": response.status,
                    "url": full_url,
                    "request_payload": payload,
                    "response_data": self._parse_response(response_data),
                    "raw_response": response_data
                }
                
                logger.info(f"TEST API call successful: HTTP {response.status}")
                return result
                
        except urllib.error.HTTPError as e:
            error_body = ""
            try:
                error_body = e.read().decode('utf-8')
            except:
                pass
            
            result = {
                "success": False,
                "error_type": "HTTP_ERROR",
                "status_code": e.code,
                "error_message": f"HTTP {e.code}: {e.reason}",
                "error_body": error_body,
                "url": full_url,
                "request_payload": payload
            }
            
            logger.error(f"HTTP Error {e.code}: {e.reason}")
            return result
            
        except urllib.error.URLError as e:
            result = {
                "success": False,
                "error_type": "CONNECTION_ERROR", 
                "error_message": f"Connection failed: {str(e)}",
                "url": full_url,
                "request_payload": payload
            }
            
            logger.error(f"Connection Error: {e}")
            return result
            
        except Exception as e:
            result = {
                "success": False,
                "error_type": "UNEXPECTED_ERROR",
                "error_message": f"Unexpected error: {str(e)}",
                "url": full_url if 'full_url' in locals() else "Unknown",
                "request_payload": payload if 'payload' in locals() else {}
            }
            
            logger.error(f"Unexpected error: {e}")
            return result
    
    def _parse_response(self, response_text: str) -> Any:
        """解析响应数据"""
        try:
            return json.loads(response_text)
        except json.JSONDecodeError:
            return response_text
    
    def health_check(self) -> Dict[str, Any]:
        """健康检查"""
        try:
            # 尝试访问基础 URL
            req = urllib.request.Request(self.base_url, headers={'Accept': 'application/json'})
            with urllib.request.urlopen(req, timeout=10) as response:
                return {
                    "success": True,
                    "message": "TEST API is reachable",
                    "status_code": response.status,
                    "base_url": self.base_url
                }
        except Exception as e:
            return {
                "success": False,
                "message": f"TEST API unreachable: {str(e)}",
                "base_url": self.base_url
            }