---
title: "test"
publish: false
tags: ["项目实践"]
---
# test

```markdown
## 推荐：使用 pytest 测试

日常与 CI 建议以 pytest 为主，替代手跑脚本和纯 UI 验证：

1. **索引/检索**（灌数后确认有文档且检索命中）：
   ```bash
   cd pjt-sbg
   uv run pytest tests/test_rag_index.py -v
   ```
   需配置 Azure Search 环境变量（与灌数同一索引）；未配置时测试会 skip。

2. **Q&A 验证**（填入待定问题即可查看回答与 References，并可选期望关键词断言）：
   ```bash
   uv run pytest tests/test_rag_qa.py -v -s
   ```
   - 在 `tests/test_rag_qa.py` 的 `RAG_QA_QUESTIONS` 中维护问题列表：纯字符串则仅打印 Q&A；若为 `{"q": "问题", "expected_keywords": ["词1", "词2"]}` 则会自动断言回答或引用中包含至少一个关键词。
   - `-s` 保证控制台打印每个问题的回答与 References，便于人工复核。
   - 需配置 Azure Search 与 Azure OpenAI 环境变量；未配置时测试会 skip。

推荐流程：灌数 → 运行 `test_rag_index.py` → 运行 `test_rag_qa.py`；之后若切换数据定义或重新灌数，重复后两步即可。

```
