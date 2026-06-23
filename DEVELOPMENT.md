# 开发指南

这是 GUZItest 电商采集系统的开发指南。

## 快速开始

### 1. 环境要求

- Python 3.8 或以上
- pip（Python 包管理工具）
- Git

### 2. 克隆项目

```bash
git clone https://github.com/guzi893894381-del/GUZItest.git
cd GUZItest
```

### 3. 创建虚拟环境（推荐）

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Mac/Linux
python3 -m venv venv
source venv/bin/activate
```

### 4. 安装依赖

```bash
cd backend
pip install -r requirements.txt
```

### 5. 运行项目

```bash
python app.py
```

打开浏览器访问：http://localhost:5000

---

## 项目结构详解

```
GUZItest/
├── backend/                 # 后端代码
│   ├── app.py              # Flask 主应用（**重要**）
│   ├── config.py           # 配置文件
│   ├── database.py         # 数据库初始化
│   ├── requirements.txt    # Python 依赖
│   ├── models/
│   │   ├── __init__.py
│   │   └── product.py      # 商品数据模型
│   └── scrapers/           # 爬虫模块
│       ├── __init__.py
│       ├── base_scraper.py # 基础爬虫类
│       ├── pdd_scraper.py  # 拼多多爬虫
│       ├── alibaba_scraper.py  # 1688 爬虫
│       └── shopee_scraper.py   # 虾皮爬虫
├── frontend/               # 前端代码
│   └── index.html         # 主页
├── database/              # 数据库存储
│   └── products.db       # SQLite 数据库文件
└── README.md             # 项目说明
```

---

## 核心文件说明

### 1. app.py - Flask 主应用

**作用**：定义所有 API 接口，处理业务逻辑

**关键部分**：
- `@app.route()` - 定义路由
- `create_app()` - 创建应用
- `scrapers` - 爬虫字典

**主要 API**：
- `GET /` - 主页信息
- `POST /api/scrape` - 采集商品
- `GET /api/products` - 获取商品列表
- `GET /api/products/search` - 搜索商品
- `GET /api/statistics` - 统计信息

### 2. config.py - 配置文件

**作用**：集中管理项目配置

**配置类**：
- `Config` - 基础配置
- `DevelopmentConfig` - 开发环境
- `ProductionConfig` - 生产环境
- `ScraperConfig` - 爬虫配置

### 3. models/product.py - 商品数据模型

**作用**：定义数据库表结构

**字段**：
- `title` - 商品标题
- `price` - 商品价格
- `platform` - 采集平台
- `source_url` - 原链接
- 等等...

### 4. scrapers/ - 爬虫模块

**结构**：
- `base_scraper.py` - 基类，定义通用方法
- `pdd_scraper.py` - 继承基类，实现拼多多采集
- `alibaba_scraper.py` - 继承基类，实现 1688 采集
- `shopee_scraper.py` - 继承基类，实现虾皮采集

**工作流程**：
```
输入 URL
   ↓
scrapers[platform].parse_url(url)
   ↓
调用对应爬虫的 parse_url() 方法
   ↓
发送 HTTP 请求（get_page()）
   ↓
解析 HTML（BeautifulSoup）
   ↓
提取商品信息（_extract_* 方法）
   ↓
返回标准格式的商品数据
```

---

## 数据库说明

### SQLite 数据库

**文件位置**：`database/products.db`

**表**：`products`

**字段**：
| 字段名 | 类型 | 说明 |
|--------|------|------|
| id | Integer | 主键 |
| title | String | 商品标题 |
| price | Float | 商品价格 |
| platform | String | 采集平台 |
| source_url | String | 原链接 |
| image_url | String | 图片 URL |
| seller_name | String | 商家名称 |
| created_at | DateTime | 创建时间 |
| updated_at | DateTime | 更新时间 |

### 初始化数据库

数据库会在应用首次运行时自动创建，无需手动操作。

---

## API 使用示例

### 1. 采集商品

```bash
curl -X POST http://localhost:5000/api/scrape \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://mobile.pinduoduo.com/goods.html?goods_id=123",
    "platform": "拼多多"
  }'
```

**响应**：
```json
{
  "success": true,
  "message": "采集成功",
  "data": {
    "id": 1,
    "title": "商品标题",
    "price": 99.99,
    "platform": "拼多多",
    "source_url": "...",
    "created_at": "2026-06-23 10:00:00"
  }
}
```

### 2. 获取商品列表

```bash
curl http://localhost:5000/api/products?page=1&per_page=10
```

### 3. 搜索商品

```bash
curl http://localhost:5000/api/products/search?keyword=手机
```

### 4. 获取统计信息

```bash
curl http://localhost:5000/api/statistics
```

---

## 常见问题

### Q1: 如何修改数据库？

A: 修改 `backend/config.py` 中的 `SQLALCHEMY_DATABASE_URI` 字段。

### Q2: 如何添加新的平台？

A: 
1. 在 `backend/scrapers/` 中创建新文件（如 `new_platform_scraper.py`）
2. 继承 `BaseScraper` 类
3. 实现 `get_platform_name()` 和 `parse_url()` 方法
4. 在 `app.py` 中的 `scrapers` 字典中添加新爬虫

### Q3: 爬虫无法采集怎么办？

A: 
- 检查网站是否有反爬机制
- 尝试修改 `User-Agent` 或添加延迟
- 使用代理 IP（参见 `config.py`）
- 对于 JavaScript 渲染的页面，考虑使用 Selenium

### Q4: 如何部署到生产环境？

A:
1. 修改 `config.py` 中的 `ProductionConfig`
2. 使用 Gunicorn 运行：`gunicorn app:app`
3. 使用 Nginx 作为反向代理
4. 配置 HTTPS

---

## 下一步改进计划

- [ ] 支持 Selenium 动态爬取
- [ ] 集成 IP 代理池
- [ ] 添加验证码识别
- [ ] 前端 Vue.js 重构
- [ ] 用户认证系统
- [ ] 多线程采集
- [ ] 定时任务
- [ ] 数据导出功能

---

## 联系方式

如有问题，请在 GitHub Issues 中提出。

---

**最后更新**：2026-06-23
