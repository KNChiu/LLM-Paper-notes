---
description: 'Prefix-Tuning: Optimizing Continuous Prompts for Generation'
---

# Prefix-Tuning

## **摘要 (Abstract)**

* 傳統的微調方法會修改大型預訓練語言模型的所有參數，導致每個任務都需要儲存一個完整的模型副本。
* **前綴微調（**&#x50;refix-Tunin&#x67;**）是一種輕量級的替代方案，凍結了語言模型的參數，但優化了一個小的、連續的、特定於任務的向量（稱為前綴）。**
* 前綴微調受到提示學習的啟發，允許後續的詞元像「虛擬詞元」一樣關注這個前綴。
* 在 GPT-2 (用於表格到文本生成) 和 BART (用於摘要) 上的實驗表明，**僅學習 0.1% 的參數，前綴微調在全數據集設置中取得了可比較的性能，在低數據集設置中優於微調，並且在訓練期間未見過的主題上具有更好的外推能力。**

## **導論 (Introduction)**

* 微調是利用大型預訓練語言模型執行下游任務的主要方法，但其缺點在於需要更新和儲存整個語言模型的所有參數。
* 對於擁有數十億參數的大型語言模型（例如 GPT-2 和 GPT-3），為每個任務儲存一個完整的模型副本在成本上可能非常高昂。

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt="" width="563"><figcaption><p>微調（上）更新所有參數（紅色）。前綴調整（下）凍結參數並僅優化前綴（紅色）</p></figcaption></figure>

## **相關工作 (Related Work)**

#### **輕量級微調**&#x20;

凍結大部分預訓練參數並添加小型可訓練模塊的輕量級微調方法，例如 adapter-tuning。與這些方法相比，前綴微調能夠進一步大幅減少特定於任務的參數數量 (僅 0.1%)，同時保持可比較的性能。

#### **提示學習 (Prompting)**&#x20;

討論了通過在輸入前加上自然語言指令和少量範例來利用大型語言模型 (如 GPT-3) 的提示學習方法。與手動設計離散提示或自動搜索離散觸發詞 (如 AutoPrompt) 不同，前綴微調優化的是連續的前綴向量，這被認為更具表現力。

#### **可控生成 (Controllable generation)**

提及了旨在引導預訓練語言模型生成具有特定屬性 (例如情感或主題) 的文本的方法，但這些方法難以應用於需要細粒度控制的任務 (如表格到文本和摘要)。

## **前綴微調 (Prefix-Tuning):**

#### **直覺 (Intuition)**

基於提示學習的直覺，認為適當的上下文可以引導語言模型而無需改變其參數。與優化離散詞元不同，前綴微調優化連續的詞嵌入，這些嵌入的效果會傳播到所有 Transformer 激活層和後續詞元。

#### **方法 (Method)**

在自回歸語言模型的輸入前加上一個可訓練的連續前綴，或在編碼器和解碼器的輸入前都加上前綴。語言模型的參數保持凍結，只有前綴的參數被優化。前綴的激活直接來自一個可訓練的矩陣。

#### **Pθ 的參數化 (Parametrization of Pθ)**

為了穩定優化和提高性能，前綴矩陣 Pθ 通過一個較小的矩陣 P'θ 和一個大型前饋神經網路 (MLPθ) 進行重參數化。訓練完成後，重參數化的參數可以丟棄，只需要保存前綴 Pθ。

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt="" width="563"><figcaption><p>使用自回歸 LM（上）和編碼器-解碼器模型（下）進行前綴調整的註解範例</p></figcaption></figure>

## **實驗設置 (Experimental Setup):**

#### **數據集和評估指標 (Datasets and Metrics)**

用於表格到文本生成 (E2E, WebNLG, DART) 和摘要 (XSUM) 任務的標準數據集，以及每個任務使用的評估指標 (BLEU, NIST, METEOR, ROUGE, CIDEr, TER, MoverScore, BERTScore, BLEURT)。

#### **方法 (Methods)**

與前綴微調進行比較的其他方法，包括完整微調 (FINE-TUNE)、僅微調頂部兩層 (FT-TOP2) 和 adapter-tuning (ADAPTER)。

#### **架構和超參數 (Architectures and Hyperparameters)**

實驗中使用的語言模型架構 (GPT-2MEDIUM, GPT-2LARGE 用於表格到文本；BARTLARGE 用於摘要) 以及輸入的處理方式。優化器 (AdamW) 和學習率調整策略，超參數的調整和訓練資源。

## **主要結果 (Main Results):**

#### **表格到文本生成 (Table-to-text Generation)**

