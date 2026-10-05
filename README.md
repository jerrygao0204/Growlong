# Growlong

Growlong 是一套模組化智能問答系統，核心架構聚焦於**分層模組化設計 (Layered Modular Architecture)**，透過**三級工具工廠 (Three-Tier Tool Factory)**、**離線/線上解耦的資料管線 (Decoupled Data Pipeline)**、**雙重安全隔離機制 (Dual-Layer Security Isolation)** 以及**專用路由模型與 SFT 微調能力**，實現高性能問答與跨會話用戶認知演進。

* * *

## 一、 核心架構優勢

```
flowchart TD
    subgraph entry_layer ["入口層 (Entry Layer)"]
        AA["App Admin知識庫建設後台"]
        QA["QA Admin問答系統後台"]
        MCP["MCP ServerMCP 工具服務"]
    end

    subgraph infra_layer ["基礎設施層 (Infrastructure)"]
        TF["ToolFactory三級工具工廠"]
        MF["ModelFactory模型與算力工廠"]
    end

    AA --> MF
    QA --> MF
    QA --> TF
    MCP --> TF
    QA --> RAG["QAChain + ReActAgent"]
    RAG --> TF
    RAG --> MF

    subgraph storage_layer ["支援系統 (Support Systems)"]
        V[("(向量知識庫 / Milvus)")]
        GM["分層成長記憶身份/穩定/動態/成長"]
        SM["會話內短期記憶"]
    end

    TF --> V
    QA --> GM
    QA --> SM

    P["原始資料源Data Sources"] -.離線管線.-> D["資料處理管線Data Pipeline"]
    D -.寫入.-> V
```

* * *

## 二、 系統整體架構

flowchart TD
    subgraph entry_layer ["入口層 (Entry Layer)"]
        AA["App Admin<br/>知識庫建設後台"]
        QA["QA Admin<br/>問答系統後台"]
        MCP["MCP Server<br/>MCP 工具服務"]
    end

    subgraph infra_layer ["基礎設施層 (Infrastructure)"]
        TF["ToolFactory<br/>三級工具工廠"]
        MF["ModelFactory<br/>模型與算力工廠"]
    end

    AA --> MF
    QA --> MF
    QA --> TF
    MCP --> TF
    QA --> RAG["QAChain + ReActAgent"]
    RAG --> TF
    RAG --> MF

    subgraph storage_layer ["支援系統 (Support Systems)"]
        V[("(向量知識庫 / Milvus)")]
        GM["分層成長記憶身份/穩定/動態/成長"]
        SM["會話內短期記憶"]
    end

    TF --> V
    QA --> GM
    QA --> SM

    P["原始資料源<br/>Data Sources"] -.離線管線.-> D["資料處理管線<br/>Data Pipeline"]
    D -.寫入.-> V

* * *

## 三、 專案目錄結構與模組對照

```
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
├── SFT/                   # 模型蒸餾/微調、檢查點盲測與量化測試
├── search/                # 向量/關鍵字混合檢索與重排序 (Hybrid Search & Reranker)
├── tests/                 # pytest 單元測試與整合測試套件
└── utils/                 # 通用併發與原子 I/O 工具
```

| 核心層級 | 對應模組 | 關鍵職責 |
| --- | --- | --- |
| **應用與入口層** | `app_admin.py` / `qa_admin.py` / `mcp_server.py` | 互動入口、控制台與外部 MCP 工具服務暴露 |
| **資料處理與入庫** | `data_prep/` / `ingest/` | 跨頁表格合併、滑動窗口 Parsing、資料校驗與 Milvus 寫入 |
| **檢索與生成鏈路** | `search/` / `generator/` | 混合檢索 (Hybrid Retrieval)、Cross-Encoder 重排與 LLM 客戶端封裝 |
| **模型與工具底座** | `factory/` | 模型算力調度、三級工具註冊、角色可見性控制 (Role RBAC) |
| **智能體與安全防護** | `agent/` | ReAct 調度循環、AST 代碼審查、沙箱隔離與合規過濾 |
| **模型微調與蒸餾** | `SFT/` | 訓練數據構建、LoRA/QLoRA 訓練、檢查點盲測與量化驗證 |
| **記憶與上下文** | `memory/` / `memory_growth/` | 短期 Context 維護、實體上下文、原子落盤與三層動態記憶演進 |

* * *

## 四、 快速開始與使用指南

### 1\. 環境準備與服務啟動

```
# 啟動知識庫建設後台
python app_admin.py

# 啟動問答系統後台
python qa_admin.py

# 啟動 MCP 工具服務
python mcp_server.py
```

### 2\. 資料處理與入庫管線

```
# 1. 原始 PDF 解析為 Markdown（含跨頁表格合併與滑動窗口處理）
python data_prep/pdf_to_markdown.py

# 2. Markdown 結構化切分 JSON Chunk
python data_prep/markdown_to_json.py
```

### 3\. 工具路由專用模型啟用 (Optional / SFT 增強)

若您在 `SFT/` 下訓練了專用於工具路由的模型（如 `qwen3-8b-qlra-awq-4bit`），可將其與主回答模型解耦：

```
# 設定環境變數指定路由模型（僅負責 Domain / Package 選擇，溫度固定 0.0）
export TOOL_ROUTER_MODEL=qwen3-8b-qlra-awq-4bit
python qa_admin.py
```

> 路由命中審計記錄將自動輸出至 `data/admin/tool_router_audit.jsonl` 以供行為審查。

### 4\. 跨會話成長記憶構建

```
# 推薦：一鍵運行完整流水線（增量演進）
python memory_growth/app.py --user admin

# 全量重洗：重新抽取全部對話並從零構建語境
python memory_growth/app.py --user admin --full-run
```

### 5\. 自動化測試與評估 (Testing & Evaluation)

```
# 執行單元測試與整合測試
pytest

# 運行端到端評估（檢索 Hit Rate / MRR / 響應延遲 / Answer Relevance）
python eval/run_all_eval.py
```

* * *

## 五、 質量防護與技術演進說明

+   **併發與原子化保障**：文件級操作引入 `atomic_io.py`（進程鎖 + 原子替換），防止多線程/多進程寫入導致的資料損壞。
+   **併發擴展機制**：`FeedbackStore` 當前使用 `threading.Lock` 保護單進程線程安全；多 Worker 部署時可無縫擴展為 `filelock.FileLock`。
+   **環境安全**：內置配置包含占位鑒權資訊，生產部署前請透過環境變數或加密配置文件完成替換。
