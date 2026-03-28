# API Reference

## 小红书 CLI 命令（`python scripts/cli.py`）

### check-login
无参数。返回：
```json
{
  "logged_in": false,
  "qrcode_path": "/tmp/xhs/login_qrcode.png",
  "qrcode_image_url": "https://api.qrserver.com/...",
  "qr_login_url": "https://www.xiaohongshu.com/mobile/login?..."
}
```
已登录时：`{ "logged_in": true }`

### search-feeds
```bash
python scripts/cli.py search-feeds --keyword "防晒霜" --sort-by 最多点赞
```
返回结构（关键字段）：
```json
{
  "feeds": [
    {
      "id": "笔记ID",
      "xsecToken": "安全令牌（后续操作必须携带）",
      "displayTitle": "笔记标题",
      "cover": "封面图URL",
      "user": {
        "userId": "用户ID",
        "nickname": "发布者昵称"
      },
      "interactInfo": {
        "likedCount": "点赞数（字符串）",
        "collectedCount": "收藏数",
        "commentCount": "评论数"
      }
    }
  ],
  "count": 20
}
```

### get-feed-detail
```bash
python scripts/cli.py get-feed-detail --feed-id <id> --xsec-token <xsecToken>
```
返回关键字段（嵌套在 `note` 下）：
```json
{
  "note": {
    "noteId": "笔记ID",
    "title": "笔记标题",
    "createTime": "发布时间戳（毫秒）",
    "imageList": [
      { "urlDefault": "图片URL" }
    ],
    "interactInfo": {
      "likedCount": "点赞数",
      "collectedCount": "收藏数",
      "commentCount": "评论数"
    }
  }
}
```

## 重要：笔记链接拼接规则，一定要带 xsec_token

```
https://www.xiaohongshu.com/explore/{id}?xsec_token={xsecToken}
```

## 钉钉 AI 表格 Tools（通过 dws skill 调用）

### AI 表格字段类型
| fieldName | type |
|-----------|------|
| 笔记标题 | text |
| 封面图 | link |
| 发布者 | text |
| 点赞数 | number |
| 收藏数 | number |
| 评论数 | number |
| 发布日期 | date |
| 笔记链接 | link |
