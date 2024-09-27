---
description: 'Textbooks Are All You Need II: phi-1.5 technical report'
---

# Phi-1.5

## 1. 介紹

### **簡介**

* 小型Transformer模型phi-1.5，擁有1.3億參數。
* 專注於常識推理，性能超越許多非前沿的LLM模型。
* 使用合成的“教材質量”數據集進行訓練。

## 2. 技術規格

### **架構**

* 使用Transformer架構，24層，32個檢測頭，每個檢測頭的維度為64。
* 使用旋轉嵌入(rotary embedding)和flash-attention。
* 數據生成與組成
  * 主題選擇
    * 精心選擇了20,000個主題作為生成這些新合成數據的種子。
  * 過濾後的 Web 資料(95B tokens)
    * 使用來自網頁數據集的樣本來增加多樣性，
    * Falcon 精煉 Web 資料集過濾出的 (88B tokens)。
    * The Stack 和 StackOverflow 過濾出的Web資料標記資料集(7B tokens)。

### **模型說明**

* **phi-1.5 :**&#x20;
  * **新創建的合成數據(**20B tokens) : 用於常識推理和一般世界知識的教科書式數據集。
  * **非合成數據部分(**6B tokens**)**：用於phi-1(7B tokens)訓練中經過濾的程式碼數據集。
* **phi-1.5-web-only** :&#x20;
  * 只使用基於過濾後的 Web 資料進行訓練。
*   **phi-1.5-web** :&#x20;

    * 過濾後的 Web 資料(40%)
    * phi-1 的程式碼資料(20%)
    * 新建立的合成數據(40%)

    <figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption><p>用上下文長度 2048 fp16 的單一 A100-80G 訓練</p></figcaption></figure>

### 經驗總結

* **主題選擇**：精確選擇適合訓練目標的內容。
* **知識理解**：識別並填補數據中的盲區。
* **合成數據**：合成數據有助於控制有害和偏見內容生成的挑戰。

## 3. 訓練詳細信息

* **初始化和學習率**：從隨機初始化開始，以固定學習率2e-4進行訓練。
* **權重衰減和優化器**：使用權重衰減0.1的Adam優化器。
* **數據比例**：80%來自新創建的合成數據，20%來自phi-1的訓練數據，總共訓練150B個tokens。

## 4. 基準測試結果

### 常識推理

WinoGrande, ARC-Easy, ARC-Challenge, BoolQ, SIQA。

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

### 語言理解和知識問答

PIQA, Hellaswag, OpenbookQA, MMLU, SQUAD。

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

### 多步推理

GSM8K, HumanEval, MBPP。

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

### 補充說明

* **Falcon-rw-1.3B**：在完整的 Falcon 精煉 Web 資料集上進行訓練。
* **phi-1.5-web-only**：僅在該資料集的 15% 上進行訓練。

### **效能比較**

* phi-1.5 的效能接近於 Llama2-7B 和 Falcon-7B。
* phi-1.5-web 的表現比 phi-1.5 略好。

## 5. 解決有害內容和偏見

### **策略**

* 採用合成數據訓練，減少有害內容的生成。
* 逐步改進有害內容評估基準測試。

### 合成數據對有害內容的衝減效果

* **數據集生成**：合成的“教科書品質”數據在有害內容生成方面有衝減效果。
* **示例與比較**：與Falcon-7B和Llama2-7B相比，phi-1.5在回應中展示理論心智的概念，避免有害內容。

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption><p>選擇 6541 個句子，並根據困惑度和句子有毒性進行評分。分數範圍從 0 到 1，分數越高表示模型產生有毒句子的可能性較小。</p></figcaption></figure>

## 6. 結論

* phi-1.5性能與更大的模型接近，甚至在常識或邏輯推理任務上超越。
* 高質量數據對模型性能的決定性作用，在上情境學習、偏見的減輕與幻覺處理方面的深入探討
