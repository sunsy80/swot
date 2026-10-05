# SWOT (See what's out there)

> 自用高精细度 **sing-box** 规则集仓库，提供原始 JSON 格式与 GitHub Actions 自动编译生成的 SRS 二进制格式。

由 GitHub 用户 [@sunsy80](https://github.com/sunsy80) 维护。

---

## 规则列表 (Rule Sets)

| 规则名称 | 原始 JSON 规则 | 编译 SRS 规则 | 说明 | 建议路由 |
| :--- | :--- | :--- | :--- | :--- |
| **TradingView 核心服务** | [`tradingview.json`](rules/tradingview.json) | `tradingview.srs` | 行情数据、Pine 脚本引擎、选股器、图表云存储、实时推流等核心业务 | `proxy` |
| **TradingView 广告追踪** | [`tradingview-ads.json`](rules/tradingview-ads.json) | `tradingview-ads.srs` | Snowplow 数据打点、交互行为追踪与广告曝光像素 | `block` |

---

## 包含域名明细

### 1. `tradingview` (核心服务与数据流)
- `pine-facade.tradingview.com`：Pine Script 脚本引擎网关（编译与回测请求调度门面）
- `news-mediator.tradingview.com`：财经快讯流聚合与中介调度
- `scanner.tradingview.com`：选股器/筛选器（多品种指标扫描过滤）
- `charts-storage.tradingview.com`：画线、指标模板与图表布局云存储
- `notifications.tradingview.com`：价格预警与系统通知中心
- `pushstream.tradingview.com`：WebSocket / TCP 实时长连接行情推送
- `data.tradingview.com`：K 线历史 Bar 数据与基本面财务数据接口

### 2. `tradingview-ads` (分析与打点)
- `snowplow-pixel.tradingview.com`：Snowplow 行为分析打点像素（阻断不影响交易与图表功能）

---

## sing-box 客户端配置指南

在 sing-box 的配置文件（如 `config.json`）中按如下方式引用：

```json
{
  "route": {
    "rule_set": [
      {
        "tag": "tv-ads",
        "type": "remote",
        "format": "binary",
        "url": "https://fastly.jsdelivr.net/gh/sunsy80/swot@main/rules/tradingview-ads.srs",
        "download_detour": "proxy",
        "update_interval": "1d"
      },
      {
        "tag": "tv-core",
        "type": "remote",
        "format": "binary",
        "url": "https://fastly.jsdelivr.net/gh/sunsy80/swot@main/rules/tradingview.srs",
        "download_detour": "proxy",
        "update_interval": "1d"
      }
    ],
    "rules": [
      {
        "rule_set": "tv-ads",
        "outbound": "block"
      },
      {
        "rule_set": "tv-core",
        "outbound": "proxy"
      }
    ]
  }
}
```

> **提示**：
> - 直连 GitHub Raw 链接为：`https://raw.githubusercontent.com/sunsy80/swot/main/rules/<规则名>.srs`
> - 国内网络环境下推荐使用 jsDelivr CDN 加速链接：`https://fastly.jsdelivr.net/gh/sunsy80/swot@main/rules/<规则名>.srs`
> - sing-box 匹配遵循**从上到下**原则，拦截规则（`tv-ads`）必须置于代理规则（`tv-core`）之前。

---

## 如何添加并维护新规则

仓库已配置 **GitHub Actions 自动化 CI/CD**（详见 [`.github/workflows/compile.yml`](.github/workflows/compile.yml)）：

1. 在 `rules/` 目录下新建或修改任何 `.json` 文件，例如 `rules/new-rule.json`。
2. 按照 sing-box Headless Rule 规范书写规则：
   ```json
   {
     "version": 2,
     "rules": [
       {
         "domain": [ ... ],
         "domain_suffix": [ ... ]
       }
     ]
   }
   ```
3. 执行 `git push` 推送至 GitHub：
   - GitHub Actions 会在 10 秒内自动拉取最新版 sing-box；
   - 自动将 `rules/*.json` 批量编译为同名 `rules/*.srs`；
   - 自动将生成的二进制 `.srs` 文件 Commit 并 Push 回仓库，无需手动干预。
