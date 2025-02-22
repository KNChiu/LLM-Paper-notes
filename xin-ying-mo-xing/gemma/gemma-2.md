---
description: 'Gemma 2: Improving Open Language Models at a Practical Size'
---

# Gemma 2

## 摘要

* Gemma 2 輕量級開源語言模型系列，參數量範圍為 2B 到 27B。
* 採用已知 Transformer 改進架構，例如交錯使用局部-全局注意力機制（interleaving local-global attentions）和分組查詢注意力機制（group-query attention）。

## 導論

* 儘管小型語言模型的效能快速提升，但主要受限於訓練資料集大小，導致效益遞減，本研究探索在不增加訓練資料量的情況下提高小型模型效能的替代方案，重點關注知識蒸餾。
* 知識蒸餾透過使用大型語言模型作為教師，為小型模型提供更豐富的訓練目標，模擬超出可用資料量的訓練效果，以知識蒸餾取代下一個標記預測來訓練 2B 和 9B 模型。
* 除了透過蒸餾訓練的模型之外，還發布了為此工作從頭開始訓練的 27B 模型。

<figure><img src="../../.gitbook/assets/image (61).png" alt="" width="437"><figcaption></figcaption></figure>

## 模型架構

### 主要架構改進

Gemma 2 基於僅解碼器的 Transformer 架構，上下文長度為 8192 個標記，使用旋轉位置嵌入（RoPE）和近似 GeGLU 非線性函數。

* 局部滑動窗口（Local Sliding Window）和全局注意力（Global Attention）：
  * 每隔一層交替使用局部滑動窗口注意力和全局注意力。
* Logit 軟限制（soft-capping）
  * 限制每個注意力層和最終層中的 Logit 值。
* 後置歸一化（Post-norm）和帶 RMSNorm 的預歸一化（pre-norm）
  * 使用 RMSNorm對每個 Transformer 子層、注意力層和前饋層的輸入和輸出進行歸一化。
* 群組查詢注意力（Grouped-Query Attention）
  * 使用 num\_groups = 2 的 GQA，在保持下游效能的同時提高推理速度。

### 預訓練（Pre-training）

* Tokens
  * Gemma 2 27B 使用 13T tokens 進行訓練，9B 模型使用 8T tokens，2B使用 2T tokens。
* Tokenizer
  * 使用與 Gemma 1 和 Gemini 相同的 tokenizer，一個具有分割數字、保留空格和位元組級編碼的 SentencePiece 詞彙表。
*   關鍵差異

    * 訓練資料：主要來自網路文件、程式碼和科學文章的英文資料。
    * 知識蒸餾：使用大型語言模型作為教師，在超過理論預測計算最佳詞彙數 50 倍的詞彙量上訓練小型模型。
    * 計算基礎設施：使用 TPUv4、TPUv5e 和 TPUv5p 進行訓練。
    * 碳足跡：預計訓練 Gemma 模型產生的碳排放量為 1247.61 噸二氧化碳當量。

    <figure><img src="../../.gitbook/assets/image (62).png" alt="" width="432"><figcaption></figcaption></figure>

### 後訓練（Post-training）

* 對預先訓練的模型進行微調，將其轉變為指令微調模型（instruction-tuned models）。
* 對純文字、純英語合成和人類生成的提示回應對（prompt-response pairs）進行微調（SFT）。
* 在這些模型的基礎上使用 RLHF，獎勵模型使用標記的存英文偏好資料訓練，使用與 SFT 階段相同的提示策略。
* 最後將每個階段獲得的模型進行平均，以提高整體效能。

#### 關鍵差異

* 資料：使用來自內部和外部公開資料的混合資料，其中使用 LMSYS-chat-1M 的提示但不包括答案。
* 監督式微調（SFT）：主要使用教師（一個更大的模型）合成的提示和回應進行行為轉移。
* 從人類反饋中強化學習 (RLHF)：使用與 Gemma 1.1 類似的 RLHF 演算法，但使用不同的獎勵模型。
* 模型合併：將使用不同超參數運行的管道獲得的不同模型進行平均。
* 資料過濾：過濾資料以去除包含特定個人資訊、不安全或有毒模型輸出、錯誤的自我識別資料和重複範例的資料。
* 格式：Gemma 2 模型使用與 Gemma 1 模型相同的控制詞彙進行微調，如表 4 所示但格式架構不同。

<figure><img src="../../.gitbook/assets/image (60).png" alt="" width="435"><figcaption></figcaption></figure>

## 模型簡化測試

重點關注知識蒸餾對小型語言模型的影響。

<figure><img src="../../.gitbook/assets/image (63).png" alt="" width="447"><figcaption></figcaption></figure>

### 主要成果

