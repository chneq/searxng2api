# SearXNG2API

将 SearXNG 公开实例的 HTML 搜索结果转为 JSON API 的 Cloudflare Worker 代理。

## Features

- 从官方实例列表自动筛选高可用实例
- 代理搜索请求（`/search`），HTML → JSON 转换
- 支持普通搜索（`general`）和图片搜索（`images`）
- 支持 GET / POST 请求
- 代理 `/config` 接口
- 查看可用实例列表（`/list`）
- 根路径返回使用说明（`/`）
- 可设置 `BASE_URL` 固定代理某个实例
- 可设置 `MORE_RESULT` 忽略搜索引擎参数以获取更多结果

## 环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `BASE_URL` | 固定使用指定的 SearXNG 实例地址，不自动筛选 | `https://example.com` |
| `MORE_RESULT` | 设为 `enable` 时忽略 `engines` 参数，返回所有引擎结果 | `enable` |

## API

### `GET /`

返回服务状态和使用说明。

```json
{
  "code": 200,
  "message": "SearXNG2API Proxy 服务运行中",
  "usage": {
    "search": "/search?q=<关键词>&categories=general|images",
    "config": "/config",
    "list": "/list"
  }
}
```

### `GET/POST /search?q=<关键词>`

代理搜索请求，返回 JSON 格式结果。

**参数：**

| 参数 | 必填 | 说明 |
|------|------|------|
| `q` | 是 | 搜索关键词 |
| `categories` | 否 | 搜索分类，`general`（默认）或 `images` |
| `engines` | 否 | 指定搜索引擎（当 `MORE_RESULT=enable` 时忽略） |

**响应格式：**

```json
{
  "proxy": "https://实例地址/search?q=...",
  "query": "搜索词",
  "number_of_results": 0,
  "results": [
    {
      "url": "...",
      "title": "...",
      "content": "...",
      "publishedDate": null,
      "engine": "...",
      "category": "general|images"
    }
  ],
  "answers": [],
  "corrections": [],
  "infoboxes": [],
  "suggestions": [],
  "unresponsive_engines": []
}
```

### `GET /config`

代理 SearXNG 实例的配置接口。

### `GET /list`

返回当前可用的实例列表。

```json
{
  "code": 200,
  "data": ["https://instance1.example.com", "https://instance2.example.com"]
}
```

## 实例筛选规则

从 [searx.space](https://searx.space/data/instances.json) 获取实例列表，按以下条件筛选：

| 条件 | 阈值 |
|------|------|
| 网络类型 | `normal` |
| 今日在线率 | ≥ 90% |
| 初始化耗时中位数 | < 5 秒 |
| 搜索耗时中位数 | < 5 秒 |
| 初始化成功率 | ≥ 90% |
| 搜索成功率 | ≥ 80% |

额外规则：
- 黑名单中的实例排除（已知异常实例）
- 请求实例列表超时 10 秒自动放弃
- 无可用实例时返回 500 JSON 错误响应

## 快速开始

### 直接使用

把以下地址填入 Cherry Studio 或其他支持 OpenAI API 格式的应用中：

```
https://searxng2api.svia.workers.dev
```

### 自行部署

1. 注册 [Cloudflare](https://dash.cloudflare.com/) 账号
2. 进入 **Workers & Pages**
3. 新建 Worker
4. 将 `worker.js` 内容复制到编辑器中保存
5. （可选）在面板中设置环境变量 `BASE_URL` 或 `MORE_RESULT`

## 更新记录

### 2025-04-xx（当前 fork）
- 实例筛选条件放宽：在线率从 100% 降至 ≥90%，耗时从 <1s 放宽至 <5s，成功率从 100% 降至 ≥90% / ≥80%
- 移除 `search_go` 引擎成功率要求（原要求 Google 搜索成功率 100%）
- 实例列表请求增加 `AbortSignal.timeout(10000)` 超时保护和 try/catch 异常处理
- 无可用实例时返回 JSON 500 错误信息，替代原来的 401 跳转
- 新增 `/` 根路径，返回 API 使用说明 JSON
- 搜索请求未指定 `categories` 时默认使用 `general`
- HTML 解析时未知分类回退到 `generalParse`，替代原来的 401 拒绝
- `parseUrl` 增加 HTTP/HTTPS 协议解析

### 2025-04-06
- 新增 `MORE_RESULT` 环境变量，控制是否忽略 `engines` 参数

### 2025-04-05
- 解析方式由正则匹配改为 HTMLRewriter API
- 支持 `categories=images` 图片搜索

### 2025-04-04
- 新增 `BASE_URL` 环境变量，可固定代理指定的 SearXNG 服务
- 新增 `/list` 接口，返回可用实例列表

### 2025-03-27
- 新增黑名单列表，排除异常实例
- 优化服务可用性判断逻辑

### 2025-03-24
- 支持忽略指定的 `engines` 参数

### 2025-03-23
- 放宽实例筛选条件（在线率 ≥90%、耗时 <5s、成功率 ≥80%）
- 增加 AbortSignal.timeout 超时处理
- fetch 异常时返回 JSON 错误而非直接崩溃

### 2025-03-22
- 初版发布
