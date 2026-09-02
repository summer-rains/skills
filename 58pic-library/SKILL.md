---
version: 0.1.0
name: 58pic-library
description: |
  Look up your own 千图AI / 58pic data: favorite folders and their contents,
  download history, AI generation history, and reference-image upload records.
  Use when: "my 58pic favorites", "list my favorite folders", "what's in my
  favorites", "my download history", "my AI creations", "my generation history",
  "我的收藏夹", "收藏夹里有什么", "我的下载记录", "我的创作记录", "我上传过的参考图".
argument-hint: "[favorites|downloads|generations|uploads]"
allowed-tools: Bash
---

# 千图AI (58pic) — My Library

Read-only lookups of the current account's own data. All commands cost 0 credits.

## Bootstrap

```bash
58pic --version
58pic auth status -f json | jq '{loggedIn, authMethod}'
```

If unauthenticated, use the `58pic-account` skill first.

## UX Rules

1. Show a short, useful list; do not dump raw JSON.
2. Favorite folders and generation tasks: include the `id` / `task_id` so the user can drill in.
3. These are all read-only history lookups — never imply a credit cost.
4. Image URLs come back absolute (`https://…qiantucdn.com/…`). Render them as markdown images where helpful.

## Favorite folders

List the account's favorite folders (newest first):

```bash
58pic favorites -f json \
  | jq '[.body.data.list[] | {id, name, count: .pic_num, cover: .cover_img, updated: .updated_at}]'
```

Fields: `id` (pass to the next command), `name`, `pic_num`, `cover_img`, `preview_pics` (up to 4), `is_private`, `updated_at`.

Pagination: `58pic favorites --page 2 -f json`

## Contents of one favorite folder

Pass a folder `id` from the previous command:

```bash
58pic favorites 28791419 -f json \
  | jq '{count: .body.data.count, has_more: .body.data.has_more,
         items: [.body.data.list[] | {pic_id, title, preview: .preview_url, type: .media_type}]}'
```

- No total is returned — page until `has_more` is `false`.
- `media_type` is `image` or `video`.
- Some fields (`keyword`, `author`, `format`, `origin_url`) may be empty for AI-generated items — that's expected, the upstream record is sparse for those.

## Download history

Personal vs. enterprise identity is decided automatically by the API Key:

```bash
58pic my-downloads -f json \
  | jq '{identity: .body.data.identity, total: .body.data.total,
         list: [.body.data.list[] | {pic_id, title, downloaded_at, format, is_ai_download}]}'
```

Optional filters:

```bash
58pic my-downloads --start 2026-08-01 --end 2026-08-31 -f json   # date range (pair required)
58pic my-downloads --pic-id 12345 -f json                         # one asset
58pic my-downloads --page 2 --page-size 30 -f json
```

Enterprise identity adds `downloader_uid` / `downloader_name` / `ai_model_name` per row and a `summary` block.

## AI generation history

Tasks aggregated (image / video / music):

```bash
58pic my-generations -f json \
  | jq '[.body.data.list[] | {task_id, card_name, status: .status_text, created_at,
                         prompt: (.prompt[:60]), credits: .credits_used,
                         images: [.images[].origin_url]}]'
```

Filters:

```bash
58pic my-generations --card-id 293 -f json                        # 293=AI绘图, 292=AI海报, 294=AI艺术字 …
58pic my-generations --start 2026-08-01 --end 2026-08-31 -f json  # span ≤ 90 days
```

`status_text`: `成功` / `进行中` / `失败` / `已取消`. Rendered images are under `images[].origin_url` (absolute); `video_url` / `music_url` for those media types.

## Reference-image upload records

```bash
58pic uploads -f json \
  | jq '[.body.data.list[] | {file_name, extension, uploaded: .created_at, url: .preview_url}]'
```

Filter by scene: `58pic uploads --biz-scene ai_mcp_reference -f json`

To upload a new reference image, use the `58pic-generate` skill.

## References

- `references/troubleshooting.md` — auth, empty results, date-range errors
