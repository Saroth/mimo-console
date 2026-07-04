# AGENTS.md

## 项目概述

单文件 Python CLI 工具 (`mimo_stat.py`)，用于查询小米 MiMo 平台 (platform.xiaomimimo.com) 的 token 使用量数据，支持 tmux 状态栏集成。

## 快速开始

```bash
pipx install -e .   # 安装为全局命令
mimo-stat           # 详细输出
mimo-stat -t        # tmux 状态栏单行格式
```

## 配置说明

首次运行会创建 `~/.config/mimo-stat/config.json`，需要填入 cookie 后才能正常使用。Cookie 来源于浏览器登录 `platform.xiaomimimo.com`。

## 架构

- `mimo_stat.py` — 完整应用（配置、API、格式化、CLI）
- `setup.py` — 最小化 setuptools 配置，`psutil` 虽在依赖中但当前代码**未使用**
- 配置文件: `~/.config/mimo-stat/config.json` (cookie, base_url)
- 缓存文件: `~/.config/mimo-stat/cache.json` (30秒有效期，错误响应也会缓存)
- Claude 统计: `~/.claude/stats-cache.json` (外部文件，30分钟有效期)

## API 认证特殊处理

POST 请求需要从 cookie 中提取 `api-platform_ph`，作为 URL 查询参数附加，并进行 URL 编码（特别是 `+` → `%2B`）。详见 `get_ph_from_cookie()` 和 `api_post()`。

## 缓存行为

- `load_cache()` 对于缓存的认证失败返回 `{"error": msg}` — 调用方需检查 `if "error" in cached`
- `save_cache()` 将错误存储在顶层，而非 `data` 键中

## API 端点

基础路径: `https://platform.xiaomimimo.com/api/v1`

| 端点 | 方法 | 说明 |
|------|------|------|
| `/tokenPlan/detail` | GET | 套餐详情（类型、额度、到期时间） |
| `/tokenPlan/usage` | GET | 使用量（已消耗 credits） |
| `/tokenPlan/list` | GET | 套餐列表 |
| `/tokenPlan/subscription/status` | GET | 订阅状态 |
| `/usage/token-plan/list` | POST | 每日使用明细（需带 `api-platform_ph` 参数） |
| `/balance` | GET | 账户余额 |

认证方式: 小米账号 Cookie，通过 `account.xiaomi.com` 登录获取。POST 请求需在 URL 中带 `api-platform_ph` 查询参数。

## Token 到 Credit 转换率

| 模型 | 命中缓存 | 未命中缓存 | 输出 |
|------|----------|------------|------|
| mimo-v2.5-pro | 2.5 | 300 | 600 |
| mimo-v2.5 | 2 | 100 | 200 |
| mimo-v2-pro | 2.5 | 300 | 600 |
| mimo-v2-omni | 2 | 100 | 200 |

## 无测试 / CI

本项目无测试套件、无 linting、无类型检查。修改后直接运行 `mimo-stat` 验证（需要有效 cookie）。
