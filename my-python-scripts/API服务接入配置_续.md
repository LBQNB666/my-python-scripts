import hashlib

class DingTalkBot:
    """钉钉群机器人"""
    
    def __init__(self, webhook_url, secret=None):
        self.webhook_url = webhook_url
        self.secret = secret
    
    def _sign(self):
        """生成签名"""
        if not self.secret:
            return None
        
        timestamp = str(int(time.time() * 1000))
        secret_enc = self.secret.encode('utf-8')
        string_to_sign = f'{timestamp}\n{self.secret}'
        string_to_sign_enc = string_to_sign.encode('utf-8')
        hmac_code = hmac.new(secret_enc, string_to_sign_enc, hashlib.sha256).digest()
        sign = base64.b64encode(hmac_code).decode('utf-8')
        
        return f'{self.webhook_url}&timestamp={timestamp}&sign={sign}'
    
    def send_text(self, content):
        """发送文本消息"""
        url = self._sign() if self.secret else self.webhook_url
        
        data = {
            'msgtype': 'text',
            'text': {'content': f'{content}'}
        }
        
        response = requests.post(url, json=data)
        return response.json()
    
    def send_markdown(self, title, text):
        """发送 Markdown 消息"""
        url = self._sign() if self.secret else self.webhook_url
        
        data = {
            'msgtype': 'markdown',
            'markdown': {
                'title': title,
                'text': text
            }
        }
        
        response = requests.post(url, json=data)
        return response.json()
    
    def send_link(self, title, text, message_url, pic_url=None):
        """发送链接消息"""
        url = self._sign() if self.secret else self.webhook_url
        
        data = {
            'msgtype': 'link',
            'link': {
                'title': title,
                'text': text,
                'messageUrl': message_url,
                'picUrl': pic_url or ''
            }
        }
        
        response = requests.post(url, json=data)
        return response.json()
    
    def send_action_card(self, title, text, single_title, single_url):
        """发送 ActionCard 消息"""
        url = self._sign() if self.secret else self.webhook_url
        
        data = {
            'msgtype': 'actionCard',
            'actionCard': {
                'title': title,
                'text': text,
                'singleTitle': single_title,
                'singleURL': single_url
            }
        }
        
        response = requests.post(url, json=data)
        return response.json()

# 使用
bot = DingTalkBot('你的Webhook地址', '你的Secret')

# 发送文本
bot.send_text('Hello World!')

# 发送 Markdown
bot.send_markdown(
    '日报摘要',
    '### 今日完成\n\n- 任务1 ✓\n- 任务2 ✓\n\n### 明日计划\n\n- 任务3\n- 任务4'
)

# 发送链接
bot.send_link(
    '点击查看详情',
    '这是一条链接消息',
    'https://example.com'
)
```

### 5. 企业微信群机器人

```python
import requests
import json

class WeComBot:
    """企业微信群机器人"""
    
    def __init__(self, webhook_url):
        self.webhook_url = webhook_url
    
    def send_text(self, content):
        """发送文本消息"""
        data = {
            'msgtype': 'text',
            'text': {'content': content}
        }
        
        response = requests.post(self.webhook_url, json=data)
        return response.json()
    
    def send_markdown(self, content):
        """发送 Markdown 消息"""
        data = {
            'msgtype': 'markdown',
            'markdown': {'content': content}
        }
        
        response = requests.post(self.webhook_url, json=data)
        return response.json()
    
    def send_news(self, articles):
        """发送图文消息"""
        data = {
            'msgtype': 'news',
            'news': {
                'articles': articles  # [{'title': '标题', 'description': '描述', 'url': '链接', 'picurl': '图片URL'}]
            }
        }
        
        response = requests.post(self.webhook_url, json=data)
        return response.json()

# 使用
bot = WeComBot('你的Webhook地址')

bot.send_text('Hello!')
bot.send_markdown('### 标题\n> 引用文本')
```

### 6. 钉钉工作通知

```python
import requests
import json

