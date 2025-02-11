# **海事法規 FAQ 系統專案技術報告**

---

## 📌 **1. 專案構想**

本專案旨在設計並實作一套結合高效知識檢索與語意生成能力的 **AI 驅動 FAQ 系統**，聚焦於解析與回應 **海事法規（UR E26 與 UR E27）** 的複雜查詢。隨著全球船舶營運合規標準日益嚴格，法規查詢需求逐漸從靜態文件檢索轉向即時、動態且具推理能力的智慧系統。本專案整合 **AI 代理（AI Agents）**、**檢索增強生成（RAG）架構**，並搭載 **LLaMA 3.2 3B 模型**，旨在建立一個具備高效語意理解與知識推理能力的智能平台。

**研究目標：**
- **自動化法規查詢處理：** 降低人工查詢負擔，提升法規檢索效率。
- **強化法規依據透明性：** 確保回答具備可追溯的法規依據，提升可信度。
- **知識推理能力擴展：** 建構可延伸至其他法規領域的自動化推理架構。

---

## 🎯 **2. 選題背景**

選擇 **UR E26 與 UR E27** 作為核心研究法規，源於其在國際海事組織（IMO）技術規範中的關鍵地位：

- **UR E26：** 涵蓋船舶電氣系統安裝的技術標準，聚焦於系統可靠性、電氣隔離、短路保護及安全防護機制等設計要求。
- **UR E27：** 著重於控制系統的冗餘設計、故障容忍機制及系統恢復能力，確保自動化控制系統於異常狀態下仍能維持運作。

此選題回應產業界對於快速且準確解讀法規的迫切需求，特別針對船級社、船東、造船廠等在進行技術審查、風險評估及合規驗證時，能即時獲取可靠的法規依據。

---

## ⚙️ **3. 執行規劃**

### **技術架構與工具鏈**

- **Docker：** 提供一致的容器化運行環境，確保系統跨平台部署的穩定性與可擴展性。透過容器化技術，輕鬆實現服務隔離、版本控制與快速擴展。
- **n8n：** 開源自動化工作流程平台，負責整合 API、管理資料流與觸發自動化任務。其低代碼架構支援快速開發並提升維護效率。
- **Ollama（LLaMA 3.2 3B）：** 高效能語言模型，具備深度語意理解與自然語言生成能力，用於推理法規條文並產生具邏輯性的回答。
- **ChromaDB：** 高效向量資料庫，支援語意檢索，適合處理非結構化法規文本與多語言查詢需求。
- **PDF 解析工具（如 `PyMuPDF`）：** 負責將 PDF 格式的法規文件轉換為結構化資料，以利後續語意嵌入與檢索。

### **初步系統架構與流程**

```plaintext
[使用者提問（API/Line Bot）]
    ↓
[n8n Webhook（接收與觸發工作流程）]
    ↓
[ChromaDB（語意向量檢索）]
    ↓
[LLaMA 3.2 3B（初步語意推理與回答生成）]
    ↓
[自動化回覆（附引用法規依據）]
```

### **核心流程步驟：**
1. **PDF 解析與結構化處理：** 將 UR E26/E27 的 PDF 文檔解析為 JSON 格式，利於後續資料處理。
2. **語意嵌入與儲存：** 使用 Sentence Transformers 將法規文本轉換為向量，存入 ChromaDB 進行語意檢索。
3. **自動化工作流程管理：** 透過 n8n 串接 API，管理資料流並驅動檢索與模型推理流程。
4. **語意推理與生成：** 依據檢索結果，使用 LLaMA 進行語意推理並產生具備法規依據的回覆。

---

## ⚠️ **4. 執行方案檢討與挑戰**

1. **PDF 解析限制：** 現有工具對於複雜表格、多層次標題與跨頁段落處理不足，影響資料完整性。
2. **語意檢索精度不足：** 對於法律專業術語、多義詞或隱含語意的處理仍有提升空間，可能導致檢索結果不夠精確。
3. **模型幻覺現象（Hallucination）：** LLaMA 在面對資料不足或不確定情境下，可能產生不具實證依據的推測性回答。
4. **效能瓶頸：** 系統在資源受限環境（如 MacBook Air M1 8GB RAM）下，執行大型模型推理可能出現延遲或記憶體溢位。

