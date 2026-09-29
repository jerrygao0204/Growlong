# Growlong

 Growlong 是一套模組化智能問答系統，核心架構聚焦於**分層模組化設計 (Layered Modular Architecture)**，透過**三級工具工廠 (Three-Tier Tool Factory)**、**離線/線上解耦的資料管線 (Decoupled Data Pipeline)** 以及**雙重安全隔離機制 (Dual-Layer Security Isolation)**，實現高性能問答與跨會話用戶認知演進。

---

## 一、 核心架構優勢

1. **基礎能力高度共享**：基於 `ModelFactory` 與 `ToolFactory` 統一管理推理算力與三級（Domain → Package → Tool）工具生態，支援多端入口（後台、問答、MCP 服務）無縫複用。
2. **建庫與問答完全解耦**：採用「離線文檔解析/分塊/校驗/入庫」與「線上檢索/生成」雙軌架構，資料源格式（PDF/Web/DB）解耦，支援獨立擴容與迭代。
3. **長文檔自研解析管線**：內置**滑動窗口分片機制 (Sliding Window Chunking)** 與**跨頁表格合併演算法 (Cross-Page Table Merging)**，突破長文本與複雜表格的上下文限制。
4. **四層成長型記憶系統 (Layered Growth Memory)**：摒棄傳統流水帳式 Context 堆疊，將用戶認知拆解為**身份畫像 (Identity Profile)**、**穩定語境 (Stable Context)**、**動態語境 (Dynamic Context)** 與 **成長軌跡 (Growth Context)**，實現跨會話認知積累。
5. **雙重安全防護體系 (Dual-Layer Security)**：
   - **代碼側**：採用 AST 靜態審查 (AST Static Analysis) 與子進程沙箱 (Subprocess Sandbox) 機制。
   - **文本側**：結合正則脫敏 (Regex Anonymization) 與 LLM 語義二次審查 (Semantic Moderation)。

---

## 二、 系統整體架構

```mermaid
flowchart TD
    subgraph 入口層 (Entry Layer)
        AA[App Admin<br/>知識庫建設後台]
        QA[QA Admin<br/>問答系統後台]
        MCP[MCP Server<br/>MCP 工具服務]
    end

    subgraph 基礎設施層 (Infrastructure)
        TF[ToolFactory<br/>三級工具工廠]
        MF[ModelFactory<br/>模型與算力工廠]
    end

    AA --> MF
    QA --> MF
    QA --> TF
    MCP --> TF
    QA --> RAG[QAChain + ReActAgent]
    RAG --> TF
    RAG --> MF

    subgraph 資料與狀態存儲 (Data & State)
        V[(Vector DB / Milvus)]
        GM[分層成長記憶<br/>Layered Growth Memory]
        SM[會話短期記憶<br/>Short-term Memory]
    end

    TF --> V
    QA --> GM
    QA --> SM

    P[原始資料源<br/>Data Sources] -.離線管線.-> D[資料處理管線<br/>Data Pipeline]
    D -.寫入.-> V
```

---

## 三、 專案目錄結構與模組對照

```bash
agent_jerry_gao/
├── agent/                 # ReAct Agent、AST 安全檢查、子進程沙箱與傳輸層
├── api/                   # 可觀測性與基礎指標統計
├── config/                # 全局 YAML/Python 配置（Prompt/Rules/Tools）
├── data_prep/             # 離線文檔解析（滑動窗口/表格識別）與結構化分塊
├── eval/                  # 檢索與生成端到端評估套件 (Evaluation Suite)
├── factory/               # 模型工廠、三級工具工廠與內置工具庫
├── generator/             # LLM 統一客戶端與 QAChain 邏輯
├── ingest/                # 資料校驗 (Validator) 與向量庫寫入 (Milvus Ingestion)
├── memory/                # 會話級短期記憶、實體上下文與反饋存儲
├── memory_growth/         # 四層成長記憶抽取、分層映射與動態 Prompt 渲染
├── search/                # 向量/關鍵字混合檢索與重排序 (Hybrid Search & Reranker)
├── tests/                 # pytest 單元測試與整合測試套件
└── utils/                 # 通用併發與原子 I/O 工具
```

| 核心層級 | 對應模組 | 關鍵職責 |
|---|---|---|
| **應用與入口層** | `app_admin.py` / `qa_admin.py` / `mcp_server.py` | 互動入口、控制台與外部 MCP 工具服務暴露 |
| **資料處理與入庫** | `data_prep/` / `ingest/` | 跨頁表格合併、滑動窗口 Parsing、資料校驗與 Milvus 寫入 |
| **檢索與生成鏈路** | `search/` / `generator/` | 混合檢索 (Hybrid Retrieval)、Cross-Encoder 重排與 LLM 客戶端封裝 |
| **模型與工具底座** | `factory/` | 模型算力調度、三級工具註冊、角色可見性控制 (Role RBAC) |
| **智能體與安全防護** | `agent/` | ReAct 調度循環、AST 代碼審查、沙箱隔離與合規過濾 |
| **記憶與上下文** | `memory/` / `memory_growth/` | 短期 Context 維護、原子化 I/O 落盤與四層成長記憶渲染 |

---

## 四、 快速開始與使用指南

### 1. 環境準備與服務啟動

```bash
# 啟動知識庫建設後台
python app_admin.py

# 啟動問答系統後台
python qa_admin.py

# 啟動 MCP 工具服務
python mcp_server.py
```

### 2. 資料處理與入庫管線

```bash
# 1. 原始 PDF 解析為 Markdown（含跨頁表格合併與滑動窗口處理）
python data_prep/pdf_to_markdown.py

# 2. Markdown 結構化切分 JSON Chunk
python data_prep/markdown_to_json.py
```

### 3. 自動化測試與評估 (Testing & Evaluation)

```bash
# 執行單元測試與整合測試
pytest

# 運行端到端評估（檢索 Hit Rate / MRR / 響應延遲 / Answer Relevance）
python eval/run_all_eval.py
```

### 4. 跨會話成長記憶構建

```bash
# 從會話歷史抽取事實並構建四層結構化 Prompt
python memory_growth/1_extractor.py
python memory_growth/2_layer_mapper.py
python memory_growth/3_context_builder.py
```

---

## 五、 質量防護與技術演進說明

- **併發與原子化保障**：文件級操作引入 `atomic_io.py`（進程鎖 + 原子替換），防止多線程/多進程寫入導致的資料損壞。
- **併發擴展機制**：`FeedbackStore` 當前使用 `threading.Lock` 保護單進程線程安全；多 Worker 部署時可無縫擴展為 `filelock.FileLock`。
- **環境安全**：內置配置包含占位鑒權資訊，生產部署前請透過環境變數或加密配置文件完成替換。
