# 中国市场 SaaS 产品技术选型参考

> 适用：面向中国用户的个人开发者 SaaS 产品。记录经过验证的选型决策和坑。

---

## 分发形态决策

### Web + PWA（推荐）vs 原生 App

**结论：个人开发者做面向中国用户的金融/工具类产品，首选 Web + PWA，不做原生 App。**

原因：
- 金融类 App 在国内 App Store 审核极难，需要金融牌照或合规资质
- Android 国内应用市场（华为/小米/OPPO）各有审核要求，维护成本高
- Web 响应式 + PWA 覆盖手机/桌面，用户可"添加到桌面"获得类 App 体验
- 一套代码，迭代速度远快于原生

**PWA 限制：**
- iOS Safari 对 PWA 支持有限（推送通知不可用，离线缓存受限）
- 安卓 Chrome 支持完整

**替代推送方案：** Telegram Bot 或微信公众号模板消息（用户主动绑定）

---

## 支付方案

### 个人开发者聚合支付

面向中国市场的个人开发者，推荐使用聚合支付服务商：

- 支持：微信支付 + 支付宝
- 无需营业执照，个人开发者可用
- 费率：约 0.6% 手续费
- 接入方式：REST API

**接入流程：**
1. 注册账号，实名认证
2. 创建应用，获取 appid + appsecret
3. 调用支付接口生成支付链接
4. 配置回调 URL，验签后更新订单状态

**验签（Python）：**
```python
import hashlib

def verify_sign(params: dict, appsecret: str) -> bool:
    sign = params.pop('sign', '')
    sorted_str = '&'.join(f"{k}={v}" for k, v in sorted(params.items()) if v)
    raw = sorted_str + appsecret
    expected = hashlib.md5(raw.encode()).hexdigest()
    return sign == expected
```

**替代方案（有营业执照后）：**
- 微信支付官方商户号
- 支付宝开放平台

---

## 中国金融数据免费 API

### akshare（指数/股票/基金行情）

```bash
pip install akshare
```

```python
import akshare as ak

# 沪深300日行情
df = ak.stock_zh_index_daily(symbol="sh000300")

# 基金净值（场外）
df = ak.fund_open_fund_info_em(fund="000001", indicator="单位净值走势")
```

- 完全免费，无需注册
- 数据来源：东方财富、同花顺等
- 更新频率：日级别

### 天天基金净值（非官方 API）

```
GET https://fundgz.1234567.com.cn/js/{fund_code}.js
```

返回 JSONP 格式，包含实时估算净值（交易时间内）。

### 财经新闻免费来源

| 来源 | URL | 格式 |
|---|---|---|
| 东方财富快讯 | `https://newsapi.eastmoney.com/kuaixun/v1/getlist_102_ajaxResult_50_1_.html` | JSON |
| 新浪财经 | `https://feed.mix.sina.com.cn/api/roll/get?pageid=153&lid=2516&num=50&page=1` | JSON |

---

## 标准技术栈（Python SaaS）

```
后端:    Python 3.11 + FastAPI + SQLAlchemy 2.0 + APScheduler
数据库:  PostgreSQL 15 + Redis 7
爬虫:    httpx + Playwright（动态页面）+ BeautifulSoup4
认证:    python-jose（JWT）+ bcrypt
前端:    React 18 + TypeScript + Vite + TailwindCSS + Recharts
PWA:     vite-plugin-pwa
状态:    Zustand
部署:    Docker Compose + Nginx
```

---

## LLM API 格式切换

中国 SaaS 项目常需要在多个 LLM 提供商之间切换。关键认知：**DeepSeek、GLM、Gemini 等都实现了 OpenAI 兼容接口**，只有 Anthropic 用自己的格式。

### 两种格式对比

| 格式 | 认证头 | 端点 | 响应路径 | 适用提供商 |
|---|---|---|---|---|
| `anthropic` | `x-api-key` + `anthropic-version` | `/messages` | `content[0].text` | Anthropic |
| `openai` | `Authorization: Bearer` | `/chat/completions` | `choices[0].message.content` | DeepSeek、GLM、Gemini、Mistral、Groq 及大多数中转 |

### 推荐配置方式（.env 三行切换）

```bash
# 切换到 DeepSeek
LLM_BASE_URL=https://api.deepseek.com/v1
LLM_MODEL=deepseek-chat
LLM_API_FORMAT=openai

# 切换到 GLM（免费额度多）
LLM_BASE_URL=https://open.bigmodel.cn/api/paas/v4
LLM_MODEL=glm-4-flash
LLM_API_FORMAT=openai

# 切换到 Gemini（OpenAI 兼容层）
LLM_BASE_URL=https://generativelanguage.googleapis.com/v1beta/openai
LLM_MODEL=gemini-2.0-flash
LLM_API_FORMAT=openai
```

### llm_client.py 标准实现模式

```python
def _build_request(system, user, max_tokens):
    if settings.LLM_API_FORMAT == "anthropic":
        return (
            f"{settings.LLM_BASE_URL}/messages",
            {"x-api-key": settings.LLM_API_KEY, "anthropic-version": "2023-06-01"},
            {"model": settings.LLM_MODEL, "max_tokens": max_tokens,
             "system": system, "messages": [{"role": "user", "content": user}]},
        )
    else:  # openai 兼容
        return (
            f"{settings.LLM_BASE_URL}/chat/completions",
            {"Authorization": f"Bearer {settings.LLM_API_KEY}"},
            {"model": settings.LLM_MODEL, "max_tokens": max_tokens,
             "messages": [{"role": "system", "content": system},
                          {"role": "user", "content": user}]},
        )

def _extract_text(data):
    if "content" in data:   # Anthropic
        return data["content"][0]["text"].strip()
    return data["choices"][0]["message"]["content"].strip()  # OpenAI
```

**坑：** LLM 返回 JSON 时经常包裹在 ` ```json ``` ` 代码块里，需要剥离后再 `json.loads()`。

---

## 免责声明（金融类产品必须）

所有预测/分析页面必须显示：

> 本系统提供的分析仅供参考，不构成投资建议。投资有风险，入市需谨慎。

---

## 定价参考（2026年国内 SaaS）

工具类/数据类产品常见定价区间：
- 月度：¥19-49
- 年度：月度 × 8-10（7-8折）
- 永久：年度 × 3-5
