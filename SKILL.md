---
name: xhs-aitable-skill
description: 追踪小红书热点笔记并保存到钉钉 AI 表格。当用户提到搜索/追踪/分析小红书热点、选品、爆款笔记，或要把小红书数据保存/整理/记录到表格时使用。
---

# 小红书热点追踪

搜索小红书关键词，提取 topN 笔记数据，并将获取到的数据直接保存到钉钉 AI 表格。

## 前置依赖检查

- 检查python3是否安装，`python3 --version`，确保必须要安装 `python3`
- 检查依赖库是否安装：`python3 -c "import requests, websockets; print('已安装')"`，若报错则运行 `pip3 install requests websockets`

## 注意事项

- 前置依赖检查未完成前，禁止打开浏览器以及小红书页面
- 不要跳过登录检查 — 未登录时 search-feeds 会静默失败
- 不要编造 feed_id、xsec_token、baseId、tableId、fieldId — 必须从返回值提取
- 不要在未确认 fieldId 的情况下写入记录 — 必须先 get_fields 确认字段
- 小红书的内容必须要保存到 AI 表格中，无需询问，除非用户明确不要保存到 AI 表格
- 用户说"找/搜/追踪"时，优先理解为电商选品场景，用 `--sort-by 最多点赞`

## CLI 命令总览

| 命令 | 用途 | 必填参数 |
|------|------|---------|
| `check-login` | 检查登录状态，未登录时自动返回二维码 | 无 |
| `wait-login` | 等待扫码完成（阻塞，最多120秒） | 无 |
| `search-feeds` | 按关键词搜索笔记 | `--keyword` |
| `get-feed-detail` | 获取笔记详情（完整图片、发布日期） | `--feed-id`, `--xsec-token` |

AI 表格操作通过调用内置的 `dws aitable` 技能实现（`doc create` / `doc search` / `sheet list` / `sheet create` / `field list` / `field create` / `record list` / `record create` / `record update`）

## 意图判断决策树

用户说"搜索/追踪/找/分析 XX 热点/爆款/选品":
- 默认电商选品 → `search-feeds --keyword XX --sort-by 最多点赞`
- 用户说"最新" → `--sort-by 最新`
- 用户说"只看看/不用保存" → 搜索后展示，跳过写入

用户说"保存/整理/记录/分析数据"（即使没提表格）:
- 默认写入 AI 表格 → 进入表格定位流程
- 用户若明确指出表格名，查找对应的 AI 表格，未查找则自动命名并创建
- 无表格名 → 自动命名"小红书热点\_关键词\_日期" → 创建 AI 表格
- 一次会话最多只创建一个 AI 表格

关键区分: `search-feeds` 返回摘要（封面图+点赞数）；`get-feed-detail` 返回完整图片列表+发布日期，仅在需要完整图片时调用

## 核心工作流

```bash

# Step 0: 前置依赖检查

# Step 1: 调用 use_browser 技能启动浏览器，访问小红书首页：https://www.xiaohongshu.com/explore/，获取 CDP WebSocket 地址
# 返回示例: ws://127.0.0.1:9222/devtools/browser/xxx
# 从地址解析 HOST 和 PORT，例如 HOST=127.0.0.1, PORT=9222
# 后续所有命令均需传入 --host HOST --port PORT

# Step 2: 检查登录状态
python3 scripts/cli.py --host HOST --port PORT check-login
# 返回 {"logged_in": true} → 继续
# 返回 {"logged_in": false, "qrcode_image_url": "..."} → 展示二维码，然后：
python3 scripts/cli.py --host HOST --port PORT wait-login
# 返回 {"logged_in": true} → 继续

# Step 3: 搜索笔记（电商选品默认最多点赞），返回的笔记列表确保要包含封面图信息
python3 scripts/cli.py --host HOST --port PORT search-feeds --keyword "防晒霜" --sort-by 最多点赞
# 返回 feeds[]{id, xsecToken, displayTitle, cover, interactInfo{likedCount}, user{nickname}}
# 提取: feeds[].id → feed_id, feeds[].xsecToken → xsec_token

# Step 4: 调用 dws 的 aitable 技能将数据直接保存至钉钉 AI 表格，不需要额外创建其他中间文件进行展示。

# Step 4.1: 用户若明确指出表格名，查找对应的 AI 表格，未查找到则自动命名并创建。如果未指定表格名，也自动创建。自动命名规则: "小红书热点_关键词_日期"

# Step 4.2: 如果是新建表格，创建完成后，需要再创建初始字段，分别是：笔记标题、封面图、发布者、点赞数、收藏数、评论数、笔记链接。其中笔记标题是主字段。
## 重要：笔记链接拼接规则，一定要带 xsec_token。https://www.xiaohongshu.com/explore/{id}?xsec_token={xsecToken}


# Step 4.3: 将记录批量插入进表格中，记录的最终结构为：
| fieldName | type |
|-----------|------|
| 笔记标题 | text |
| 封面图 | link |
| 发布者 | text |
| 点赞数 | number |
| 收藏数 | number |
| 评论数 | number |
| 笔记链接 | link |

# Step 5: 写入成功后必须返回表格链接，向用户展示: "已保存 XX 条，查看: https://alidocs.dingtalk.com/i/nodes/{baseId}"
```

## 上下文传递规则

| 操作 | 从返回中提取 | 用于 |
|------|------------|------|
| `search-feeds` | `feeds[].id` | `get-feed-detail` 的 `--feed-id` |
| `search-feeds` | `feeds[].xsecToken` | `get-feed-detail` 的 `--xsec-token` |
| `search-feeds` | `feeds[].cover` | 封面图写入表格 |
| `search-feeds` | `feeds[].interactInfo.likedCount` | 点赞数写入表格 |
| `search-feeds` | `feeds[].user.nickname` | 发布者写入表格 |
| `get-feed-detail` | `note.imageList[].urlDefault` | 完整图片列表（按需） |


## 错误处理

1. `check-login` 返回未登录 → 展示二维码，调用 `wait-login`，不要继续搜索
2. `search-feeds` 返回空列表 → 提示换关键词或检查登录，不写入表格
3. `search_bases` 找不到表格 → 询问用户是否新建，不自动新建
4. `create_records` 失败 → 展示错误，不猜测字段名重试

## 详细参考 (按需读取)

- [references/api-reference.md](./references/api-reference.md) — 完整字段路径 + 返回值结构
- [references/error-codes.md](./references/error-codes.md) — 错误码 + 调试流程
