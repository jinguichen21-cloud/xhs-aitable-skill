# 错误码 + 调试流程

## 小红书 MCP 错误

| 错误信息 | 含义 | 修复建议 |
|---------|------|---------|
| 未登录 / isLoggedIn=false | Cookie 失效或未登录 | 调用 get_login_qrcode 重新扫码登录 |
| 搜索Feeds失败: 缺少关键词参数 | keyword 为空 | 确认用户提供了关键词 |
| 搜索Feeds失败: ... | 网络或服务异常 | 检查 MCP Server 是否正常运行 |
| get_feed_detail 失败 | feed_id 或 xsec_token 错误 | 确认从 search_feeds 返回中提取，不要手动拼写 |

## 钉钉 AI 表格错误

| 错误信息 | 含义 | 修复建议 |
|---------|------|---------|
| base not found | baseId 不存在或无权限 | 重新 search_bases 确认 baseId |
| table not found | tableId 不存在 | 重新 get_base 确认 tableId |
| field not found | fieldId 不存在 | 重新 get_fields 确认 fieldId，不要猜测 |
| records 格式错误 | cells 包装缺失 | 检查格式：records=[{cells:{fieldId:value}}] |
| 超出批量上限 | 单次 create_records 超过 100 条 | 分批写入，每批 ≤ 30 条 |

## 调试流程

1. search_feeds 返回空 → 换关键词 / 检查登录状态
2. create_records 写入后字段为空 → 检查 fieldId 是否与 get_fields 返回一致
3. 表格找不到 → search_bases 用模糊关键词重试，或让用户确认表格名
4. 整体流程卡住 → 从 check_login_status 重新开始
