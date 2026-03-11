# datatk-quote-skill

基于 [dataTrack](https://www.datatk.com) QuoteNode REST API 的行情查询技能，支持股票、期货、外汇等多市场实时和历史数据查询。

---

## 目录

- [前置条件](#前置条件)
- [配置](#配置)
- [快速开始](#快速开始)
- [接口一览](#接口一览)
- [命令行使用示例](#命令行使用示例)
- [市场代码](#市场代码)
- [错误处理](#错误处理)
- [参考文档](#参考文档)

---

## 前置条件

1. 安装 **Node.js 18+**（脚本使用原生 `fetch`，无需额外依赖）。
2. 前往 [dataTrack 服务页面](https://www.datatk.com/service) 获取你的 **Endpoint** 和 **API Key**。

---

## 配置

1）复制示例配置文件：

```bash
cp env.example.json env.json
```

2）编辑项目根目录下的 `env.json`，将真实的接入地址和密钥填入：

```json
{
  "endpoint": "https://quote.datatk.com",
  "apiKey": "<你的API Key>"
}
```

说明：
- `env.json` 已在 `.gitignore` 中忽略，**不要把真实 API Key 提交到仓库**。
- `endpoint` 末尾的斜杠会被自动去除。

---

## 快速开始

所有请求使用统一的脚本 `scripts/request.mjs`，通过 `--path` 指定接口路径，通过 `--body` 传入 JSON 请求体：

```bash
node scripts/request.mjs --path /Api/V1/Quotation/Detail --body '{"instrument":"US|AAPL","lang":"en"}'
```

成功时，脚本会将原始 JSON 响应打印到标准输出。  
请求失败时（HTTP 状态码非 `200`），脚本会打印状态码和错误信息后退出。

---

## 安全说明

该技能会携带 API Key 向 QuoteNode REST 发起 HTTPS 请求。为降低被误用为“任意外连/外传工具”的风险，`scripts/request.mjs` 内置了安全约束：

- 仅允许 HTTPS endpoint
- endpoint 域名白名单（默认仅允许 `*.datatk.com`）
- path 必须以 `/Api/` 开头
- 禁止使用 IP 形式的 endpoint

如你使用的是私有网关域名，请在 `scripts/request.mjs` 中调整 allowlist。

---

## 接口一览

| 功能 | 路径 |
| --- | --- |
| 证券详情 | `/Api/V1/Quotation/Detail` |
| 证券列表 | `/Api/V1/Basic/BasicInfo` |
| 逐笔成交 | `/Api/V1/Quotation/Tick` |
| Level-2 买卖盘 | `/Api/V1/Quotation/DepthQuoteL2` |
| 分时图 | `/Api/V1/History/TimeLine` |
| K 线 | `/Api/V1/History/KLine` |
| 经纪商列表 | `/Api/V1/Quotation/Brokers` |
| 经纪商队列 | `/Api/V1/Quotation/Broker` |
| 交易日历 | `/Api/V1/Basic/Holiday` |

---

## 命令行使用示例

### 查询美股实时行情（苹果）

```bash
node scripts/request.mjs \
  --path /Api/V1/Quotation/Detail \
  --body '{"instrument":"US|AAPL","lang":"zh-CN"}'
```

### 查询港股列表（第 1 页，每页 50 条）

```bash
node scripts/request.mjs \
  --path /Api/V1/Basic/BasicInfo \
  --body '{"instrument":"HKEX|1","page":1,"pageSize":50,"lang":"zh-CN"}'
```

### 查询腾讯日 K 线（前复权）

```bash
node scripts/request.mjs \
  --path /Api/V1/History/KLine \
  --body '{"instrument":"HKEX|00700","period":"day","right":1}'
```

### 查询港股 Level-2 买卖盘（5 档）

```bash
node scripts/request.mjs \
  --path /Api/V1/Quotation/DepthQuoteL2 \
  --body '{"instrument":"HKEX|00700|R","depth":5}'
```

### 查询港股交易日历

```bash
node scripts/request.mjs \
  --path /Api/V1/Basic/Holiday \
  --body '{"market":"HKEX","startDate":"2025-01-01","endDate":"2025-12-31","lang":"zh-CN"}'
```

### 查询逐笔成交

```bash
node scripts/request.mjs \
  --path /Api/V1/Quotation/Tick \
  --body '{"instrument":"HKEX|00700|R","page":1,"pageSize":20}'
```

---

## 市场代码

### 股票市场

| 代码 | 说明 |
| --- | --- |
| `SSE` | 上海证券交易所 |
| `SZSE` | 深圳证券交易所 |
| `HKEX` | 香港交易所 |
| `US` | 美国股市 |

### 期货市场（部分）

| 代码 | 说明 |
| --- | --- |
| `SHFE` | 上海期货交易所 |
| `CFFEX` | 中国金融期货交易所 |
| `COMEX` | 纽约商品交易所 |
| `CME` | 芝加哥商业交易所 |

### 外汇

| 代码 | 说明 |
| --- | --- |
| `FOREX` | 外汇市场 |

> 完整的市场代码、证券类型、K 线周期、复权类型等枚举值，见 `references/reference.md`。

---

## instrument 字段格式

`instrument` 参数的格式为 `{市场}|{代码}` 或 `{市场}|{代码}|{数据标志}`。

| 数据标志 | 说明 |
| --- | --- |
| `R` | 实时行情 |
| `D` | 延迟行情 |

**示例：**

- `US|AAPL` — 苹果美股
- `HKEX|00700` — 腾讯港股
- `HKEX|00700|R` — 腾讯港股实时行情
- `COMEX|GC2604` — COMEX 黄金期货合约

---

## 错误处理

HTTP 状态码非 `200` 时，响应体格式如下：

```json
{
  "code": 429,
  "reason": "OPENAPI_QUOTA_EXCEEDED",
  "message": "Rate limit exceeded"
}
```

常见状态码：

| 状态码 | 说明 |
| --- | --- |
| `400` | 请求参数有误 |
| `401` | 认证失败，检查 API Key |
| `403` | 无权限访问该接口或市场 |
| `429` | 超出请求频率限制 |
| `500` | 服务内部错误 |

业务错误码（响应体内 `code` 字段）：

| 业务码 | 说明 |
| --- | --- |
| `0` | 成功 |
| `1001` | 参数校验失败 |
| `1002` | 找不到该证券 |
| `2001` | 无效的 API Key |
| `2003` | 无接口权限 |
| `2004` | 无市场权限 |

---

## 参考文档

| 文档 | 内容 |
| --- | --- |
| [references/openapi.md](references/openapi.md) | 全部接口路径、请求参数说明和示例 |
| [references/reference.md](references/reference.md) | 枚举值、市场代码、错误码 |
| [references/response.md](references/response.md) | 各接口响应字段含义 |
| [references/architecture.md](references/architecture.md) | REST 层架构说明 |
