# TrendRadar 开发者技术文档

> 版本：v5.5.0 | 更新日期：2025-02-01

## 目录

- [一、快速开始](#一快速开始)
- [二、项目架构](#二项目架构)
- [三、核心模块详解](#三核心模块详解)
- [四、数据模型](#四数据模型)
- [五、配置系统](#五配置系统)
- [六、开发指南](#六开发指南)
- [七、测试与调试](#七测试与调试)
- [八、常见开发任务](#八常见开发任务)
- [九、API 参考](#九api-参考)

---

## 一、快速开始

### 1.1 环境要求

```bash
# Python 版本
Python 3.10+

# 可选依赖
- Docker (用于容器化部署)
- Git (用于版本控制)
```

### 1.2 安装步骤

```bash
# 1. 克隆项目
git clone https://github.com/sansan0/TrendRadar.git
cd TrendRadar

# 2. 创建虚拟环境（推荐）
python -m venv venv

# Windows 激活虚拟环境
venv\Scripts\activate

# Linux/Mac 激活虚拟环境
source venv/bin/activate

# 3. 安装依赖
pip install -e .

# 或使用 requirements.txt
pip install -r requirements.txt

# 4. 验证安装
python -m trendradar --help
```

### 1.3 配置文件

复制并编辑配置文件：

```bash
# 配置文件位置
config/config.yaml              # 主配置
config/frequency_words.txt      # 关键词配置
config/ai_analysis_prompt.txt   # AI 分析提示词
config/ai_translation_prompt.txt # AI 翻译提示词
```

### 1.4 首次运行

```bash
# 基础运行（仅爬取，不推送）
python -m trendradar

# 查看帮助
python -m trendradar --help

# 测试爬虫（不保存）
python -m trendradar --test-crawl

# 生成报告（不推送）
python -m trendradar --no-notification
```

---

## 二、项目架构

### 2.1 目录结构

```
TrendRadar/
├── config/                         # 配置文件目录
│   ├── config.yaml                # 主配置文件
│   ├── frequency_words.txt        # 关键词配置
│   ├── ai_analysis_prompt.txt    # AI 分析提示词
│   └── ai_translation_prompt.txt # AI 翻译提示词
│
├── trendradar/                      # 核心代码包
│   ├── __main__.py               # 程序入口
│   ├── __init__.py
│   ├── context.py                # 应用上下文（统一接口）
│   │
│   ├── core/                     # 核心业务逻辑
│   │   ├── __init__.py
│   │   ├── config.py             # 配置加载和多账号解析
│   │   ├── loader.py             # 数据加载器
│   │   ├── frequency.py          # 关键词匹配逻辑
│   │   ├── analyzer.py           # 统计分析
│   │   └── data.py              # 数据模型
│   │
│   ├── crawler/                  # 数据抓取模块
│   │   ├── __init__.py
│   │   ├── fetcher.py            # 热榜抓取器
│   │   └── rss/                 # RSS 抓取
│   │       ├── __init__.py
│   │       ├── fetcher.py        # RSS 抓取器
│   │       └── parser.py         # RSS 解析器
│   │
│   ├── storage/                  # 存储管理模块
│   │   ├── __init__.py
│   │   ├── base.py               # 存储后端基类
│   │   ├── manager.py            # 存储管理器
│   │   ├── local.py              # 本地 SQLite 存储
│   │   ├── remote.py             # 远程 S3 存储
│   │   └── sqlite_mixin.py       # SQLite 混入
│   │
│   ├── report/                   # 报告生成模块
│   │   ├── __init__.py
│   │   ├── generator.py          # 报告生成器
│   │   ├── helpers.py            # 辅助函数
│   │   ├── html.py               # HTML 报告
│   │   ├── rss_html.py           # RSS HTML
│   │   └── formatter.py          # 格式化器
│   │
│   ├── notification/              # 通知推送模块
│   │   ├── __init__.py
│   │   ├── dispatcher.py         # 通知调度器
│   │   ├── senders.py            # 各平台发送器
│   │   ├── renderer.py           # 渲染器
│   │   ├── push_manager.py       # 推送记录管理器
│   │   ├── splitter.py           # 内容分批发送
│   │   ├── batch.py              # 批次处理
│   │   └── formatters.py         # 格式化器
│   │
│   ├── ai/                       # AI 模块
│   │   ├── __init__.py
│   │   ├── client.py             # LiteLLM 客户端封装
│   │   ├── analyzer.py           # AI 分析器
│   │   ├── translator.py         # AI 翻译器
│   │   └── formatter.py          # AI 结果格式化
│   │
│   └── utils/                    # 工具模块
│       ├── __init__.py
│       ├── time.py               # 时间处理
│       └── url.py                # URL 标准化
│
├── mcp_server/                     # MCP AI 分析服务器
│   ├── __init__.py
│   └── server.py
│
├── docker/                         # Docker 部署配置
│   ├── docker-compose.yml
│   ├── Dockerfile
│   ├── Dockerfile.mcp
│   ├── .env
│   └── manage.py
│
├── output/                         # 输出数据目录
│   ├── news/                     # 热榜数据（SQLite）
│   ├── rss/                      # RSS 数据（SQLite）
│   ├── html/                     # HTML 报告
│   ├── txt/                      # TXT 快照
│   └── index.html                # 当日汇总
│
├── .github/                        # GitHub Actions 配置
│   └── workflows/
│       ├── crawler.yml            # 定时爬虫
│       └── check-in.yml           # 签到续期
│
├── pyproject.toml                # Python 项目配置
├── requirements.txt              # 依赖列表
├── README.md                     # 项目文档
└── LICENSE                       # GPL-3.0 许可证
```

### 2.2 架构设计模式

#### 分层架构

```
┌─────────────────────────────────────────────────────┐
│                     表现层                         │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────┐│
│  │   报告生成    │  │   通知推送    │  │  HTML   ││
│  │   (report)   │  │(notification) │  │ Output  ││
│  └──────────────┘  └──────────────┘  └─────────┘│
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                     业务层                         │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────┐│
│  │  数据分析    │  │   AI 分析    │  │ 统计计算││
│  │   (core)     │  │    (ai)      │  │(analyzer)││
│  └──────────────┘  └──────────────┘  └─────────┘│
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                     数据层                         │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────┐│
│  │  数据抓取    │  │   存储管理    │  │  数据   ││
│  │  (crawler)   │  │  (storage)   │  │  模型   ││
│  └──────────────┘  └──────────────┘  └─────────┘│
└─────────────────────────────────────────────────────┘
```

#### 依赖注入模式

```python
# 使用 AppContext 进行依赖注入
from trendradar.context import AppContext
from trendradar.core import load_config

config = load_config()
ctx = AppContext(config)

# 所有操作通过 ctx 进行
storage = ctx.get_storage_manager()
stats, total = ctx.count_frequency(results, word_groups, ...)
html_file = ctx.generate_html(stats, ...)
```

### 2.3 核心类图

```
┌─────────────────┐
│   AppContext    │
│  (应用上下文)   │
└────────┬────────┘
         │
    ┌────┴────┬────────────┐
    │         │            │
┌───▼──┐ ┌───▼───┐ ┌────▼────┐
│Storage│ │AIAnalyzer│ │Notification│
│Manager│ │         │ │Dispatcher │
└───┬──┘ └────┬───┘ └────┬────┘
    │        │            │
┌───▼────────▼────────────▼───┐
│    StorageBackend (ABC)      │
├───────────────────────────────┤
│ + save_news_data()           │
│ + get_today_all_data()      │
│ + detect_new_titles()       │
│ + save_html_report()        │
│ + has_pushed_today()        │
│ + record_push()             │
└──────────────────────────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌──▼────┐
│Local  │ │Remote │
│Storage│ │Storage│
└───────┘ └───────┘
```

---

## 三、核心模块详解

### 3.1 应用入口 (__main__.py)

#### NewsAnalyzer 类

```python
class NewsAnalyzer:
    """新闻分析器 - 主程序类"""
    
    def __init__(self, config: Optional[Dict] = None):
        """
        初始化
        - 加载配置
        - 创建 AppContext
        - 初始化存储管理器
        - 初始化数据抓取器
        """
        self.ctx = AppContext(config)
        self.storage_manager = self.ctx.get_storage_manager()
        self.data_fetcher = DataFetcher(self.proxy_url)
    
    def run(self):
        """主执行流程"""
        # 1. 爬取热榜数据
        results, id_to_name, failed_ids = self._crawl_data()
        
        # 2. 爬取 RSS 数据
        rss_items, rss_new_items, raw_rss_items = self._crawl_rss_data()
        
        # 3. 数据分析流水线
        stats, html_file, ai_result = self._run_analysis_pipeline(...)
        
        # 4. 发送通知
        self._send_notification_if_needed(...)
```

#### 执行流程图

```
开始
  │
  ▼
[加载配置]
  │
  ▼
[初始化 AppContext]
  │
  ▼
[初始化存储管理器]
  │
  ▼
[初始化数据抓取器]
  │
  ├─→ [爬取热榜数据]
  │      └─→ DataFetcher.crawl_websites()
  │
  ├─→ [爬取 RSS 数据]
  │      └─→ RSSFetcher.fetch_all()
  │
  ▼
[数据处理与分析]
  │
  ├─→ [检测新增标题]
  ├─→ [关键词统计]
  ├─→ [AI 智能分析] (可选)
  └─→ [生成 HTML 报告]
  │
  ▼
[发送通知] (可选)
  │
  ├─→ 飞书
  ├─→ 钉钉
  ├─→ 企业微信
  ├─→ Telegram
  ├─→ 邮件
  └─→ 其他渠道
  │
  ▼
结束
```

### 3.2 应用上下文 (context.py)

```python
class AppContext:
    """应用上下文 - 统一接口"""
    
    # === 配置访问 ===
    @property
    def timezone(self) -> str:
        return self.config.get("TIMEZONE", "Asia/Shanghai")
    
    @property
    def platforms(self) -> List[Dict]:
        return self.config.get("PLATFORMS", [])
    
    @property
    def platform_ids(self) -> List[str]:
        return [p["id"] for p in self.platforms]
    
    # === 时间操作 ===
    def get_time(self) -> datetime:
        return get_configured_time(self.timezone)
    
    def format_date(self) -> str:
        return format_date_folder(timezone=self.timezone)
    
    def format_time(self) -> str:
        return format_time_filename(timezone=self.timezone)
    
    # === 存储操作 ===
    def get_storage_manager(self):
        if self._storage_manager is None:
            self._storage_manager = get_storage_manager(...)
        return self._storage_manager
    
    # === 统计分析 ===
    def count_frequency(self, results, word_groups, filter_words, ...):
        return count_word_frequency(
            results=results,
            word_groups=word_groups,
            filter_words=filter_words,
            ...
        )
    
    # === 报告生成 ===
    def generate_html(self, stats, total_titles, ...):
        return generate_html_report(...)
    
    # === 通知发送 ===
    def create_notification_dispatcher(self) -> NotificationDispatcher:
        return NotificationDispatcher(
            config=self.config,
            get_time_func=self.get_time,
            split_content_func=self.split_content,
            translator=translator,
        )
```

### 3.3 存储管理器 (storage/manager.py)

```python
class StorageManager:
    """存储管理器 - 统一存储接口"""
    
    def __init__(self, backend_type: str = "auto", ...):
        """
        Args:
            backend_type: "local" | "remote" | "auto"
            data_dir: 本地数据目录
            enable_txt: 是否启用 TXT 快照
            enable_html: 是否启用 HTML 报告
            remote_config: S3 兼容配置
        """
        self.backend_type = backend_type
        self.data_dir = data_dir
        self._backend = None
    
    def _resolve_backend_type(self) -> str:
        """解析实际使用的后端类型"""
        if self.backend_type == "auto":
            if StorageManager.is_github_actions():
                if self._has_remote_config():
                    return "remote"
                else:
                    return "local"
            else:
                return "local"
        return self.backend_type
    
    def get_backend(self) -> StorageBackend:
        """获取存储后端实例（延迟初始化）"""
        if self._backend is None:
            resolved_type = self._resolve_backend_type()
            if resolved_type == "remote":
                self._backend = self._create_remote_backend()
            else:
                self._backend = self._create_local_backend()
        return self._backend
    
    # 统一接口（委托给实际后端）
    def save_news_data(self, data: NewsData) -> bool:
        return self.get_backend().save_news_data(data)
    
    def get_today_all_data(self, date: Optional[str] = None):
        return self.get_backend().get_today_all_data(date)
    
    def detect_new_titles(self, current_data: NewsData):
        return self.get_backend().detect_new_titles(current_data)
    
    def save_html_report(self, html_content: str, filename: str):
        return self.get_backend().save_html_report(html_content, filename)
```

### 3.4 通知调度器 (notification/dispatcher.py)

```python
class NotificationDispatcher:
    """统一的多账号通知调度器"""
    
    def __init__(self, config, get_time_func, split_content_func, translator=None):
        self.config = config
        self.get_time_func = get_time_func
        self.split_content_func = split_content_func
        self.max_accounts = config.get("MAX_ACCOUNTS_PER_CHANNEL", 3)
        self.translator = translator
    
    def dispatch_all(self, report_data, report_type, ...):
        """分发通知到所有已配置的渠道"""
        results = {}
        
        # 执行翻译（如果启用）
        report_data, rss_items, rss_new_items = self._translate_content(...)
        
        # 飞书
        if self.config.get("FEISHU_WEBHOOK_URL"):
            results["feishu"] = self._send_feishu(...)
        
        # 钉钉
        if self.config.get("DINGTALK_WEBHOOK_URL"):
            results["dingtalk"] = self._send_dingtalk(...)
        
        # 企业微信
        if self.config.get("WEWORK_WEBHOOK_URL"):
            results["wework"] = self._send_wework(...)
        
        # Telegram（需配对验证）
        if self.config.get("TELEGRAM_BOT_TOKEN") and self.config.get("TELEGRAM_CHAT_ID"):
            results["telegram"] = self._send_telegram(...)
        
        # ... 其他渠道
        
        return results
    
    def _send_to_multi_accounts(self, channel_name, config_value, send_func, **kwargs):
        """通用多账号发送逻辑"""
        accounts = parse_multi_account_config(config_value)
        accounts = limit_accounts(accounts, self.max_accounts, channel_name)
        results = []
        
        for i, account in enumerate(accounts):
            if account:
                account_label = f"账号{i+1}" if len(accounts) > 1 else ""
                result = send_func(account, account_label=account_label, **kwargs)
                results.append(result)
        
        return any(results) if results else False
```

---

## 四、数据模型

### 4.1 核心数据模型 (storage/base.py)

```python
@dataclass
class NewsItem:
    """新闻条目数据模型（热榜数据）"""
    
    # 基础信息
    title: str              # 新闻标题
    source_id: str          # 来源平台ID（如 toutiao, baidu）
    source_name: str = ""   # 来源平台名称（运行时使用，数据库不存储）
    rank: int = 0           # 当前排名
    url: str = ""           # 链接 URL
    mobile_url: str = ""    # 移动端 URL
    crawl_time: str = ""    # 抓取时间（HH:MM 格式）
    
    # 统计信息（用于分析）
    ranks: List[int] = field(default_factory=list)      # 历史排名列表
    first_time: str = ""                                 # 首次出现时间
    last_time: str = ""                                  # 最后出现时间
    count: int = 1                                       # 出现次数
    rank_timeline: List[Dict[str, Any]] = field(default_factory=list)
    # 完整排名时间线：[{"time": "09:30", "rank": 1}, {"time": "10:00", "rank": 2}]
    # None 表示脱榜：[{"time": "11:00", "rank": None}]
    
    def to_dict(self) -> Dict[str, Any]:
        """转换为字典"""
        return {
            "title": self.title,
            "source_id": self.source_id,
            "source_name": self.source_name,
            "rank": self.rank,
            "url": self.url,
            "mobile_url": self.mobile_url,
            "crawl_time": self.crawl_time,
            "ranks": self.ranks,
            "first_time": self.first_time,
            "last_time": self.last_time,
            "count": self.count,
            "rank_timeline": self.rank_timeline,
        }


@dataclass
class RSSItem:
    """RSS 条目数据模型"""
    
    title: str
    feed_id: str               # RSS 源 ID（如 "hacker-news"）
    feed_name: str = ""        # RSS 源名称
    url: str = ""
    published_at: str = ""     # RSS 发布时间（ISO 格式）
    summary: str = ""          # 摘要/描述
    author: str = ""
    crawl_time: str = ""       # 抓取时间
    
    # 统计信息
    first_time: str = ""
    last_time: str = ""
    count: int = 1


@dataclass
class NewsData:
    """新闻数据集合"""
    
    date: str                                 # 日期（YYYY-MM-DD）
    crawl_time: str                           # 抓取时间（HH:MM）
    items: Dict[str, List[NewsItem]]           # 按来源分组的新闻
    id_to_name: Dict[str, str] = field(default_factory=dict)   # ID到名称映射
    failed_ids: List[str] = field(default_factory=list)        # 失败的ID
    
    def get_total_count(self) -> int:
        """获取新闻总数"""
        return sum(len(news_list) for news_list in self.items.values())
    
    def merge_with(self, other: "NewsData") -> "NewsData":
        """合并另一个 NewsData 到当前数据"""
        # 合并规则：
        # - 相同 source_id + title 的新闻合并排名历史
        # - 更新 last_time 和 count
        # - 保留较早的 first_time
        ...
```

### 4.2 数据存储结构

```
output/
├── news/                              # 热榜数据（SQLite）
│   └── 2025-12-27.db
│       └── 表结构：
│           ├── news (热榜新闻表)
│           │   ├── id (PK)
│           │   ├── title
│           │   ├── source_id
│           │   ├── rank
│           │   ├── url
│           │   ├── mobile_url
│           │   ├── crawl_time
│           │   ├── ranks (JSON)
│           │   ├── first_time
│           │   ├── last_time
│           │   ├── count
│           │   └── rank_timeline (JSON)
│           │
│           ├── push_records (推送记录表)
│           │   ├── date
│           │   ├── report_type
│           │   └── pushed_at
│           │
│           └── ai_analysis_records (AI 分析记录表)
│               ├── date
│               ├── mode (daily/current/incremental)
│               └── analyzed_at
│
├── rss/                               # RSS 数据（SQLite）
│   └── 2025-12-27.db
│       └── 表结构：
│           ├── rss (RSS 条目表)
│           │   ├── id (PK)
│           │   ├── title
│           │   ├── feed_id
│           │   ├── url
│           │   ├── published_at
│           │   ├── summary
│           │   ├── author
│           │   ├── crawl_time
│           │   ├── first_time
│           │   ├── last_time
│           │   └── count
│
├── html/                              # HTML 报告
│   └── 2025-12-27/
│       ├── 12-30.当日汇总.html
│       ├── 12-30.current.html
│       └── 12-30.index.html
│
├── txt/                               # TXT 快照
│   └── 2025-12-27/
│       └── 12-30.txt
│
└── index.html                         # 当日汇总（根目录）
```

---

## 五、配置系统

### 5.1 配置文件格式

```yaml
# config/config.yaml

# 1. 基础设置
app:
  timezone: "Asia/Shanghai"
  show_version_update: true

# 2. 热榜平台
platforms:
  enabled: true
  sources:
    - id: "weibo"
      name: "微博"
    - id: "zhihu"
      name: "知乎"

# 3. RSS 订阅
rss:
  enabled: true
  freshness_filter:
    enabled: true
    max_age_days: 3
  feeds:
    - id: "hacker-news"
      name: "Hacker News"
      url: "https://hnrss.org/frontpage"

# 4. 报告模式
report:
  mode: "current"  # daily | current | incremental
  display_mode: "keyword"  # keyword | platform
  sort_by_position_first: false
  rank_threshold: 5
  max_news_per_keyword: 0

# 5. 推送内容控制
display:
  region_order:
    - new_items
    - hotlist
    - rss
    - standalone
    - ai_analysis
  regions:
    hotlist: true
    new_items: true
    rss: false
    standalone: false
    ai_analysis: true
  standalone:
    platforms: []
    rss_feeds: []
    max_items: 20

# 6. 推送通知
notification:
  enabled: true
  push_window:
    enabled: false
    start: "20:00"
    end: "22:00"
    once_per_day: true
  channels:
    feishu:
      webhook_url: ""
    dingtalk:
      webhook_url: ""
    wework:
      webhook_url: ""
      msg_type: "markdown"
    telegram:
      bot_token: ""
      chat_id: ""
    email:
      from: ""
      password: ""
      to: ""
    # ... 其他渠道

# 7. 存储配置
storage:
  backend: "auto"  # local | remote | auto
  formats:
    sqlite: true
    txt: false
    html: true
  local:
    data_dir: "output"
    retention_days: 0
  remote:
    retention_days: 0
    endpoint_url: ""
    bucket_name: ""
    access_key_id: ""
    secret_access_key: ""
    region: ""
  pull:
    enabled: false
    days: 7

# 8. AI 模型配置
ai:
  model: "deepseek/deepseek-chat"
  api_key: ""
  api_base: ""
  timeout: 120
  temperature: 1.0
  max_tokens: 5000
  num_retries: 1
  fallback_models: []

# 9. AI 分析功能
ai_analysis:
  enabled: true
  analysis_window:
    enabled: false
    start: "12:00"
    end: "21:00"
    once_per_day: false
  language: "Chinese"
  prompt_file: "ai_analysis_prompt.txt"
  mode: "follow_report"  # follow_report | daily | current | incremental
  max_news_for_analysis: 60
  include_rss: false
  include_rank_timeline: true

# 10. AI 翻译功能
ai_translation:
  enabled: false
  language: "English"
  prompt_file: "ai_translation_prompt.txt"

# 11. 高级设置
advanced:
  debug: false
  crawler:
    request_interval: 2000
    use_proxy: false
    default_proxy: "http://127.0.0.1:10801"
  weight:
    rank: 0.6
    frequency: 0.3
    hotness: 0.1
  max_accounts_per_channel: 3
```

### 5.2 关键词配置 (frequency_words.txt)

```txt
# ═══════════════════════════════════════════════════════════════
#                    频率词配置文件
#                         Version: 1.1.0
# ═══════════════════════════════════════════════════════════════

# ───────────────────────────────────────────────────────────────
#                        全局过滤区
# ───────────────────────────────────────────────────────────────
# 在这里写入你不想看到的词，每行一个。

[GLOBAL_FILTER]
广告
震惊


# ───────────────────────────────────────────────────────────────
#                        词组定义区
# ───────────────────────────────────────────────────────────────
# 每个词组用空行分隔，同一词组内的关键词是"或"的关系。

[WORD_GROUPS]

# 基础用法：直接写关键词
华为

# 多个关键词归为一组
华为
鸿蒙
任正非

# 给词组起个名字（推荐）
[华为]
华为
鸿蒙
任正非

# 进阶用法：正则表达式
/\bai\b/i => AI

# 必须词（需同时包含）
+苹果
+发布会

# 过滤词（仅限当前词组）
[苹果]
苹果
!水果
!果园

# 限制显示条数
[科技新闻]
科技
@5
```

### 5.3 配置加载代码

```python
# trendradar/core/__init__.py
from trendradar.core.config import load_config

# 加载配置
config = load_config()

# 访问配置
platforms = config.get("PLATFORMS", [])
timezone = config.get("TIMEZONE", "Asia/Shanghai")
```

---

## 六、开发指南

### 6.1 开发环境搭建

```bash
# 1. 克隆项目
git clone https://github.com/sansan0/TrendRadar.git
cd TrendRadar

# 2. 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或
venv\Scripts\activate  # Windows

# 3. 安装开发依赖
pip install -e ".[dev]"

# 4. 安装代码格式化工具
pip install black isort flake8 mypy

# 5. 配置 pre-commit（可选）
pip install pre-commit
pre-commit install
```

### 6.2 代码规范

```bash
# 格式化代码
black trendradar/
isort trendradar/

# 代码检查
flake8 trendradar/

# 类型检查
mypy trendradar/
```

### 6.3 添加新的数据源

#### 6.3.1 添加热榜平台

1. 在 `config/config.yaml` 中添加平台配置

```yaml
platforms:
  sources:
    - id: "new_platform"
      name: "新平台"
```

2. 如果需要自定义抓取逻辑，在 `trendradar/crawler/fetcher.py` 中扩展

```python
class DataFetcher:
    def fetch_custom_platform(self, id_value: str) -> Dict:
        """自定义平台抓取逻辑"""
        # 实现抓取逻辑
        pass
```

#### 6.3.2 添加 RSS 源

在 `config/config.yaml` 中添加 RSS 源

```yaml
rss:
  feeds:
    - id: "custom-rss"
      name: "自定义 RSS"
      url: "https://example.com/feed.xml"
      max_items: 50
      enabled: true
      max_age_days: 7
```

### 6.4 添加新的通知渠道

#### 步骤 1：在 `trendradar/notification/senders.py` 中实现发送函数

```python
def send_to_custom_channel(
    webhook_url: str,
    report_data: Dict,
    report_type: str,
    update_info: Optional[Dict],
    proxy_url: Optional[str],
    mode: str = "daily",
    account_label: str = "",
    batch_size: int = 4000,
    batch_interval: float = 1.0,
    split_content_func: Optional[Callable] = None,
    rss_items: Optional[List[Dict]] = None,
    rss_new_items: Optional[List[Dict]] = None,
    ai_analysis: Optional[AIAnalysisResult] = None,
    display_regions: Optional[Dict] = None,
    standalone_data: Optional[Dict] = None,
) -> bool:
    """发送到自定义渠道"""
    try:
        # 1. 渲染内容
        content = render_custom_content(...)
        
        # 2. 分批发送
        batches = split_content_func(content, batch_size)
        for i, batch in enumerate(batches):
            # 3. 构造请求
            payload = {
                "title": f"{report_type}",
                "content": batch,
            }
            
            # 4. 发送请求
            response = requests.post(webhook_url, json=payload, proxies=proxies, timeout=30)
            response.raise_for_status()
        
        print(f"✅ 自定义渠道{account_label} 通知发送成功")
        return True
    except Exception as e:
        print(f"❌ 自定义渠道{account_label} 通知发送失败: {e}")
        return False
```

#### 步骤 2：在 `trendradar/notification/dispatcher.py` 中添加调度逻辑

```python
class NotificationDispatcher:
    def dispatch_all(self, ...):
        results = {}
        
        # 添加自定义渠道
        if self.config.get("CUSTOM_WEBHOOK_URL"):
            results["custom"] = self._send_custom(
                report_data, report_type, update_info, proxy_url, mode,
                rss_items, rss_new_items, ai_analysis, display_regions, standalone_data
            )
        
        return results
    
    def _send_custom(self, ...):
        """发送到自定义渠道（支持多账号）"""
        display_regions = display_regions or {}
        if not display_regions.get("HOTLIST", True):
            report_data = {"stats": [], "failed_ids": [], "new_titles": [], "id_to_name": {}}
        
        return self._send_to_multi_accounts(
            channel_name="自定义渠道",
            config_value=self.config["CUSTOM_WEBHOOK_URL"],
            send_func=lambda url, account_label: send_to_custom_channel(
                webhook_url=url,
                report_data=report_data,
                report_type=report_type,
                update_info=update_info,
                proxy_url=proxy_url,
                mode=mode,
                account_label=account_label,
                batch_size=self.config.get("CUSTOM_BATCH_SIZE", 4000),
                batch_interval=self.config.get("BATCH_SEND_INTERVAL", 1.0),
                split_content_func=self.split_content_func,
                rss_items=rss_items if display_regions.get("RSS", True) else None,
                rss_new_items=rss_new_items if display_regions.get("RSS", True) else None,
                ai_analysis=ai_analysis if display_regions.get("AI_ANALYSIS", True) else None,
                display_regions=display_regions,
                standalone_data=standalone_data if display_regions.get("STANDALONE", False) else None,
            ),
        )
```

#### 步骤 3：更新配置文件

```yaml
notification:
  channels:
    custom:
      webhook_url: ""
```

### 6.5 添加新的存储后端

#### 步骤 1：创建存储后端类

```python
# trendradar/storage/custom.py

from typing import Dict, List, Optional, Any
from trendradar.storage.base import StorageBackend, NewsData, RSSData

class CustomStorageBackend(StorageBackend):
    """自定义存储后端"""
    
    def __init__(self, config: Dict):
        self.config = config
        self.backend_name = "Custom"
        self._init_connection()
    
    def _init_connection(self):
        """初始化连接"""
        pass
    
    def save_news_data(self, data: NewsData) -> bool:
        """保存新闻数据"""
        try:
            # 实现保存逻辑
            return True
        except Exception as e:
            print(f"保存新闻数据失败: {e}")
            return False
    
    def get_today_all_data(self, date: Optional[str] = None) -> Optional[NewsData]:
        """获取当天所有数据"""
        # 实现读取逻辑
        return None
    
    def detect_new_titles(self, current_data: NewsData) -> Dict:
        """检测新增标题"""
        # 实现检测逻辑
        return {}
    
    def save_html_report(self, html_content: str, filename: str, is_summary: bool = False) -> Optional[str]:
        """保存 HTML 报告"""
        # 实现保存逻辑
        return None
    
    def is_first_crawl_today(self, date: Optional[str] = None) -> bool:
        """检查是否是第一次抓取"""
        # 实现检查逻辑
        return True
    
    def cleanup(self) -> None:
        """清理资源"""
        pass
    
    def cleanup_old_data(self, retention_days: int) -> int:
        """清理过期数据"""
        return 0
    
    @property
    def supports_txt(self) -> bool:
        return False
    
    # === 推送记录相关方法 ===
    
    def has_pushed_today(self, date: Optional[str] = None) -> bool:
        return False
    
    def record_push(self, report_type: str, date: Optional[str] = None) -> bool:
        return True
    
    def has_ai_analyzed_today(self, date: Optional[str] = None) -> bool:
        return False
    
    def record_ai_analysis(self, analysis_mode: str, date: Optional[str] = None) -> bool:
        return True
```

#### 步骤 2：在 `trendradar/storage/manager.py` 中注册

```python
class StorageManager:
    def _resolve_backend_type(self) -> str:
        """解析实际使用的后端类型"""
        if self.backend_type == "custom":
            return "custom"
        # ... 其他逻辑
    
    def get_backend(self) -> StorageBackend:
        if self._backend is None:
            resolved_type = self._resolve_backend_type()
            if resolved_type == "custom":
                self._backend = self._create_custom_backend()
            # ... 其他逻辑
        return self._backend
    
    def _create_custom_backend(self) -> Optional[StorageBackend]:
        """创建自定义后端"""
        try:
            from trendradar.storage.custom import CustomStorageBackend
            return CustomStorageBackend(self.remote_config)
        except ImportError as e:
            print(f"自定义后端导入失败: {e}")
            return None
```

---

## 七、测试与调试

### 7.1 单元测试

```python
# tests/test_storage.py

import pytest
from trendradar.storage.base import NewsData, NewsItem

def test_news_item_to_dict():
    """测试 NewsItem 转字典"""
    item = NewsItem(
        title="测试标题",
        source_id="weibo",
        rank=1,
        url="https://example.com"
    )
    
    result = item.to_dict()
    
    assert result["title"] == "测试标题"
    assert result["source_id"] == "weibo"
    assert result["rank"] == 1


def test_news_data_merge():
    """测试 NewsData 合并"""
    data1 = NewsData(
        date="2025-12-27",
        crawl_time="12-30",
        items={
            "weibo": [
                NewsItem(title="标题1", source_id="weibo", rank=1)
            ]
        }
    )
    
    data2 = NewsData(
        date="2025-12-27",
        crawl_time="13-00",
        items={
            "weibo": [
                NewsItem(title="标题1", source_id="weibo", rank=2),
                NewsItem(title="标题2", source_id="weibo", rank=3)
            ]
        }
    )
    
    merged = data1.merge_with(data2)
    
    assert len(merged.items["weibo"]) == 2
    assert merged.items["weibo"][0].ranks == [1, 2]
```

### 7.2 运行测试

```bash
# 运行所有测试
pytest

# 运行特定文件
pytest tests/test_storage.py

# 运行特定测试
pytest tests/test_storage.py::test_news_item_to_dict

# 查看覆盖率
pytest --cov=trendradar --cov-report=html
```

### 7.3 调试技巧

#### 7.3.1 启用调试模式

```yaml
# config/config.yaml
advanced:
  debug: true
```

#### 7.3.2 使用 logging

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

logger.debug("调试信息")
logger.info("普通信息")
logger.warning("警告信息")
logger.error("错误信息")
```

#### 7.3.3 断点调试

```python
# 使用 Python pdb
import pdb; pdb.set_trace()

# 或使用 ipdb
import ipdb; ipdb.set_trace()
```

---

## 八、常见开发任务

### 8.1 添加新的数据字段

```python
# 1. 更新数据模型
@dataclass
class NewsItem:
    # ... 现有字段
    custom_field: str = ""  # 新增字段
    
    def to_dict(self) -> Dict[str, Any]:
        result = {
            # ... 现有字段
            "custom_field": self.custom_field,
        }
        return result

# 2. 更新数据库 schema（如果使用 SQLite）
def update_database_schema(db_path: str):
    """更新数据库 schema"""
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    
    # 添加新列
    cursor.execute("ALTER TABLE news ADD COLUMN custom_field TEXT")
    
    conn.commit()
    conn.close()
```

### 8.2 自定义报告模板

```python
# trendradar/report/generator.py

def generate_custom_report(stats, total_titles, ...):
    """生成自定义报告"""
    
    # 1. 准备报告数据
    report_data = prepare_report_data(...)
    
    # 2. 自定义渲染逻辑
    content = f"""
    # 自定义报告标题
    
    ## 摘要
    总新闻数: {total_titles}
    匹配新闻: {len(stats)}
    
    ## 详细内容
    """
    
    for stat in stats:
        content += f"\n### {stat['word']}\n"
        for title in stat['titles'][:5]:
            content += f"- {title['title']}\n"
    
    return content
```

### 8.3 修改 AI 分析提示词

编辑 `config/ai_analysis_prompt.txt`：

```txt
[system]
你是一个专业的新闻分析师，擅长分析热点新闻和舆情态势。

[user]
请分析以下热点新闻数据：

报告模式: {report_mode}
报告类型: {report_type}
当前时间: {current_time}
新闻数量: {news_count}
RSS数量: {rss_count}
监控平台: {platforms}
关注关键词: {keywords}

## 热榜新闻内容
{news_content}

## RSS 订阅内容
{rss_content}

## 分析要求

请按照以下结构返回 JSON 格式的分析结果：

```json
{{
  "core_trends": "核心热点与舆情态势分析...",
  "sentiment_controversy": "舆论风向与争议点...",
  "signals": "异常与弱信号...",
  "rss_insights": "RSS 深度洞察...",
  "outlook_strategy": "研判与策略建议..."
}}
```

分析语言: {language}
```

---

## 九、API 参考

### 9.1 核心接口

#### AppContext

```python
class AppContext:
    """应用上下文"""
    
    # === 配置访问 ===
    @property
    def timezone(self) -> str: ...
    
    @property
    def platforms(self) -> List[Dict]: ...
    
    @property
    def platform_ids(self) -> List[str]: ...
    
    # === 时间操作 ===
    def get_time(self) -> datetime: ...
    
    def format_date(self) -> str: ...
    
    def format_time(self) -> str: ...
    
    # === 存储操作 ===
    def get_storage_manager(self) -> StorageManager: ...
    
    def get_output_path(self, subfolder: str, filename: str) -> str: ...
    
    # === 数据处理 ===
    def save_titles(self, results: Dict, id_to_name: Dict, failed_ids: List) -> str: ...
    
    def read_today_titles(self, platform_ids: Optional[List[str]] = None, quiet: bool = False) -> Tuple[Dict, Dict, Dict]: ...
    
    def detect_new_titles(self, platform_ids: Optional[List[str]] = None, quiet: bool = False) -> Dict: ...
    
    def is_first_crawl(self) -> bool: ...
    
    # === 频率词处理 ===
    def load_frequency_words(self, frequency_file: Optional[str] = None) -> Tuple[List[Dict], List[str], List[str]]: ...
    
    def matches_word_groups(self, title: str, word_groups: List[Dict], filter_words: List[str], global_filters: Optional[List[str]] = None) -> bool: ...
    
    # === 统计分析 ===
    def count_frequency(self, results: Dict, word_groups: List[Dict], filter_words: List[str], id_to_name: Dict, ...) -> Tuple[List[Dict], int]: ...
    
    # === 报告生成 ===
    def prepare_report(self, stats: List[Dict], failed_ids: Optional[List] = None, ...) -> Dict: ...
    
    def generate_html(self, stats: List[Dict], total_titles: int, ...) -> str: ...
    
    def render_html(self, report_data: Dict, total_titles: int, ...) -> str: ...
    
    # === 通知发送 ===
    def create_notification_dispatcher(self) -> NotificationDispatcher: ...
    
    def create_push_manager(self) -> PushRecordManager: ...
```

#### StorageManager

```python
class StorageManager:
    """存储管理器"""
    
    def __init__(self, backend_type: str = "auto", data_dir: str = "output", ...): ...
    
    @staticmethod
    def is_github_actions() -> bool: ...
    
    @staticmethod
    def is_docker() -> bool: ...
    
    def get_backend(self) -> StorageBackend: ...
    
    def pull_from_remote(self) -> int: ...
    
    # === 统一接口 ===
    def save_news_data(self, data: NewsData) -> bool: ...
    
    def save_rss_data(self, data: RSSData) -> bool: ...
    
    def get_rss_data(self, date: Optional[str] = None) -> Optional[RSSData]: ...
    
    def get_latest_rss_data(self, date: Optional[str] = None) -> Optional[RSSData]: ...
    
    def detect_new_rss_items(self, current_data: RSSData) -> dict: ...
    
    def get_today_all_data(self, date: Optional[str] = None) -> Optional[NewsData]: ...
    
    def get_latest_crawl_data(self, date: Optional[str] = None) -> Optional[NewsData]: ...
    
    def detect_new_titles(self, current_data: NewsData) -> dict: ...
    
    def save_txt_snapshot(self, data: NewsData) -> Optional[str]: ...
    
    def save_html_report(self, html_content: str, filename: str, is_summary: bool = False) -> Optional[str]: ...
    
    def is_first_crawl_today(self, date: Optional[str] = None) -> bool: ...
    
    def cleanup(self) -> None: ...
    
    def cleanup_old_data(self, retention_days: int) -> int: ...
    
    # === 推送记录 ===
    def has_pushed_today(self, date: Optional[str] = None) -> bool: ...
    
    def record_push(self, report_type: str, date: Optional[str] = None) -> bool: ...
    
    def has_ai_analyzed_today(self, date: Optional[str] = None) -> bool: ...
    
    def record_ai_analysis(self, analysis_mode: str, date: Optional[str] = None) -> bool: ...
```

#### NotificationDispatcher

```python
class NotificationDispatcher:
    """通知调度器"""
    
    def __init__(self, config: Dict, get_time_func: Callable, split_content_func: Callable, translator: Optional["AITranslator"] = None): ...
    
    def dispatch_all(self, report_data: Dict, report_type: str, ...) -> Dict[str, bool]: ...
    
    def dispatch_rss(self, rss_items: List[Dict], feeds_info: Optional[Dict[str, str]] = None, ...) -> Dict[str, bool]: ...
```

#### AIAnalyzer

```python
class AIAnalyzer:
    """AI 分析器"""
    
    def __init__(self, ai_config: Dict[str, Any], analysis_config: Dict[str, Any], get_time_func: Callable, debug: bool = False): ...
    
    def analyze(self, stats: List[Dict], rss_stats: Optional[List[Dict]] = None, ...) -> AIAnalysisResult: ...
```

### 9.2 命令行接口

```bash
# 运行主程序
python -m trendradar

# 查看帮助
python -m trendradar --help

# 测试爬虫
python -m trendradar --test-crawl

# 仅爬取，不推送
python -m trendradar --no-notification

# 指定配置文件
python -m trendradar --config /path/to/config.yaml

# MCP 服务器
python -m mcp_server.server
```

---

## 附录

### A. 错误码

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| CONFIG_ERROR | 配置文件错误 | 检查 config.yaml 语法 |
| NETWORK_ERROR | 网络请求失败 | 检查网络连接或代理配置 |
| STORAGE_ERROR | 存储操作失败 | 检查存储权限和磁盘空间 |
| API_ERROR | AI API 调用失败 | 检查 API Key 和额度 |
| NOTIFICATION_ERROR | 通知发送失败 | 检查通知渠道配置 |

### B. 常见问题

**Q: 如何添加新的热榜平台？**

A: 在 `config/config.yaml` 的 `platforms.sources` 中添加配置，确保平台 ID 在 newsnow API 中可用。

**Q: 如何自定义报告模板？**

A: 参考 `trendradar/report/` 目录下的模板文件，复制并修改。

**Q: 如何调试通知发送？**

A: 启用 `advanced.debug: true` 并设置 `notification.push_window.enabled: false`。

**Q: 如何处理敏感信息？**

A: 使用环境变量或 GitHub Secrets 存储敏感配置，不要提交到代码仓库。

### C. 贡献指南

1. Fork 项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

---

**文档维护者**: sansan0
**最后更新**: 2025-02-01
**项目地址**: https://github.com/sansan0/TrendRadar