class DingTalkWork:
    """钉钉工作通知（需要 Access Token）"""
    
    def __init__(self, app_key, app_secret):
        self.app_key = app_key
        self.app_secret = app_secret
        self.access_token = None
    
    def _get_token(self):
        """获取 access_token"""
        url = 'https://api.dingtalk.com/v1.0/oauth2/accessToken'
        data = {'appKey': self.app_key, 'appSecret': self.app_secret}
        
        response = requests.post(url, json=data)
        result = response.json()
        
        if 'accessToken' in result:
            self.access_token = result['accessToken']
            return self.access_token
        else:
            raise Exception(f'获取Token失败: {result}')
    
    def send_to_user(self, user_ids, msg_content):
        """发送给指定用户"""
        if not self.access_token:
            self._get_token()
        
        url = 'https://api.dingtalk.com/v1.0/robot/robots/instance/sendToUser'
        headers = {'x-acs-dingtalk-access-token': self.access_token}
        
        data = {
            'robotCode': self.app_key,
            'userIds': user_ids,
            'msg': msg_content
        }
        
        response = requests.post(url, json=data, headers=headers)
        return response.json()
```

---

## 📊 Open API 通用调用模板

```python
import requests
import json
import time
from typing import Dict, Any, Optional

class APIClient:
    """通用 API 客户端"""
    
    def __init__(self, base_url: str, headers: Dict = None):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        self.session.headers.update(headers or {})
    
    def request(self, method: str, endpoint: str, 
                params: Dict = None, data: Any = None,
                json_data: Dict = None,
                retry: int = 3) -> Dict:
        """发送请求
        
        Args:
            method: GET, POST, PUT, DELETE
            endpoint: API 端点
            params: URL 参数
            data: 表单数据
            json_data: JSON 数据
            retry: 重试次数
        """
        url = f'{self.base_url}/{endpoint.lstrip("/")}'
        
        for attempt in range(retry):
            try:
                response = self.session.request(
                    method=method,
                    url=url,
                    params=params,
                    data=data,
                    json=json_data,
                    timeout=30
                )
                
                response.raise_for_status()
                return response.json()
            
            except requests.exceptions.RequestException as e:
                if attempt < retry - 1:
                    time.sleep(1 * (attempt + 1))
                    continue
                raise Exception(f'API请求失败: {e}')
        
        return None
    
    def get(self, endpoint: str, params: Dict = None):
        return self.request('GET', endpoint, params=params)
    
    def post(self, endpoint: str, json_data: Dict = None):
        return self.request('POST', endpoint, json_data=json_data)
    
    def put(self, endpoint: str, json_data: Dict = None):
        return self.request('PUT', endpoint, json_data=json_data)
    
    def delete(self, endpoint: str, params: Dict = None):
        return self.request('DELETE', endpoint, params=params)

# 使用示例
class MyAPI(APIClient):
    """自定义 API"""
    
    def __init__(self, api_key: str):
        super().__init__(
            base_url='https://api.example.com',
            headers={'Authorization': f'Bearer {api_key}'}
        )
    
    def get_user_info(self, user_id: str):
        return self.get(f'/users/{user_id}')
    
    def create_order(self, order_data: Dict):
        return self.post('/orders', json_data=order_data)
    
    def update_product(self, product_id: str, data: Dict):
        return self.put(f'/products/{product_id}', json_data=data)

# 使用
api = MyAPI('your_api_key')
user = api.get_user_info('12345')
orders = api.create_order({'product_id': 'P001', 'quantity': 10})
```

---

## 🔐 API 密钥管理

```python
import os

# 方式1：环境变量（推荐）
API_KEY = os.getenv('MY_API_KEY')

# 方式2：配置文件
# 创建 config.json（不要提交到 Git）
# {
#     "api_keys": {
#         "sms": "your_sms_key",
#         "email": "your_email_key"
