# Troubleshooting

## Authentication

| Symptom | Fix |
|---|---|
| `未配置认证信息` / HTTP 401 | `58pic config init --api-key sk_…` or `58pic auth login`, then retry |
| `OAuth token 已过期，请重新执行 58pic auth login` | `58pic auth login` |

Re-check: `58pic auth status -f json | jq '{loggedIn, authMethod}'`

---

## Empty results

An empty `data.list` is usually genuine, not an error. Check `code` first:

```bash
echo "$RESULT" | jq '.body | {code, msg}'
```

`code == 200` with an empty list = the account really has nothing here.

- **`my-downloads` empty under an enterprise API Key** — enterprise download records only cover downloads made through that enterprise. A brand-new or unused enterprise account shows `total: 0`.
- **`favorites <id>` returns `code: 502` / `主站返回非 JSON`** — the folder id is wrong, deleted, or belongs to another account. Re-run `58pic favorites` to get valid ids.
- **`favorites <id>` fields mostly empty** (`keyword`, `author`, `format`, `origin_url`) — expected for folders that hold AI-generated works; the upstream record is sparse. `pic_id` / `title` / `preview_url` are still populated.

---

## Date-range filters

`--start` and `--end` must be provided **together** and formatted `YYYY-MM-DD`:

```bash
58pic my-downloads --start 2026-08-01 --end 2026-08-31 -f json     # OK
58pic my-downloads --start 2026-08-01 -f json                       # rejected: pair required
```

`my-generations` additionally caps the span at **90 days** — split longer ranges into multiple calls.

---

## card_id (my-generations)

Only these are accepted (others → 400):

| id | capability |
|---|---|
| `0` | all |
| `292` | AI海报 |
| `293` | AI绘图 |
| `294` | AI艺术字 |
| `295` | AI人像抠图 |
| `297` | 智能抠图 |
| `304` | 消除笔 |
| `305` | 画质超清 |
| `307` | AI无损放大 |

---

## Pagination

- `favorites` / `my-downloads` / `my-generations` / `uploads` — return `total` (or `total_page`); page with `--page`.
- `favorites <favor_id>` — **no total**; keep paging with `--page` until `has_more` is `false`.

---

## API response codes

| `code` | Meaning |
|---|---|
| `200` | Success |
| `400` | Bad parameter — check `.msg` |
| `403` / `300` | Enterprise identity issue (need to switch enterprise account) — `.msg` has detail |
| `502` | Upstream (58pic_web) hiccup — retry after a few seconds |

HTTP: `401` re-authenticate · `429` back off · `5xx` retry.