---

## 🔧 **5. 優化方案與技術補強**

### **改進措施：**

1. **進階 PDF 處理：** 採用 `pdfplumber` 或自訂規則進行結構化解析，解決表格與跨頁段落解析不完整的問題。
2. **混合檢索策略：** 結合傳統關鍵字檢索（Elasticsearch）與語意檢索（ChromaDB）進行雙層檢索，提升檢索結果的準確性與覆蓋率。
3. **反思式工作流（Reflection Workflow）：** 導入模型自我檢核機制，提升回答的邏輯一致性與法規依據的準確性。
4. **模型優化：** 部署量化模型（如 LLaMA 3B-int8），降低記憶體消耗並提升推理效率，適應低資源環境。
5. **容器化最佳化：** 透過 Docker Compose 管理多個服務，提升系統的可維護性與擴展性，並支援負載平衡與高可用性部署。

---

## 🚀 **6. 第二次執行規劃與程式碼實作**

### **優化後系統架構：**

```plaintext
[User Query (API/Line Bot)]
    ↓
[n8n Webhook (Trigger Workflow)]
    ↓
[Hybrid Search System]
    ├── [Keyword Search (Elasticsearch)]
    └── [Semantic Search (ChromaDB)]
        ↓
[LLaMA 3.2 3B (Initial Answer Generation)]
    ↓
[Reflection Workflow (Self-Assessment)]
    ↓
[Answer Refinement & User Response]
```

### **核心程式碼範例：**

#### **1. PDF 解析與資料結構化**
```python
import pdfplumber
import json

pdf_path = 'UR_E26_E27.pdf'
extracted_data = []

with pdfplumber.open(pdf_path) as pdf:
    for page in pdf.pages:
        text = page.extract_text()
        if text:
            paragraphs = text.split('\n')
            for para in paragraphs:
                if len(para.strip()) > 30:
                    extracted_data.append({"content": para.strip()})

with open('parsed_data.json', 'w') as f:
    json.dump(extracted_data, f, ensure_ascii=False, indent=4)
```

#### **2. 嵌入資料至 ChromaDB**
```python
import chromadb
from sentence_transformers import SentenceTransformer

client = chromadb.Client()
collection = client.create_collection("maritime_laws")

model = SentenceTransformer('all-MiniLM-L6-v2')

with open('parsed_data.json') as file:
    data = json.load(file)

for item in data:
    embedding = model.encode(item['content']).tolist()
    collection.add(
        documents=[item['content']],
        metadatas=[{"source": "UR E26/E27"}],
        embeddings=[embedding]
    )
```

#### **3. n8n 自動化工作流程設計**
- 設定 Webhook 節點接收來自 API/Line Bot 的提問。
- 呼叫 ChromaDB API 進行檢索並取得相關法規內容。
- 使用 Ollama 調用 LLaMA 3.2 3B 進行語意推理與回答生成。
- 加入自我反思流程，進行答案驗證與修正，確保回覆品質。

---

## ✅ **結論**

本專案結合語意檢索、深度語言模型與自動化工作流程，構建出一個具備即時查詢、法規推理與自我反思能力的智慧法規 FAQ 系統。

- **高精度回答：** 雙層檢索與模型反思機制確保回覆具備高準確性與法規依據。
- **效能優化：** 模型量化與容器化部署提升系統穩定性與執行效率。
- **擴展性強：** 系統架構具備良好的模組化設計，易於擴展至其他法規領域。
- **自動化維護：** 結合 n8n 自動化與 Docker 環境，簡化系統部署與維護流程。

此系統已針對 **MacBook Air M1（8GB RAM）** 進行優化，確保在資源有限的硬體環境下仍可穩定運行，並具備未來雲端部署與大規模擴展的潛力。

![output-2](https://github.com/user-attachments/assets/b18b92b6-79c1-4763-8d14-a59200ed2075)
