# urban-waddle3

一个功能完善的 Web 爬虫框架

## 目录

- [项目概述](#项目概述)
- [快速开始](#快速开始)
- [核心架构](#核心架构)
- [配置管理](#配置管理)
- [爬虫引擎](#爬虫引擎)
- [网站适配](#网站适配)
- [数据存储](#数据存储)
- [代理管理](#代理管理)
- [会话管理](#会话管理)
- [性能优化](#性能优化)
- [错误处理](#错误处理)
- [监控告警](#监控告警)
- [安全防护](#安全防护)
- [测试验证](#测试验证)
- [部署运维](#部署运维)
- [API接口](#api接口)
- [扩展开发](#扩展开发)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [版本历史](#版本历史)
- [附录](#附录)

---

## 项目概述

### 简介

urban-waddle3 是一个功能强大、易于扩展的 Web 爬虫框架，专为高效数据采集而设计。该框架提供了完整的爬虫解决方案，包括请求管理、数据解析、存储、代理池、会话管理等核心功能。

### 主要特性

- **高性能**: 支持异步并发爬取，充分利用系统资源
- **易扩展**: 模块化设计，支持自定义插件和中间件
- **智能防护**: 内置反爬虫机制，包括代理轮换、请求频率控制等
- **数据管道**: 灵活的数据处理和存储管道
- **监控告警**: 实时监控爬虫运行状态，异常及时告警
- **分布式**: 支持分布式部署，可横向扩展

### 技术栈

- Python 3.8+
- aiohttp / requests
- BeautifulSoup4 / lxml
- Redis / MongoDB
- Docker

### 适用场景

- 电商数据采集
- 新闻资讯爬取
- 社交媒体数据分析
- 价格监控
- SEO 数据采集

---

## 快速开始

### 环境要求

- Python 3.8 或更高版本
- pip 或 conda 包管理工具
- Redis (可选，用于分布式任务队列)
- MongoDB (可选，用于数据存储)

### 安装

#### 使用 pip 安装

```bash
pip install urban-waddle3
```

#### 从源码安装

```bash
git clone https://github.com/xfdb88/urban-waddle3.git
cd urban-waddle3
pip install -e .
```

### 第一个爬虫

创建一个简单的爬虫示例：

```python
from urban_waddle3 import Spider, Request

class MySpider(Spider):
    name = 'my_spider'
    start_urls = ['https://example.com']
    
    def parse(self, response):
        # 提取数据
        title = response.css('h1::text').get()
        
        # 返回数据
        yield {
            'title': title,
            'url': response.url
        }
        
        # 跟踪链接
        for link in response.css('a::attr(href)').getall():
            yield Request(url=link, callback=self.parse)

# 运行爬虫
if __name__ == '__main__':
    spider = MySpider()
    spider.run()
```

### 运行爬虫

```bash
# 命令行方式
python my_spider.py

# 或使用框架命令
waddle run my_spider
```

---

## 核心架构

### 架构概览

urban-waddle3 采用模块化、分层的架构设计：

```
┌─────────────────────────────────────────┐
│           用户接口层 (CLI/API)           │
├─────────────────────────────────────────┤
│            爬虫引擎 (Engine)             │
├──────────┬──────────┬──────────┬────────┤
│ 调度器   │ 下载器   │ 解析器   │ 管道  │
│(Scheduler)│(Downloader)│(Parser)│(Pipeline)│
├──────────┴──────────┴──────────┴────────┤
│              中间件层                    │
├─────────────────────────────────────────┤
│         核心组件 (请求/响应/任务)        │
└─────────────────────────────────────────┘
```

### 核心组件

#### 1. 引擎 (Engine)

- 协调各个组件的工作
- 控制数据流
- 管理爬虫生命周期

#### 2. 调度器 (Scheduler)

- 管理待爬取的 URL 队列
- 去重处理
- 优先级调度

#### 3. 下载器 (Downloader)

- 执行 HTTP 请求
- 管理连接池
- 处理重试逻辑

#### 4. 解析器 (Parser)

- 解析 HTML/JSON 响应
- 提取数据
- 生成新的请求

#### 5. 数据管道 (Pipeline)

- 数据清洗
- 数据验证
- 数据存储

### 工作流程

1. Engine 从 Scheduler 获取待爬取的请求
2. Downloader 执行请求，获取响应
3. Parser 解析响应，提取数据和新的 URL
4. 提取的数据经过 Pipeline 处理后存储
5. 新的 URL 提交给 Scheduler
6. 重复以上步骤，直到队列为空

---

## 配置管理

### 配置文件

项目支持多种配置方式：

#### 1. Python 配置文件

创建 `settings.py`：

```python
# 爬虫设置
SPIDER_SETTINGS = {
    'concurrent_requests': 16,
    'download_delay': 1,
    'retry_times': 3,
    'timeout': 30,
}

# 下载器设置
DOWNLOADER_SETTINGS = {
    'user_agent': 'Mozilla/5.0...',
    'cookies_enabled': True,
    'headers': {
        'Accept-Language': 'zh-CN,zh;q=0.9',
    }
}

# 数据存储设置
STORAGE_SETTINGS = {
    'type': 'mongodb',
    'host': 'localhost',
    'port': 27017,
    'database': 'spider_data',
}

# 代理设置
PROXY_SETTINGS = {
    'enabled': True,
    'pool_size': 100,
    'verify_ssl': False,
}
```

#### 2. YAML 配置文件

创建 `config.yaml`：

```yaml
spider:
  name: my_spider
  concurrent_requests: 16
  download_delay: 1
  retry_times: 3
  
downloader:
  timeout: 30
  user_agent: "Mozilla/5.0..."
  
storage:
  type: mongodb
  host: localhost
  port: 27017
  database: spider_data
  
proxy:
  enabled: true
  pool_size: 100
```

#### 3. 环境变量

```bash
export WADDLE_CONCURRENT_REQUESTS=16
export WADDLE_DOWNLOAD_DELAY=1
export WADDLE_STORAGE_TYPE=mongodb
```

### 配置优先级

环境变量 > 命令行参数 > 配置文件 > 默认值

### 常用配置项

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| concurrent_requests | 并发请求数 | 16 |
| download_delay | 下载延迟(秒) | 0 |
| retry_times | 重试次数 | 3 |
| timeout | 请求超时(秒) | 30 |
| log_level | 日志级别 | INFO |

---

## 爬虫引擎

### 引擎简介

爬虫引擎是框架的核心，负责协调各个组件的工作，控制整个爬取流程。

### 引擎特性

- **异步架构**: 基于 asyncio，支持高并发
- **智能调度**: 自动管理请求队列和优先级
- **中间件支持**: 可自定义请求/响应处理逻辑
- **生命周期管理**: 完整的启动、暂停、停止机制

### 使用引擎

```python
from urban_waddle3 import Engine, Spider

# 创建爬虫实例
spider = MySpider()

# 创建引擎
engine = Engine(spider)

# 配置引擎
engine.configure({
    'concurrent_requests': 32,
    'download_delay': 0.5,
})

# 启动引擎
engine.start()

# 等待完成
engine.wait_for_completion()

# 停止引擎
engine.stop()
```

### 引擎钩子

```python
class MySpider(Spider):
    def spider_opened(self):
        """爬虫启动时调用"""
        self.logger.info('Spider started')
    
    def spider_closed(self, reason):
        """爬虫关闭时调用"""
        self.logger.info(f'Spider closed: {reason}')
    
    def spider_idle(self):
        """爬虫空闲时调用"""
        self.logger.info('Spider is idle')
```

### 信号系统

```python
from urban_waddle3.signals import spider_opened, request_sent

@spider_opened.connect
def on_spider_opened(spider):
    print(f'Spider {spider.name} opened')

@request_sent.connect
def on_request_sent(request):
    print(f'Request sent: {request.url}')
```

---

## 网站适配

### 适配器模式

为不同网站创建专门的适配器：

```python
from urban_waddle3 import SiteAdapter

class EcommerceSiteAdapter(SiteAdapter):
    """电商网站适配器"""
    
    def parse_product(self, response):
        """解析商品信息"""
        return {
            'title': response.css('.product-title::text').get(),
            'price': response.css('.price::text').get(),
            'image': response.css('.product-img::attr(src)').get(),
        }
    
    def parse_list(self, response):
        """解析列表页"""
        for item in response.css('.product-item'):
            yield self.parse_product(item)
```

### 常见网站类型

#### 1. 电商网站

```python
class TaobaoAdapter(SiteAdapter):
    name = 'taobao'
    allowed_domains = ['taobao.com']
    
    def parse_product(self, response):
        return {
            'id': response.css('.item-id::text').get(),
            'title': response.css('.item-title::text').get(),
            'price': response.css('.price::text').re(r'[\d.]+')[0],
            'sales': response.css('.sales::text').get(),
        }
```

#### 2. 新闻网站

```python
class NewsAdapter(SiteAdapter):
    name = 'news'
    
    def parse_article(self, response):
        return {
            'title': response.css('h1::text').get(),
            'author': response.css('.author::text').get(),
            'content': response.css('.article-content::text').getall(),
            'publish_time': response.css('.publish-time::text').get(),
        }
```

#### 3. API 接口

```python
class APIAdapter(SiteAdapter):
    name = 'api'
    
    def parse_json(self, response):
        data = response.json()
        return {
            'items': data.get('data', []),
            'total': data.get('total', 0),
        }
```

### 动态网站处理

```python
from urban_waddle3 import SeleniumMiddleware

class DynamicSiteAdapter(SiteAdapter):
    custom_settings = {
        'downloader_middlewares': {
            SeleniumMiddleware: 543,
        }
    }
    
    def parse(self, response):
        # 等待 JavaScript 加载完成
        response.wait_for_selector('.content-loaded')
        return response.css('.data::text').getall()
```

---

## 数据存储

### 存储后端

框架支持多种存储后端：

#### 1. MongoDB

```python
STORAGE_SETTINGS = {
    'type': 'mongodb',
    'host': 'localhost',
    'port': 27017,
    'database': 'spider_db',
    'collection': 'items',
}

# 使用
from urban_waddle3.storage import MongoDBStorage

storage = MongoDBStorage(**STORAGE_SETTINGS)
storage.save(item)
```

#### 2. MySQL

```python
STORAGE_SETTINGS = {
    'type': 'mysql',
    'host': 'localhost',
    'port': 3306,
    'user': 'root',
    'password': 'password',
    'database': 'spider_db',
}
```

#### 3. Redis

```python
STORAGE_SETTINGS = {
    'type': 'redis',
    'host': 'localhost',
    'port': 6379,
    'db': 0,
}
```

#### 4. JSON/CSV 文件

```python
STORAGE_SETTINGS = {
    'type': 'file',
    'format': 'json',  # or 'csv'
    'path': './data/output.json',
}
```

### 数据管道

```python
from urban_waddle3 import Pipeline

class CleanPipeline(Pipeline):
    """数据清洗管道"""
    
    def process_item(self, item, spider):
        # 清洗数据
        item['title'] = item['title'].strip()
        item['price'] = float(item['price'])
        return item

class ValidationPipeline(Pipeline):
    """数据验证管道"""
    
    def process_item(self, item, spider):
        # 验证必填字段
        required_fields = ['title', 'price']
        for field in required_fields:
            if field not in item:
                raise ValueError(f'Missing field: {field}')
        return item

class StoragePipeline(Pipeline):
    """数据存储管道"""
    
    def open_spider(self, spider):
        self.storage = MongoDBStorage()
    
    def process_item(self, item, spider):
        self.storage.save(item)
        return item
    
    def close_spider(self, spider):
        self.storage.close()
```

### 数据去重

```python
from urban_waddle3 import DuplicateFilter

class ItemDuplicateFilter(DuplicateFilter):
    """数据去重过滤器"""
    
    def __init__(self):
        self.seen = set()
    
    def is_duplicate(self, item):
        item_id = item.get('id')
        if item_id in self.seen:
            return True
        self.seen.add(item_id)
        return False
```

---

## 代理管理

### 代理池

```python
from urban_waddle3 import ProxyPool

# 创建代理池
proxy_pool = ProxyPool()

# 添加代理
proxy_pool.add('http://proxy1.com:8080')
proxy_pool.add('http://proxy2.com:8080')

# 从文件加载
proxy_pool.load_from_file('proxies.txt')

# 从 API 加载
proxy_pool.load_from_api('https://api.proxy.com/list')
```

### 代理中间件

```python
class ProxyMiddleware:
    """代理中间件"""
    
    def __init__(self, proxy_pool):
        self.proxy_pool = proxy_pool
    
    def process_request(self, request, spider):
        # 为请求设置代理
        proxy = self.proxy_pool.get_proxy()
        request.meta['proxy'] = proxy
        return request
    
    def process_response(self, request, response, spider):
        # 检查代理是否有效
        if response.status in [403, 407]:
            # 标记代理失效
            proxy = request.meta.get('proxy')
            self.proxy_pool.mark_failed(proxy)
        return response
```

### 代理验证

```python
class ProxyValidator:
    """代理验证器"""
    
    async def validate(self, proxy):
        """验证代理是否可用"""
        try:
            response = await self.request(
                'https://httpbin.org/ip',
                proxy=proxy,
                timeout=10
            )
            return response.status == 200
        except Exception:
            return False
```

### 智能代理切换

```python
PROXY_SETTINGS = {
    'enabled': True,
    'pool_size': 100,
    'rotation_strategy': 'round_robin',  # or 'random', 'priority'
    'failure_threshold': 3,
    'retry_interval': 300,
}
```

---

## 会话管理

### Cookie 管理

```python
from urban_waddle3 import CookieJar

# 创建 Cookie 管理器
cookie_jar = CookieJar()

# 设置 Cookie
cookie_jar.set('session_id', 'abc123', domain='.example.com')

# 获取 Cookie
cookies = cookie_jar.get_cookies('https://example.com')

# 加载 Cookie
cookie_jar.load_from_file('cookies.txt')

# 保存 Cookie
cookie_jar.save_to_file('cookies.txt')
```

### 会话保持

```python
class LoginSpider(Spider):
    """需要登录的爬虫"""
    
    def start_requests(self):
        # 先进行登录
        yield Request(
            url='https://example.com/login',
            callback=self.login,
            dont_filter=True
        )
    
    def login(self, response):
        """登录逻辑"""
        return FormRequest.from_response(
            response,
            formdata={
                'username': 'user',
                'password': 'pass'
            },
            callback=self.after_login
        )
    
    def after_login(self, response):
        """登录后开始爬取"""
        if '登录成功' in response.text:
            for url in self.start_urls:
                yield Request(url=url, callback=self.parse)
```

### Session 池

```python
class SessionPool:
    """Session 池管理"""
    
    def __init__(self, size=10):
        self.sessions = []
        self.size = size
        self._init_sessions()
    
    def _init_sessions(self):
        """初始化 session 池"""
        for _ in range(self.size):
            session = self._create_session()
            self.sessions.append(session)
    
    def get_session(self):
        """获取可用 session"""
        return self.sessions.pop(0)
    
    def return_session(self, session):
        """归还 session"""
        self.sessions.append(session)
```

---

## 性能优化

### 并发控制

```python
CONCURRENT_SETTINGS = {
    'concurrent_requests': 32,  # 并发请求数
    'concurrent_requests_per_domain': 8,  # 每个域名并发数
    'concurrent_requests_per_ip': 0,  # 每个 IP 并发数
}
```

### 下载延迟

```python
DOWNLOAD_SETTINGS = {
    'download_delay': 0.5,  # 固定延迟
    'randomize_download_delay': True,  # 随机延迟
    'autothrottle_enabled': True,  # 自动限速
    'autothrottle_start_delay': 0.5,
    'autothrottle_max_delay': 3.0,
}
```

### 缓存机制

```python
CACHE_SETTINGS = {
    'cache_enabled': True,
    'cache_expire_time': 3600,  # 缓存过期时间
    'cache_backend': 'redis',
    'cache_key_prefix': 'waddle:cache:',
}

# 使用缓存
class CachedSpider(Spider):
    custom_settings = {
        'cache_enabled': True,
    }
    
    def parse(self, response):
        # 响应会被自动缓存
        return response.css('.data::text').getall()
```

### 资源限制

```python
RESOURCE_SETTINGS = {
    'memory_limit': '1GB',  # 内存限制
    'cpu_limit': '2',  # CPU 核心数限制
    'max_request_queue_size': 10000,  # 请求队列大小
}
```

### 性能监控

```python
from urban_waddle3 import PerformanceMonitor

monitor = PerformanceMonitor()

# 记录性能指标
monitor.record('requests_per_second', 100)
monitor.record('response_time', 0.5)

# 获取统计信息
stats = monitor.get_stats()
print(f'平均响应时间: {stats.avg_response_time}')
print(f'请求成功率: {stats.success_rate}')
```

---

## 错误处理

### 异常类型

```python
from urban_waddle3.exceptions import (
    SpiderException,
    DownloadException,
    ParseException,
    StorageException,
)
```

### 错误处理策略

```python
ERROR_SETTINGS = {
    'retry_enabled': True,
    'retry_times': 3,
    'retry_http_codes': [500, 502, 503, 504, 408, 429],
    'retry_priority_adjust': -1,
}
```

### 自定义错误处理

```python
class ErrorHandlerMiddleware:
    """错误处理中间件"""
    
    def process_exception(self, request, exception, spider):
        """处理异常"""
        if isinstance(exception, TimeoutError):
            spider.logger.warning(f'Request timeout: {request.url}')
            # 重试请求
            return request.copy()
        
        elif isinstance(exception, ConnectionError):
            spider.logger.error(f'Connection error: {request.url}')
            # 切换代理重试
            new_request = request.copy()
            new_request.meta['switch_proxy'] = True
            return new_request
        
        return None
```

### 降级策略

```python
class FallbackSpider(Spider):
    """支持降级的爬虫"""
    
    def parse(self, response):
        try:
            # 尝试主要解析逻辑
            return self.parse_primary(response)
        except Exception as e:
            self.logger.warning(f'Primary parse failed: {e}')
            # 降级到备用解析逻辑
            return self.parse_fallback(response)
    
    def parse_primary(self, response):
        """主要解析逻辑"""
        return response.css('.primary-selector').get()
    
    def parse_fallback(self, response):
        """备用解析逻辑"""
        return response.css('.fallback-selector').get()
```

### 错误日志

```python
import logging

# 配置日志
LOG_SETTINGS = {
    'log_level': 'INFO',
    'log_file': 'spider.log',
    'log_format': '%(asctime)s [%(name)s] %(levelname)s: %(message)s',
}

# 记录错误
class MySpider(Spider):
    def parse(self, response):
        try:
            # 爬取逻辑
            pass
        except Exception as e:
            self.logger.error(f'Parse error: {e}', exc_info=True)
```

---

## 监控告警

### 监控指标

```python
from urban_waddle3 import Monitor

monitor = Monitor()

# 记录指标
monitor.record_metric('requests_total', 1000)
monitor.record_metric('requests_success', 950)
monitor.record_metric('requests_failed', 50)
monitor.record_metric('response_time_avg', 0.5)
```

### 实时监控

```python
class MonitorMiddleware:
    """监控中间件"""
    
    def __init__(self):
        self.monitor = Monitor()
    
    def process_request(self, request, spider):
        request.meta['start_time'] = time.time()
        self.monitor.increment('requests_total')
        return request
    
    def process_response(self, request, response, spider):
        duration = time.time() - request.meta['start_time']
        self.monitor.record('response_time', duration)
        self.monitor.increment('requests_success')
        return response
    
    def process_exception(self, request, exception, spider):
        self.monitor.increment('requests_failed')
        return None
```

### 告警配置

```python
ALERT_SETTINGS = {
    'enabled': True,
    'alert_channels': ['email', 'slack', 'webhook'],
    'alert_rules': [
        {
            'metric': 'error_rate',
            'operator': '>',
            'threshold': 0.1,  # 错误率超过 10%
            'message': 'Error rate too high',
        },
        {
            'metric': 'requests_per_second',
            'operator': '<',
            'threshold': 10,
            'message': 'Requests per second too low',
        },
    ],
}
```

### 告警通知

```python
class AlertNotifier:
    """告警通知器"""
    
    def send_email(self, subject, content):
        """发送邮件告警"""
        pass
    
    def send_slack(self, message):
        """发送 Slack 告警"""
        pass
    
    def send_webhook(self, data):
        """发送 Webhook 告警"""
        pass
```

### 监控面板

访问监控面板：`http://localhost:8080/monitor`

主要功能：
- 实时请求统计
- 错误率监控
- 响应时间分布
- 爬虫状态概览
- 历史数据查询

---

## 安全防护

### 反爬虫策略

#### 1. User-Agent 轮换

```python
USER_AGENTS = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36',
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36',
]

class RandomUserAgentMiddleware:
    def process_request(self, request, spider):
        request.headers['User-Agent'] = random.choice(USER_AGENTS)
        return request
```

#### 2. 请求头伪装

```python
DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8',
    'Accept-Encoding': 'gzip, deflate, br',
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1',
}
```

#### 3. 频率限制

```python
RATE_LIMIT_SETTINGS = {
    'enabled': True,
    'max_requests_per_second': 10,
    'max_requests_per_minute': 300,
}
```

### 验证码处理

```python
from urban_waddle3 import CaptchaSolver

class CaptchaMiddleware:
    def __init__(self):
        self.solver = CaptchaSolver()
    
    def process_response(self, request, response, spider):
        if self.has_captcha(response):
            # 识别验证码
            captcha_code = self.solver.solve(response.captcha_image)
            # 重新提交请求
            return request.copy(formdata={'captcha': captcha_code})
        return response
```

### IP 封禁应对

```python
class IPRotationMiddleware:
    """IP 轮换中间件"""
    
    def process_response(self, request, response, spider):
        # 检测 IP 是否被封
        if self.is_blocked(response):
            spider.logger.warning('IP blocked, rotating...')
            # 切换代理
            new_request = request.copy()
            new_request.meta['force_new_proxy'] = True
            return new_request
        return response
    
    def is_blocked(self, response):
        """检测是否被封"""
        blocked_keywords = ['访问受限', '请求过于频繁', 'Access Denied']
        return any(kw in response.text for kw in blocked_keywords)
```

### 数据加密

```python
from urban_waddle3 import Encryptor

encryptor = Encryptor(key='your-secret-key')

# 加密数据
encrypted = encryptor.encrypt(data)

# 解密数据
decrypted = encryptor.decrypt(encrypted)
```

---

## 测试验证

### 单元测试

```python
import unittest
from urban_waddle3 import Spider, Request

class TestSpider(unittest.TestCase):
    def setUp(self):
        self.spider = Spider()
    
    def test_parse_title(self):
        """测试标题解析"""
        response = self.create_response('<h1>Test Title</h1>')
        result = self.spider.parse(response)
        self.assertEqual(result['title'], 'Test Title')
    
    def test_generate_requests(self):
        """测试请求生成"""
        requests = list(self.spider.start_requests())
        self.assertGreater(len(requests), 0)

if __name__ == '__main__':
    unittest.main()
```

### 集成测试

```python
class IntegrationTest(unittest.TestCase):
    def test_full_spider_run(self):
        """测试完整爬虫流程"""
        spider = MySpider()
        engine = Engine(spider)
        
        # 运行爬虫
        engine.start()
        engine.wait_for_completion()
        
        # 验证结果
        self.assertGreater(engine.stats.item_count, 0)
        self.assertEqual(engine.stats.error_count, 0)
```

### Mock 测试

```python
from unittest.mock import Mock, patch

class TestWithMock(unittest.TestCase):
    @patch('urban_waddle3.Downloader')
    def test_download(self, mock_downloader):
        """使用 Mock 测试下载"""
        mock_response = Mock()
        mock_response.status = 200
        mock_response.text = '<html>Test</html>'
        mock_downloader.return_value.download.return_value = mock_response
        
        spider = MySpider()
        result = spider.parse(mock_response)
        self.assertIsNotNone(result)
```

### 性能测试

```python
import time

class PerformanceTest(unittest.TestCase):
    def test_spider_speed(self):
        """测试爬虫速度"""
        spider = MySpider()
        start_time = time.time()
        
        # 运行爬虫
        spider.run()
        
        duration = time.time() - start_time
        requests_per_second = spider.stats.request_count / duration
        
        # 断言性能指标
        self.assertGreater(requests_per_second, 10)
```

### 测试覆盖率

```bash
# 安装覆盖率工具
pip install coverage

# 运行测试并生成覆盖率报告
coverage run -m unittest discover
coverage report
coverage html
```

---

## 部署运维

### Docker 部署

#### Dockerfile

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制代码
COPY . .

# 运行爬虫
CMD ["python", "main.py"]
```

#### docker-compose.yml

```yaml
version: '3.8'

services:
  spider:
    build: .
    environment:
      - WADDLE_CONCURRENT_REQUESTS=16
      - WADDLE_STORAGE_TYPE=mongodb
    depends_on:
      - mongodb
      - redis
    volumes:
      - ./data:/app/data
  
  mongodb:
    image: mongo:5.0
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
  
  redis:
    image: redis:6.2
    ports:
      - "6379:6379"

volumes:
  mongodb_data:
```

### Kubernetes 部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spider-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spider
  template:
    metadata:
      labels:
        app: spider
    spec:
      containers:
      - name: spider
        image: urban-waddle3:latest
        env:
        - name: WADDLE_CONCURRENT_REQUESTS
          value: "16"
        resources:
          limits:
            memory: "1Gi"
            cpu: "1"
```

### 监控部署

```bash
# 部署 Prometheus
kubectl apply -f prometheus.yaml

# 部署 Grafana
kubectl apply -f grafana.yaml
```

### 日志管理

```python
# 使用 ELK 栈收集日志
LOG_SETTINGS = {
    'handlers': {
        'logstash': {
            'class': 'logstash.TCPLogstashHandler',
            'host': 'logstash.example.com',
            'port': 5000,
        }
    }
}
```

### 自动化部署

```yaml
# .github/workflows/deploy.yml
name: Deploy Spider

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Build Docker image
      run: docker build -t spider:${{ github.sha }} .
    
    - name: Push to registry
      run: docker push spider:${{ github.sha }}
    
    - name: Deploy to k8s
      run: kubectl set image deployment/spider spider=spider:${{ github.sha }}
```

---

## API接口

### RESTful API

```python
from urban_waddle3 import APIServer

# 创建 API 服务器
api = APIServer()

@api.route('/spider/start', methods=['POST'])
def start_spider():
    """启动爬虫"""
    spider_name = request.json.get('spider_name')
    spider = get_spider(spider_name)
    spider.start()
    return {'status': 'started', 'spider': spider_name}

@api.route('/spider/stop', methods=['POST'])
def stop_spider():
    """停止爬虫"""
    spider_name = request.json.get('spider_name')
    spider = get_spider(spider_name)
    spider.stop()
    return {'status': 'stopped', 'spider': spider_name}

@api.route('/spider/status/<spider_name>', methods=['GET'])
def get_spider_status(spider_name):
    """获取爬虫状态"""
    spider = get_spider(spider_name)
    return {
        'status': spider.status,
        'stats': spider.get_stats(),
    }

# 启动 API 服务器
api.run(host='0.0.0.0', port=8080)
```

### WebSocket API

```python
from urban_waddle3 import WebSocketServer

ws = WebSocketServer()

@ws.on('connect')
def on_connect(client):
    """客户端连接"""
    print(f'Client connected: {client.id}')

@ws.on('spider.start')
def on_spider_start(client, data):
    """启动爬虫"""
    spider_name = data['spider_name']
    spider = get_spider(spider_name)
    spider.start()
    
    # 推送实时状态
    @spider.on_stats_update
    def on_stats(stats):
        client.emit('spider.stats', stats)

ws.run(port=8081)
```

### GraphQL API

```python
from urban_waddle3 import GraphQLServer

schema = '''
type Query {
    spiders: [Spider]
    spider(name: String!): Spider
}

type Mutation {
    startSpider(name: String!): Spider
    stopSpider(name: String!): Spider
}

type Spider {
    name: String!
    status: String!
    stats: Stats
}

type Stats {
    requestCount: Int
    itemCount: Int
    errorCount: Int
}
'''

graphql = GraphQLServer(schema)
graphql.run(port=8082)
```

### API 文档

访问 API 文档：`http://localhost:8080/docs`

---

## 扩展开发

### 自定义中间件

```python
from urban_waddle3 import Middleware

class CustomMiddleware(Middleware):
    """自定义中间件"""
    
    def __init__(self, settings):
        self.settings = settings
    
    @classmethod
    def from_crawler(cls, crawler):
        """从 crawler 创建中间件实例"""
        return cls(crawler.settings)
    
    def process_request(self, request, spider):
        """处理请求"""
        # 修改请求
        request.headers['Custom-Header'] = 'value'
        return request
    
    def process_response(self, request, response, spider):
        """处理响应"""
        # 修改响应
        return response
    
    def process_exception(self, request, exception, spider):
        """处理异常"""
        # 处理异常
        return None
```

### 自定义管道

```python
from urban_waddle3 import Pipeline

class CustomPipeline(Pipeline):
    """自定义管道"""
    
    def open_spider(self, spider):
        """爬虫开启时调用"""
        self.file = open('output.json', 'w')
    
    def close_spider(self, spider):
        """爬虫关闭时调用"""
        self.file.close()
    
    def process_item(self, item, spider):
        """处理数据项"""
        # 处理数据
        self.file.write(json.dumps(item) + '\n')
        return item
```

### 自定义扩展

```python
from urban_waddle3 import Extension

class CustomExtension(Extension):
    """自定义扩展"""
    
    def __init__(self, crawler):
        self.crawler = crawler
        self.stats = crawler.stats
    
    @classmethod
    def from_crawler(cls, crawler):
        ext = cls(crawler)
        crawler.signals.connect(ext.spider_opened, signal=signals.spider_opened)
        crawler.signals.connect(ext.spider_closed, signal=signals.spider_closed)
        return ext
    
    def spider_opened(self, spider):
        """爬虫开启时调用"""
        spider.logger.info('Custom extension activated')
    
    def spider_closed(self, spider, reason):
        """爬虫关闭时调用"""
        spider.logger.info(f'Custom extension deactivated: {reason}')
```

### 插件系统

```python
from urban_waddle3 import Plugin

class CustomPlugin(Plugin):
    """自定义插件"""
    
    name = 'custom_plugin'
    version = '1.0.0'
    
    def install(self, app):
        """安装插件"""
        app.register_middleware(CustomMiddleware)
        app.register_pipeline(CustomPipeline)
    
    def uninstall(self, app):
        """卸载插件"""
        app.unregister_middleware(CustomMiddleware)
        app.unregister_pipeline(CustomPipeline)
```

---

## 最佳实践

### 1. 代码组织

```
project/
├── spiders/          # 爬虫模块
│   ├── __init__.py
│   ├── ecommerce.py
│   └── news.py
├── middlewares/      # 中间件
│   ├── __init__.py
│   └── custom.py
├── pipelines/        # 数据管道
│   ├── __init__.py
│   └── storage.py
├── settings.py       # 配置文件
├── main.py          # 入口文件
└── requirements.txt # 依赖列表
```

### 2. 错误处理

- 使用 try-except 捕获异常
- 记录详细的错误日志
- 实现重试机制
- 设置合理的超时时间

### 3. 性能优化

- 合理设置并发数
- 使用缓存减少重复请求
- 启用压缩传输
- 优化数据解析逻辑

### 4. 反爬虫策略

- 随机化请求间隔
- 轮换 User-Agent
- 使用代理池
- 模拟真实用户行为

### 5. 数据质量

- 验证数据完整性
- 去重处理
- 数据清洗
- 异常数据标记

### 6. 监控告警

- 设置关键指标监控
- 配置告警规则
- 定期检查日志
- 性能基准测试

### 7. 安全防护

- 不要在代码中硬编码敏感信息
- 使用环境变量管理配置
- 定期更新依赖库
- 遵守网站 robots.txt 规则

### 8. 可维护性

- 编写清晰的注释
- 遵循代码规范
- 编写单元测试
- 文档及时更新

---

## 常见问题

### Q1: 如何解决请求超时问题？

**A:** 可以通过以下方式解决：

```python
DOWNLOAD_SETTINGS = {
    'timeout': 60,  # 增加超时时间
    'retry_times': 5,  # 增加重试次数
}
```

### Q2: 如何处理动态加载的内容？

**A:** 使用 Selenium 或 Playwright 中间件：

```python
from urban_waddle3 import SeleniumMiddleware

class DynamicSpider(Spider):
    custom_settings = {
        'downloader_middlewares': {
            SeleniumMiddleware: 543,
        }
    }
```

### Q3: 如何避免 IP 被封？

**A:** 使用代理池和请求频率控制：

```python
PROXY_SETTINGS = {
    'enabled': True,
    'rotation_strategy': 'random',
}

RATE_LIMIT_SETTINGS = {
    'enabled': True,
    'max_requests_per_second': 5,
}
```

### Q4: 如何处理验证码？

**A:** 集成验证码识别服务：

```python
from urban_waddle3 import CaptchaSolver

solver = CaptchaSolver(api_key='your-api-key')
captcha_code = solver.solve(captcha_image)
```

### Q5: 如何实现分布式爬取？

**A:** 使用 Redis 作为共享队列：

```python
SCHEDULER_SETTINGS = {
    'scheduler': 'urban_waddle3.schedulers.RedisScheduler',
    'redis_url': 'redis://localhost:6379',
}
```

### Q6: 内存占用过高怎么办？

**A:** 优化内存使用：

```python
RESOURCE_SETTINGS = {
    'memory_limit': '500MB',
    'max_request_queue_size': 1000,
    'item_pipeline_batch_size': 100,
}
```

### Q7: 如何调试爬虫？

**A:** 使用调试模式：

```python
# 启用调试日志
LOG_LEVEL = 'DEBUG'

# 使用 IPython shell
from IPython import embed
embed()  # 在代码中插入断点
```

### Q8: 如何导出数据到 Excel？

**A:** 使用 Excel 导出管道：

```python
from urban_waddle3.pipelines import ExcelPipeline

ITEM_PIPELINES = {
    ExcelPipeline: 300,
}

EXCEL_SETTINGS = {
    'file_path': 'output.xlsx',
    'sheet_name': 'data',
}
```

---

## 版本历史

### v1.0.0 (2025-01-15)

**新增功能：**
- 基础爬虫框架
- 支持 HTTP/HTTPS 请求
- 基本的数据解析功能
- MongoDB 存储支持

### v1.1.0 (2025-02-20)

**新增功能：**
- 代理池管理
- 请求重试机制
- Cookie 管理
- Redis 缓存支持

**改进：**
- 优化并发性能
- 改进错误处理

**修复：**
- 修复内存泄漏问题
- 修复编码错误

### v1.2.0 (2025-03-30)

**新增功能：**
- Selenium 中间件
- 验证码识别
- 分布式爬取支持
- RESTful API

**改进：**
- 提升爬取速度 30%
- 优化内存使用

### v2.0.0 (2025-05-15)

**重大更新：**
- 全新架构设计
- 异步并发支持
- 插件系统
- GraphQL API
- 监控告警系统

**新增功能：**
- WebSocket 实时推送
- Docker 支持
- Kubernetes 部署
- 自动化测试

**改进：**
- 性能提升 200%
- 更好的错误处理
- 完善的文档

**不兼容变更：**
- API 接口调整
- 配置文件格式变更

---

## 附录

### A. 术语表

- **爬虫 (Spider)**: 自动化抓取网页数据的程序
- **引擎 (Engine)**: 控制爬虫工作流程的核心组件
- **调度器 (Scheduler)**: 管理 URL 队列的组件
- **下载器 (Downloader)**: 执行 HTTP 请求的组件
- **管道 (Pipeline)**: 处理和存储数据的组件
- **中间件 (Middleware)**: 在请求/响应过程中插入自定义逻辑的组件
- **选择器 (Selector)**: 用于从 HTML 中提取数据的工具
- **代理 (Proxy)**: 用于隐藏真实 IP 的中间服务器

### B. 参考资源

#### 官方文档
- 官方网站: https://urban-waddle3.readthedocs.io
- GitHub: https://github.com/xfdb88/urban-waddle3
- API 文档: https://urban-waddle3.readthedocs.io/api

#### 社区资源
- 讨论论坛: https://discuss.urban-waddle3.io
- Stack Overflow: 标签 `urban-waddle3`
- 中文社区: https://community.urban-waddle3.cn

#### 相关工具
- BeautifulSoup4: HTML 解析库
- lxml: 高性能 XML/HTML 解析器
- Selenium: 浏览器自动化工具
- Redis: 内存数据库
- MongoDB: NoSQL 数据库

### C. 代码示例

完整的代码示例可以在以下位置找到：

- GitHub 示例仓库: https://github.com/xfdb88/urban-waddle3-examples
- 官方教程: https://urban-waddle3.readthedocs.io/tutorials

### D. 贡献指南

欢迎贡献代码！请参考：

1. Fork 项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

详细贡献指南：https://github.com/xfdb88/urban-waddle3/CONTRIBUTING.md

### E. 许可证

本项目采用 MIT 许可证。详见 [LICENSE](LICENSE) 文件。

### F. 联系方式

- 邮箱: support@urban-waddle3.io
- 微信群: 扫描二维码加入
- QQ 群: 123456789
- Twitter: @urban_waddle3

### G. 致谢

感谢所有为本项目做出贡献的开发者和社区成员！

特别感谢：
- Scrapy 项目提供的设计灵感
- Python 社区的支持
- 所有贡献者和用户的反馈

---

**最后更新**: 2025-10-22
**文档版本**: v2.0.0
