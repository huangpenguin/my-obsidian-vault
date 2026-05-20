
**项目名称：基于多模态 RAG 的智能手册问答与自动化出题系统**

- **核心技术：** Python, FastAPI, Azure OpenAI & Ollama, Langfuse, PyMuPDF, Chroma, uv, GitHub Actions
    
- **项目描述：** 面向企业级 PDF 手册，开发了一套集“高精度图文信息提取、语义检索、多模态智能问答与多题型自动化出题”于一体的 FastAPI 核心服务。
    
- **核心职责与工程实践：**
    
    - **复杂 RAG 架构设计：** 摒弃单一的向量检索，实现了多重查询生成（Multi-query）、RRF 混合重排（RRF Rerank）以及前后 Chunk 动态扩展机制，结合日本语特化模型，大幅提升了手册特定术语的召回率。
        
    - **混合大模型与多模态解析：** 构建了支持云端（Azure GPT-4o/5.2）与本地（Ollama Gemma3/Llama4）平滑切换的 LLM 路由策略。结合 PyMuPDF 实现了对手册中复杂图表和照片的自动化文本抽取与图像描述（Image Explanation）生成。
        
    - **智能出题系统研发：** 针对教育/培训场景，设计了复杂的 Prompt 链和分类路由（Query/Type Classify），实现了 6 种题型（填空、配对、判断、选择、简答、计算）的自动化生成与一括（批量）处理。
        
    - **现代化 CI/CD 与工程规范：** 主导规范了项目的基础设施。引入极速包管理器 `uv`；通过 GitHub Actions 集成 `pytest`、`ruff` 和 `pyright` 构建自动化测试与代码审查流水线；并接入 `CodeRabbit` AI 审查与 `Langfuse` 大模型全链路监控，保障了系统的高可用性与可观测性。

**プロジェクト名：マニュアル特化型 高度RAGシステム及び練習問題自動生成APIの開発**

- **役割：** リードバックエンド / AIエンジニア
    
- **技術スタック：** FastAPI, Azure OpenAI, Ollama, Langfuse, SentenceTransformers, uv, CodeRabbit, GitHub Actions
    

#### 【概要】

企業向けPDFマニュアルからテキストや図表を高精度に抽出し、意味的検索（RAG）と6種類の形式に対応した練習問題の自動生成を提供する、FastAPIベースのモダンなWeb APIサービスのゼロイチ開発。

#### 【経験・成果】

- **高度な検索パイプライン（Advanced RAG）の構築：**
    
    - 単純なベクトル検索ではなく、「多重クエリ生成（Multi-query）」「RRF再ランキング（RRF Rerank）」「チャンク動的拡張」を組み合わせたハイブリッド検索パイプラインを実装。専門用語の多いマニュアルにおける検索精度を最大化。
        
- **マルチモーダル処理とハイブリッドLLM運用：**
    
    - PyMuPDFとGPT-4oを活用し、図表や数式（LaTeX変換）を含む複雑なPDFのテキスト抽出・画像説明の自動生成ロジックを実装。
        
    - Azure OpenAIとローカルLLM（Ollama）を用途に合わせて切り替えられる柔軟なアーキテクチャを設計し、コストと機密性の両立を実現。
        
- **AIを活用した教育用コンテンツの自動生成：**
    
    - ユーザーの意図をLLMで分類し、6種類の練習問題（穴埋め、AB群対応、択一式、計算など）を動的に生成するコアロジックを開発。
        
- **モダンなCI/CDと開発者体験（DX）の追求：**
    
    - チーム開発の品質を担保するため、GitHub Flowを導入し、GitHub Actions（`pytest`, `ruff`, `pyright`）および `CodeRabbit`（AIコードレビュー）による自動化CI/CDパイプラインを構築。
        
    - `uv` による高速なパッケージ管理や、`Langfuse` を用いたLLMのプロンプト実行ログ・トークン消費のモニタリング（可観測性）を導入し、実稼働に耐えうる堅牢なシステム基盤を整備しました。