* 與從頭開始訓練相比，從更大的模型中進行蒸餾可以提高效能，即使在訓練詞彙數相同的情況下也是如此，蒸餾的影響會隨著模型規模的增加而保持。
* 隨著模型尺寸的縮放，增益仍然存在。在此實驗將教師的尺寸保持在 7B，並訓練較小的模型來模擬最終教師和學生尺寸之間的相同差距（模型困惑度越低越好）。

<figure><img src="../../.gitbook/assets/image (64).png" alt="" width="455"><figcaption></figcaption></figure>

* 將多頭注意力（MHA）替換為 GQA 對效能的影響很小，但 GQA 只需要更少的參數並且推理速度更快。

<figure><img src="../../.gitbook/assets/image (66).png" alt="" width="437"><figcaption></figcaption></figure>

* 在相同參數數量下，更深的 9B 網路略優於更寬的 9B 網路。

<figure><img src="../../.gitbook/assets/image (67).png" alt="" width="436"><figcaption></figcaption></figure>

* 可以在推理過程中調整局部注意力層的滑動窗口大小，對困惑度的影響不大。

<figure><img src="../../.gitbook/assets/image (68).png" alt="" width="439"><figcaption></figcaption></figure>

* 與其他模型相比，Gemma 2 模型對提示/評估格式變化的穩健性更高。

<figure><img src="../../.gitbook/assets/image (69).png" alt="" width="431"><figcaption></figcaption></figure>

## 評估

本節在各種自動化基準測試和人工評估中評估預先訓練和指令微調模型的效能。

### 預先訓練（Pre-training）模型評估

* 27B 在同等規模的模型中表現最佳，甚至可以與訓練時間更長、規模更大的模型相媲美。
* 與先前版本相比，2B 和 9B 參數模型有顯著改進，這證實了知識蒸餾的有效性。

<figure><img src="../../.gitbook/assets/image (16).png" alt="" width="442"><figcaption></figcaption></figure>

### 後訓練（Post-training）模型評估

* LMSYS Chatbot Arena&#x20;
  * Gemma 2 指令微調模型的表現優於同等規模的其他開源模型。
* 人工偏好評估（Human Preference Evaluations）
  * 與舊版 Gemma 1.1 70Ｂ相比，勝率和偏好評分方面有很大改進。
* 人工多輪評估（Human Multi-Turn Evaluations）
  * 在用戶滿意度和對話目標達成方面，Gemma 2 模型明顯優於 Gemma 1.1。
* 與預訓練的模型相比
  * 指令微調模型在少樣本基準測試中表現更好，這可能是因為它們能夠更好地理解格式化的問題。

<figure><img src="../../.gitbook/assets/image (17).png" alt="" width="563"><figcaption></figcaption></figure>

## 記憶和隱私

* 與同等規模的先前模型相比，Gemma 2 記憶的訓練資料要少得多，記憶率低於 0.1%。
* 程式碼、維基百科和科學資料的記憶率更高，但總體記憶率仍然很低。
* 近似記憶率高於精確記憶率，但仍然很低。
* 沒有發現任何高嚴重性個人資料洩露的實例，包含低嚴重性個人資訊的記憶資料的比例非常低（0.00026%）。

## 責任、安全、保障

* 影響評估：
  * 認為開放式人工智慧可以將這些技術的益處傳播到整個社會，但必須評估其被惡意使用的風險。
* 安全策略和訓練階段的緩解措施：
  * 根據 Google 的安全策略對微調模型進行調整，以防止模型產生有害內容。
* 外部基準測試評估：
  * 在公開基準測試上評估 Gemma 2，以確保其安全性和可靠性。
* 保障評估：
  * 針對網路安全、程式碼漏洞偵測、化學、生物、放射性和核能（CBRN）知識以及自我擴散等方面評估模型的潛在危害。
* 負責任的開放模型方法：
  * 提倡系統級方法來設計安全、可靠和負責任的應用程式，並繼續開發負責任的生成式人工智慧工具包，以支持開發人員。

## 討論和結論

* Gemma 2 是 Gemma 系列開源語言模型的最新成員，專為文字和程式碼設計。
* 知識蒸餾是一種有效的訓練方法，其結果優於原始文字訓練。
* 希望發佈這些模型能夠促進人工智慧領域的進一步研究和發展。
* 雖然存在固有的風險，但廣泛的安全調查和負責任的部署程序表明，這些模型將對社會產生積極的影響。

## 總結

Gemma 2 代表著開源大型語言模型的顯著進步。透過採用知識蒸餾、改進 Transformer 架構和嚴格的安全措施，Gemma 2 不僅在同等規模的模型中樹立了新的效能標竿，同時也優先考慮了負責任的人工智慧原則。儘管仍有改進空間，但 Gemma 2 的發佈為人工智慧社群提供了強大的工具，有可能推動創新應用，並進一步開展負責任的人工智慧研究。
