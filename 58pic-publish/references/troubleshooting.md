# Troubleshooting

## 身份相关

| 症状 | 处理 |
|---|---|
| `is_designer: false` | 不要继续发布流程。告知用户仅限设计师账号，引导到 `https://www.58pic.com/u/designerentry/` 在线申请 |
| 已申请但仍显示非设计师 | 申请审核（`sh_status`）尚未通过，提示用户等待审核结果，不要重复提交 |

## 参数相关

| 症状 | 处理 |
|---|---|
| 关键词不足 5 组 | 提交前在客户端先校验，不足则追问用户补充，不要把请求发给服务端等报错 |
| 分类 id 不存在 | 重新拉取 `58pic categories -f json`，分类树可能有更新，不要缓存旧数据长期复用 |
| `ai_id` 对应任务未生成成功 | 用 `58pic same-style-status <ai_id> -f json` 确认任务 `status=3`（成功）后再发布 |

## AI 补全（suggest-meta）相关

| 症状 | 处理 |
|---|---|
| `resolved.title`/`resolved.keyword`/`resolved.category` 为 `false` | AI 没识别出来，不要瞎编或硬塞默认值，转人工询问用户对应字段 |
| `resolved.category` 一直是 `false` | 常见原因：这是个视频任务，还没抽出封面图（`video_cover` 未生成），当前不支持对未抽封面的视频做分类/标题自动生成，需要用户手动填 |
| `suggest-meta` 响应很慢（十几秒） | 正常现象，内部在调视觉模型识图，接口本身不设超时（由内部网关兜底），跟用户说明"AI 正在分析"，不要重复调用 |
| `suggest-meta` 返回的 `title`/`keyword` 感觉不准 | 展示给用户确认/编辑后再提交，不要直接采信，AI 生成的内容用户应该有机会改 |

## 提交与状态

| 症状 | 处理 |
|---|---|
| 提交后 `data.type` 为 `failed` | 审核未通过，不要自动重试；把失败原因（若接口返回）转达用户 |
| 用户以为提交=审核通过 | 明确说明 `waiting` 只代表"已成功进入审核队列"，不是最终结果 |
| 需要查最终审核结果 | 当前接口不提供轮询，需引导用户在 58pic 官网/APP 的作品管理页查看 |

## 通用

| 症状 | 处理 |
|---|---|
| CLI 未安装 | `npm install -g @58pic/cli` |
| 未登录 / token 过期 | 转 `58pic-account` 完成登录或刷新 |
