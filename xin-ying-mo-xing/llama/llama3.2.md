---
description: 'Llama 3.2: Revolutionizing edge AI and vision with open, customizable models'
---

# Llama3.2

## 摘要

Meta 推出的新一代大型語言模型 Llama 3.2，以及全新的 Llama Stack 平台。Llama 3.2 包含支援圖像理解的多模態模型、輕量級的移動端模型，並強調模型的開放性、可定制性和成本效益。

## Llama 3.2 系列模型

### 輕量模型（1B, 3B）

* 支援 128K tokens 的上下文長。
* 提供預先訓練和指令調整版本，具有多語言文字生成和工具呼叫功能。

### 視覺模型（11B, 90B）

* 支援圖像推理用例，例如文件級理解（包括圖表）、圖像描述和視覺定位任務，例如可以詢問企業在上一年中哪個月份的銷售額最好，然後 Llama 3.2 可以根據可用的圖表進行推理并快速提供答案。
* 可作為相應文字模型的直接替代品，在圖像理解任務上的表現優於封閉模型如 Claude 3 Haiku，和 GPT4o-mini 媲美。

### Llama Stack 發行版

* 簡化開發人員在不同環境（包括單節點、本地、雲端和設備端）中使用 Llama 模型的方式，與 AWS、Databricks、Dell Technologies、Fireworks、Infosys 和 Together AI 等合作，為其下游企業客戶構建 Llama Stack 發行版。

### 本地運行

* 因為處理是在本地完成的處理速度更快。
* 能夠在本地邊緣設備端中構建，維護隱私不會將訊息和日曆資訊等數據發送到雲端。

## **視覺模型架構和訓練**

### 適配器

* 在文字-圖像對上訓練適配器，以將圖像表示與語言表示對齊，將預先訓練的圖像編碼器整合到預先訓練的語言模型中。
* 在更新影像編碼器的參數時故意不更新語言模型參數。透過這樣完整地保留了所有純文字功能，為開發人員提供了 Llama 3.1 模型的直接替代品。

### 訓練流程

* 首先新增影像適配器和編碼器，然後對大規模雜訊（影像、文字）對資料進行預先訓練。
* 接下來對中等規模的高品質領域內和知識增強（圖像、文字）對資料進行訓練。
* 在後續訓練中，採用與文字模型類似的配方，對監督微調、拒絕抽樣和直接偏好優化進行多輪調整。
* 最後使用 Llama 3.1 模型來過濾和增強域內影像之上的問題和答案，利用合成資料生成，並使用獎勵模型對所有候選答案進行排名，以提供高品質的微調資料。我們還添加了安全緩解數據，以產生具有高安全等級的模型，同時保留該模式的有用性。
* 在訓練後，將上下文長度支援擴展到 128K 個標記，同時保持與預訓練模型相同的品質。合成數據生成，透過仔細的數據處理和過濾來確保高品質，混合數據以優化多種功能的高品質，例如摘要、重寫、指示遵循、語言推理和工具使用。

### **模型評估**

* 3B 模型在指令遵循、摘要、提示重寫和工具使用等任務上的表現優於 Gemma 2 2.6B 和 Phi 3.5-mini 模型，而 1B 模型則與 Gemma 相當。
* 在涵蓋多種語言的 150 多個基準數據集上評估了模型的性能。

<figure><img src="../../.gitbook/assets/image (4).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>



## **輕量級模型的開發**

### 剪枝與知識蒸餾

使用剪枝和蒸餾兩種方法對 1B 和 3B 模型進行優化，使其成為第一批能夠高效適應設備的高性能輕量級 Llama 模型。

#### 模型剪枝

* 對於 1B 和 3B 模型，從 Llama 3.1 8B 中一次性系統地移除部分網路，並調整權重和梯度的大小，以建立一個更小、更高效的模型，同時保留原始網路的性能。

#### 知識蒸餾

* 將 Llama 3.1 8B 和 70B 模型中的 logits 合併到模型開發的預訓練階段，其中這些較大模型的輸出（logits）用作 token 級目標。剪枝後使用知識蒸餾來恢復性能。

<figure><img src="../../.gitbook/assets/image (10).png" alt="" width="563"><figcaption></figcaption></figure>

## **Llama Stack**

### **發行**

* 發布了 Llama Stack API 的意見徵求，這是一個用於規範工具鏈組件（微調、合成數據生成）的標準化介面，用於自定義 Llama 模型和構建代理應用程式。
* 構建了 API 的參考實現，用於推理、工具使用和 RAG。
* 推出了 Llama Stack Distribution，作為一種打包多個 API 提供者的方法，這些提供者可以很好地協同工作，為開發人員提供單一端點。

<figure><img src="../../.gitbook/assets/image (11).png" alt="" width="563"><figcaption></figcaption></figure>

### **內容**

* Llama CLI（命令列介面），用於構建、配置和運行 Llama Stack 發行版。
* 多種語言的客戶端代碼，包括 Python、Node、Kotlin 和 Swift。
* 用於 Llama Stack Distribution Server 和 Agents API Provider 的 Docker 容器。
*   多種發行版，包括：

    * 透過 Meta 內部實現和 Ollama 的單節點 Llama Stack Distribution。
    * 透過 AWS、Databricks、Fireworks 和 Together 的雲端 Llama Stack 發行版。
    * 透過 PyTorch ExecuTorch 在 iOS 上實現的設備端 Llama Stack Distribution。
    * 由 Dell 支援的本地 Llama Stack Distribution。

    <figure><img src="../../.gitbook/assets/image (12).png" alt="" width="563"><figcaption></figcaption></figure>

## **系統級安全**

* 開放的方针有很多好處，它有助於確保全世界更多人能夠獲得人工智慧提供的機會，防止權力集中在少數人手中，並在整個社會中更公平、更安全地部署技術。
* 發布 Llama Guard 3 11B Vision，旨在支援 Llama 3.2 新的圖像理解功能，並過濾文字+圖像輸入提示或對這些提示的文字輸出響應。
* 優化 Llama Guard，大幅降低其部署成本，Llama Guard 3 1B 基於 Llama 3.2 1B 模型，並經過修剪和量化，其大小從 2,858 MB 減少到 438 MB，使其部署效率比以往任何時候都高。。

## **總結**

Llama 3.2 是 Meta 在大型語言模型領域的最新成果，它不僅提升了模型的性能，更引入了視覺理解能力，並針對邊緣和移動設備進行了優化。同時，Meta 推出 Llama Stack 平台，旨在簡化 Llama 模型的部署和應用，並與眾多合作夥伴共同構建開放、安全和負責任的人工智慧生態系統。
