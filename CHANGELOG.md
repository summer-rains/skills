# 更新日志

`58pic-open/skills` 变更记录。细粒度改动以 Git 历史为准。

## 2026-09-02

### 新增

- **`58pic-library`** skill —— 查看「我的数据」：收藏夹列表、收藏夹内素材、下载记录、
  AI 创作记录、参考图上传记录（对应 CLI 的 `58pic favorites` / `my-downloads` /
  `my-generations` / `uploads`，`@58pic/cli ≥ 0.2.0`）
  - `SKILL.md` + `references/troubleshooting.md`
  - 同步进 `plugins/58pic-ai/skills/58pic-library/`
- `README.md`：可用 skill 表 + `npx skills add` 列表补 `58pic-library`
- `plugins/58pic-ai/.codex-plugin/plugin.json`：version `0.2.0 → 0.3.0`，
  description / longDescription / capabilities / defaultPrompt 补 library

### 说明

- Codex 插件包 `plugins/58pic-ai/` 仍只打包 5 个 skill（缺 `58pic-publish` / `58pic-workflow`，
  为历史 drift，本次不回填）