前綴微調僅添加 0.1% 的特定於任務的參數，就能有效地進行表格到文本生成，**優於其他輕量級基線 (ADAPTER 和 FT-TOP2)，並在性能上與完整微調相當甚至更好。** 尤其是在 DART 這個更複雜的開放領域數據集上表現良好，其具有良好的泛化能力。

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt="" width="563"><figcaption><p>E2E（左）、WebNLG（中）和 DART（右）</p></figcaption></figure>

#### **摘要 (Summarization)**

前綴微調在摘要任務上（使用 XSUM 數據集）的性能略低於完整微調，可能因為 XSUM 數據集更大、輸入文章更長以及摘要任務可能更複雜。

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt="" width="563"><figcaption><p>XSUM 指標，前綴調整的效果略遜於微調</p></figcaption></figure>

### **低數據設置 (Low-data Setting)**

* **在前綴微調在訓練樣本數量較少的情況下具有明顯的優勢，性能顯著優於完整微調。**&#x20;
* 隨著數據集規模的增加，這種差距會縮小。
* 在低數據情況下，前綴微調生成的文本往往比完整微調更忠實於輸入表格的內容。

### **外推 (Extrapolation)**

* **綴微調在未見過的主題上的外推性能優於完整微調。**
* Adapter-tuning 在外推方面也取得了與前綴微調相當的良好性能，這可能表明保留預訓練語言模型的參數對外推有積極影響。

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt="" width="563"><figcaption><p>XSUM 上的外推效能。在新聞到體育的劃分或在新聞內部的劃分，前綴調整的效果都優於微調。</p></figcaption></figure>



## **內部評估 (Intrinsic Evaluation)**

#### **前綴長度 (Prefix Length)**

實驗中性能隨著前綴長度的增加而提高，直到達到一個閾值（摘要為 200，表格到文字為 10），之後性能可能會略有下降。較長的前綴對推理速度的影響可以忽略不計。

<figure><img src="../../../.gitbook/assets/image (7).png" alt="" width="563"><figcaption><p>前綴長度與摘要（左）和表格到文字（右）的表現</p></figcaption></figure>

#### **完整前綴與僅嵌入 (Full vs Embedding-only)**

僅優化「虛擬詞元」的詞嵌入 (embedding-only ablation) 的性能顯著低於優化完整的前綴激活，這表明僅調整嵌入層的表現力不足，離散提示優化的性能上限。

#### **前綴與中綴 (Prefixing vs Infixing)**

將可訓練的激活放置在輸入 x 和輸出 y 之間 (infix-tuning) 的性能略低於放置在開頭 (prefix-tuning)，這可能是因為前綴微調可以影響 x 和 y 的激活。

<figure><img src="../../../.gitbook/assets/image (8).png" alt="" width="563"><figcaption><p>嵌入和中綴的評估。僅嵌入消融和中綴調整的表現均不如完整前綴調整。</p></figcaption></figure>



#### **初始化 (Initialization)**

* **使用真實詞元的激活來初始化前綴比隨機初始化在低數據設置下能顯著提高性能。**&#x20;
* 使用與任務相關的詞元 (例如 "summarization", "table-to-text") 進行初始化略優於使用不相關的詞元。
* 這種初始化策略與盡可能保留預訓練語言模型一致。

<figure><img src="../../../.gitbook/assets/image (9).png" alt="" width="563"><figcaption><p>在低數據下，使用真實詞彙來初始化前綴的效果明顯優於隨機初始化</p></figcaption></figure>

## **討論 (Discussion)**

#### **個性化 (Personalization):**

前綴微調非常適合需要為大量獨立任務進行訓練的場景，例如在保護用戶隱私的個性化設定中，每個用戶可以擁有獨立的前綴。

#### **跨用戶批處理 (Batching Across Users)**

在個性化設定下，前綴微調允許對來自不同用戶的查詢進行批處理，因為共享的語言模型保持不變，批處理只需要簡單地在用戶輸入前加上其個性化的前綴。

#### **前綴微調的歸納偏置 (Inductive Bias of Prefix-tuning)**

* 由於前綴微調和 adapter-tuning 都凍結了預訓練參數，這可能有利於模型泛化到訓練期間未見過的領域。
* 前綴微調比 adapter-tuning 需要的參數量更少，這可能因其更徹底地利用了預訓練語言模型的能力。
* 通過僅更新少量參數即可獲得與完整微調相當的性能。

## **結論 (Conclusion)**

* **前綴微調是一種用於自然語言生成任務的輕量級微調替代方案，它在輸入前添加一個可訓練的連續前綴。**
* 儘管學習的參數比完整微調少 1000 倍，但前綴微調在全數據集設置中仍能保持可比較的性能，並且在低數據和外推設置中優於完整微調。
* **前綴微調的核心思想是通過優化一個小型的連續前綴來引導預訓練語言模型生成期望的輸出，而無需修改模型本身的大量參數。這種方法在參數效率、低數據學習和外推能力方面展現出了優勢。**
