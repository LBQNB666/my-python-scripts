# 🔌 API 服务接入配置指南

> 调用各种 REST API 服务

---

## 📦 安装依赖

```bash
pip install requests pandas openpyxl
```

---

## 🚀 常用 API 接入示例

### 1. 发送短信（阿里云/腾讯云）

```python
import requests
import hashlib
import time
import json

class SMSClient:
    """短信发送客户端"""
    
    def __init__(self, api_key, api_secret, sign_name):
        self.api_key = api_key
        self.api_secret = api_secret
        self.sign_name = sign_name
    
    def send(self, phone, template_code, template_params=None):
        """发送短信"""
        # 这里以阿里云为例
        url = 'https://dysmsapi.aliyuncs.com/'
        
        # 构建请求参数
        params = {
            'PhoneNumbers': phone,
            'SignName': self.sign_name,
            'TemplateCode': template_code,
            'TemplateParam': json.dumps(template_params) if template_params else None
        }
        
        # 添加签名（实际使用时需要按阿里云文档生成）
        # ...
        
        response = requests.post(url, json=params)
        return response.json()

# 使用
# sms = SMSClient('你的Key', '你的Secret', '签名')
# sms.send('13800138000', 'SMS_xxx', {'code': '123456'})
```

### 2. 发送邮件

```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.header import Header

class EmailClient:
    """邮件发送客户端"""
    
    def __init__(self, smtp_host, smtp_port, username, password):
        self.smtp_host = smtp_host
        self.smtp_port = smtp_port
        self.username = username
        self.password = password
    
    def send(self, to_emails, subject, body, body_type='html'):
        """发送邮件"""
        msg = MIMEMultipart()
        msg['From'] = Header(self.username)
        msg['To'] = ','.join(to_emails) if isinstance(to_emails, list) else to_emails
        msg['Subject'] = Header(subject, 'utf-8')
        
        msg.attach(MIMEText(body, body_type, 'utf-8'))
        
        with smtplib.SMTP_SSL(self.smtp_host, self.smtp_port) as server:
            server.login(self.username, self.password)
            server.sendmail(self.username, to_emails, msg.as_string())
        
        print(f'邮件已发送至: {to_emails}')
    
    def send_with_attachment(self, to_emails, subject, body, attachments):
        """发送带附件的邮件"""
        msg = MIMEMultipart()
        msg['From'] = Header(self.username)
        msg['To'] = ','.join(to_emails) if isinstance(to_emails, list) else to_emails
        msg['Subject'] = Header(subject, 'utf-8')
        
        msg.attach(MIMEText(body, 'html', 'utf-8'))
        
        # 添加附件
        for file_path in attachments:
            with open(file_path, 'rb') as f:
                att = MIMEText(f.read(), 'base64', 'utf-8')
                att['Content-Type'] = 'application/octet-stream'
                att['Content-Disposition'] = f'attachment; filename="{file_path}"'
                msg.attach(att)
        
        with smtplib.SMTP_SSL(self.smtp_host, self.smtp_port) as server:
            server.login(self.username, self.password)
            server.sendmail(self.username, to_emails, msg.as_string())
        
        print(f'邮件已发送至: {to_emails}')

# 使用
email = EmailClient('smtp.qq.com', 465, 'your@qq.com', 'your授权码')
email.send(
    ['recipient@example.com'],
    '这是主题',
    '<h1>HTML邮件内容</h1><p>Hello World!</p>'
)

# 发送带附件
email.send_with_attachment(
    ['recipient@example.com'],
    '报表',
    '请查收附件',
    ['report.xlsx', 'data.pdf']
)
```

### 3. 天气 API

```python
import requests
import json

class WeatherAPI:
    """天气查询"""
    
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = 'https://restapi.amap.com/v3/weather'
    
    def get_weather(self, city, city_type='ip'):
        """获取天气信息
        
        Args:
            city: 城市名或IP地址
            city_type: 'ip' 表示通过IP查询，'cityname' 表示通过城市名查询
        """
        url = f'{self.base_url}/weatherInfo'
        params = {
            'key': self.api_key,
            'city': city,
            'extensions': 'base'  # base: basic info, all: full info
        }
        
        response = requests.get(url, params=params)
        data = response.json()
        
        if data['status'] == '1':
            weather = data['lives'][0]
            return {
                '城市': weather['province'] + weather['city'],
                '天气': weather['weather'],
                '温度': weather['temperature'] + '°C',
                '风向': weather['winddirection'],
                '风力': weather['windpower'],
                '湿度': weather['humidity']
            }
        else:
            return {'错误': data['info']}

# 使用（需要申请高德API Key）
# weather = WeatherAPI('你的API Key')
# result = weather.get_weather('北京市')
# print(result)
```

### 4. 钉钉群机器人

```python
import requests
import json
import time
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
        hmac_code = hashlib\
