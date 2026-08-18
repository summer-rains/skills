# 分类树结构

`58pic categories -f json` 返回完整三级分类树，一次性拿全量，不分页。只包含 id/名称字段，没有封面图/载体尺寸这类展示信息。

```json
{
  "data": [
    {
      "did": 1,
      "dname": "一级分类名",
      "child": [
        {
          "kid": 11,
          "kname": "二级分类名",
          "child": [
            { "bid": 111, "bname": "三级分类名" }
          ]
        }
      ]
    }
  ]
}
```

## 提取三级选项

```bash
58pic categories -f json | jq '[
  .data[] as $d
  | $d.child[] as $k
  | $k.child[] as $b
  | {did: $d.did, dname: $d.dname, kid: $k.kid, kname: $k.kname, bid: $b.bid, bname: $b.bname}
]'
```

## 使用建议

1. 优先按用户描述的作品类型（海报/UI/PPT/摄影图等）在 `dname`/`kname`/`bname` 中做关键词匹配，减少让用户手动翻分类树。
2. 找到多个候选时，把 `dname > kname > bname` 拼成可读路径列给用户选，而不是甩原始 id。
3. `bid` 是最终提交需要的三级分类 id。

## 和 `suggest-meta` 的关系

- `categories()` 是**全站可上传的完整分类树**，不限定范围，用户想改分类、或者 `suggest-meta` 没识别出来时，从这里选。
- `suggest-meta` 的 AI 分类建议**固定收窄在 `did=60`（AI模板）这一个一级分类下**——AI 生成内容统一归到这个一级分类，`suggest-meta` 不会也不需要给出其他 `did`。
- 如果用户想把 AI 作品发到 `did=60` 之外的分类，跳过 `suggest-meta` 的分类建议，直接从 `categories()` 全树里选。
