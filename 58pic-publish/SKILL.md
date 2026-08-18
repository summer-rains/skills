---
name: 58pic-publish
description: |
  Publish an AI-generated image or video (from 58pic-generate) as a formal
  58pic asset, pending review. Designer-identity accounts only. Use when:
  "publish this AI image", "提交作品审核", "发布这张AI图", "发布素材",
  "submit my AI creation".
argument-hint: "[ai-id] [--title <text>] [--keyword <word> ...] [--no-submit]"
allowed-tools: Bash
---

# 千图AI (58pic) — 发布AI作品为素材

> **状态：方案设计中，尚未可用。** 后端 API（`open-platform/publish`、
> `open-platform/categories`、`open-platform/suggest-meta`）已经开发完成
> 并推送，但 CLI（`@58pic/cli`）尚未提供对应的 `58pic publish` /
> `58pic categories` / `58pic suggest-meta` 命令。命令语法为设计提案，
> CLI 落地前请勿按本文档直接调用。落地后删除本提示。

把 `58pic-generate` 生成成功的 AI 图片/视频，提交为正式的千图素材，进入审核队列。**仅限已认证的设计师身份账号**可用。

## 前置条件

1. CLI 与鉴权：
   ```bash
   58pic --version
   58pic auth status -f json
   ```
2. 待发布的 AI 任务已生成成功（先用 `58pic-generate` 拿到 `ai_id`，即开放平台 `same-style-status` 返回的 `details[].id`）。

## 身份校验（第一步，必须做）

```bash
58pic auth status -f json | jq '{loggedIn, authMethod, is_designer}'
```

- `loggedIn: false` → 转 `58pic-account` 完成登录。
- `is_designer: false` → **不要继续后面的流程**。告知用户该功能仅限设计师账号，引导到在线申请入口：
  `https://www.58pic.com/u/designerentry/`

设计师身份的判定口径（供理解，不需要客户端实现）：账号在 `58pic_ogc_info` 或
`58pic_upload_user` 任一表中存在 `sh_status=3` 的记录，即视为设计师。

## 使用方式

### 1. 检查用户是否已经给了标题/关键词/分类

用户描述里如果已经给了标题、够 5 个关键词、明确的分类，跳过第 2 步，直接进第 3 步收集分类 id（见下）。只要**任一项缺失**，就走第 2 步用 AI 补全。

### 2. 缺字段时，先调 AI 补全建议

```bash
58pic suggest-meta --ai-id <ai_id> -f json
```

这一步会调用视觉模型识图，**可能要等几秒到十几秒**，跟用户说清楚"AI 正在分析图片，稍等"，不要让用户以为卡住了。

返回结构：
```json
{
  "title": "...",
  "keyword": ["...", "..."],
  "did": 60,
  "kid": ...,
  "bid": ...,
  "resolved": { "title": true, "keyword": true, "category": false }
}
```

- `resolved.xxx = false` 的字段代表 AI 没能识别出来（比如视频还没抽出封面图、模型返回的分类不在候选表里），**必须转人工**——把这几项列出来问用户，不要瞎编或者硬塞一个默认值继续往下走。
- `resolved.xxx = true` 的字段，把生成的值展示给用户确认/可编辑，不要不经确认直接拿去发布——毕竟是 AI 生成的，用户应该有机会改。
- `did` 恒为 `60`（AI 生成内容固定归到「AI模板」一级分类下），不需要，也不应该去问用户选一级分类。

### 3. 分类 id 需要用户自己选/改时，查分类树

只有 `suggest-meta` 没给出分类、或者用户想改成别的分类时才需要这一步（`categories()` 是全站可上传的完整分类树，不限定在 did=60）：

```bash
58pic categories -f json
```

结构说明见 `references/category-guide.md`。

### 4. 确认后提交

`is_submit` **默认就是直接送审**，不传这个参数就等于直接送审；只有用户明确要求"先存草稿、不要送审"时才加 `--no-submit`。**直接送审前必须和用户二次确认**——这是进入公开审核队列的动作，不可随意撤回。

```bash
# 默认：直接送审
58pic publish --ai-id <ai_id> \
  --category-did 60 --category-kid <kid> --category-bid <bid> \
  --title "标题" \
  --keyword "词1" --keyword "词2" --keyword "词3" --keyword "词4" --keyword "词5" \
  -f json

# 用户明确要求先存草稿
58pic publish --ai-id <ai_id> \
  --category-did 60 --category-kid <kid> --category-bid <bid> \
  --title "标题" \
  --keyword "词1" --keyword "词2" --keyword "词3" --keyword "词4" --keyword "词5" \
  --no-submit \
  -f json
```

收益模式（`ugc_profit_type`）由服务端根据账号身份自动判定，**不需要询问用户，也不在命令行暴露**。

### 5. 解读结果

不需要轮询。返回的 `data.type` 就是最终状态，直接回复用户：

| `data.type` | 含义 |
|---|---|
| `editing` | 草稿态，尚未送审 |
| `waiting` | 已成功进入审核队列（**这就是"发布成功"**，不代表审核已通过） |
| `passed` | 审核通过 |
| `failed` | 审核未通过 |

## References

- `references/category-guide.md` — 三级分类树结构与选择方式
- `references/troubleshooting.md` — 设计师身份、关键词数量、分类、AI 补全等常见问题
