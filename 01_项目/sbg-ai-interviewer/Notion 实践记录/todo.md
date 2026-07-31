---
title: "todo"
publish: false
tags: ["项目实践"]
---
# todo

## **你现在可以做的事（RAG 未实施、DB 设计未定）**

你是 App 实装担当，RAG 要等 DB 设计（フラット構成）定稿才能正式做，但下面这些可以**提前做、且不依赖最终 schema**，还能帮你补 RAG 基础。

**1. 用「最小 schema」自己跑通一整条 RAG（最推荐）**

- **目的**：不依赖正式 DB 设计，先亲手走一遍：建索引 → 灌几条假数据 → 在 UI 里提问并看到「参考信息」。
- **做法**：
- 建一个**临时索引**（名字带 -dev / -sandbox），只放 3 个字段：id, content, vector（维度与现有 embedding 一致，例如 3072）。
- 写一个**最小的 add_documents**：3～5 条假文档（几段短文），生成 content 和 vector，调用现有的 vector_db.add_documents()。
- 本地把 rag.enabled 设为 true，用该临时 index 的环境变量跑 app，在 UI 里提问，确认能检索到并出现 References。
- **收获**：理解「索引字段 ↔ 灌数 document 必须一致」「query → embedding → 检索 → 拼进 prompt」整条链；正式 schema 下来后只是把字段从 3 个扩成正式版。

**2. 和 DB 设计侧约定「RAG 需要的交付物」**

- **目的**：DB 设计一定稿，你就能立刻开工，不再因为“缺字段定义”而空等。
- **做法**：和负责 DB 设计的人确认并写进文档（邮件或 Confluence）：
- **索引**：索引名、各字段名与类型、vector 字段名与维度、要不要 filter/facet（如 data_type）、semantic 配置名。
- **文档**：灌数时每条 document 的必填字段（如 id, content, vector, data_type, metadata_json 等）以及数据来源（表/文件、谁导出）。
- **Pydantic**：是否用 Pydantic 约束 document 或 AI 输出；若用，由谁提供/维护 schema。
- 你可以先起草一份「RAG 用仕様チェックリスト」发给他们填，避免漏项。

**3. 先写脚本骨架，用占位符字段**

- **目的**：把 create_or_update_index.py 和 add_documents.py 的「壳」搭好，正式 schema 来了只改字段定义和数据来源。
- **做法**：
- **create_or_update_index.py**：参考 pjt-interviewer-tb-chatbot，先用最小 schema（id, content, vector）写一版，在注释里标出「正式版で追加するフィールド」。
- **add_documents.py**：先做「从本地 JSON/Excel 读几条假数据 → 转成 document → add_documents」的迷你版，数据源和正式字段用 TODO/配置预留。
- 这样能提前熟悉 Azure Search 的 API，正式实现时少查文档。

**4. 环境与权限**

- 在 **.env.example** 里列全 RAG 相关变量（AZURE_SEARCH_*、AZURE_OPENAI_EMBEDDING_DEPLOYMENT 等），并注明哪些是建索引/灌数用、哪些是 app 运行时用。*
- 进度里提到「Drive・Azure Portal にアクセスできない」→ 尽早申请 **Azure Portal（以及 Azure Search / OpenAI 资源）权限**，否则无法做建索引和本地 RAG 联调；可以和「権限管理」的依頼一起推进。

**5. 可选：在参考项目里完整走一遍**

- 若 pjt-interviewer-tb-chatbot 已有可用索引和数据，可以在那边跑一次 create_or_update_index.py、add_documents.py、chainlit run main.py，从用户视角完整体验一次 RAG。这样即使 pjt-sbg 的 DB 还没定，你也已经见过「标准形态」的 RAG，之后把同样模式迁回 pjt-sbg 会更快。

---

**总结**

：

- **优先**：用「最小 schema（id / content / vector）」在本地跑通一条 RAG，并和 DB 设计侧约定好「RAG 用索引・文档・Pydantic 的交付物」。
- **并行**：写 create_or_update_index / add_documents 的脚本骨架、补全 .env.example、推进 Azure/Drive 权限。
- 详细步骤已写在 **pjt-sbg/docs/RAG_流程与下一步.md** 的「四、DB 设计未定 + RAG 零基础时，现阶段可以提前做的事」里。
