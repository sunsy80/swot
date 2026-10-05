# SWOT (See what's out there)

> 自用高精细度 **sing-box** 规则集仓库，提供原始 JSON 格式与 GitHub Actions 自动编译生成的 SRS 二进制格式。

由 GitHub 用户 [@sunsy80](https://github.com/sunsy80) 维护。

---

## 规则列表 (Rule Sets)

| 规则名称 | 原始 JSON 规则 | 编译 SRS 规则 | 说明 | 建议路由 |
| :--- | :--- | :--- | :--- | :--- |
| **TradingView 核心服务** | [`tradingview.json`](rules/tradingview.json) | `tradingview.srs` | 主站入口、静态资源CDN、期权图表、客户端依赖、行情Bar、Pine 脚本引擎、品种选股器前后端、价格预警、图表云存储、实时推流等核心业务 | `proxy` |
| **TradingView 广告与遥测** | [`tradingview-ads.json`](rules/tradingview-ads.json) | `tradingview-ads.srs` | Snowplow 数据打点、交互行为追踪、广告曝光像素与客户端崩溃遥测分析 | `block` |

---

## 包含域名明细

### 1. `tradingview` (核心业务与数据流，共 14 个域名)
- `www.tradingview.com`：主站前端网页服务与核心入口
- `s3.tradingview.com`：S3 静态资源托管节点（JS/CSS 脚本包、图表静态图标等素材）
- `static.tradingview.com`：全局静态文件与 CDN 媒体资源分发
- `tvd-packages.tradingview.com`：桌面端专用依赖包分发接口（TradingView Desktop 组件更新与运行库）
- `pine-facade.tradingview.com`：Pine Script 脚本引擎网关（编译与回测请求调度门面）
- `news-mediator.tradingview.com`：财经快讯流聚合与中介调度
- `scanner.tradingview.com`：选股器/筛选器前端服务（多品种指标扫描过滤）
- `scanner-backend.tradingview.com`：选股器核心计算后端（股票/外汇/加密货币条件批量筛选与计算接口）
- `pricealerts.tradingview.com`：价格预警服务（用户设定的价格警报与触发条件监听）
- `options-charting.tradingview.com`：期权策略与波动率图表计算服务
- `charts-storage.tradingview.com`：画线、指标模板与图表布局云存储
- `notifications.tradingview.com`：价格预警推送与系统通知中心
- `pushstream.tradingview.com`：WebSocket / TCP 实时长连接行情推送
- `data.tradingview.com`：K 线历史 Bar 数据与基本面财务数据接口

### 2. `tradingview-ads` (广告追踪与遥测诊断，共 2 个域名)
- `snowplow-pixel.tradingview.com`：Snowplow 行为分析打点像素（广告曝光与点击流统计）
- `telemetry.tradingview.com`：遥测与诊断数据收集节点（客户端崩溃日志、运行性能及系统统计指标）

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
> - sing-box 匹配遵循**自上而下、先匹配先生效**原则，拦截规则（`tv-ads`）必须置于代理规则（`tv-core`）之前。

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
   - GitHub Actions 会自动拉取最新版 sing-box；
   - 自动将 `rules/*.json` 批量编译为同名 `rules/*.srs`；
   - 自动将生成的二进制 `.srs` 文件 Commit 并 Push 回仓库，无需手动干预。
