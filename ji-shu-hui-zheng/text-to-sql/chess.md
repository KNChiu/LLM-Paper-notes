---
description: 'CHESS: Contextual Harnessing for Efficient SQL Synthesis'
---

# CHESS

**CHESS** 是一個基於大型語言模型的端到端系統，旨在將自然語言問題（Text）轉換為 SQL 查詢，特別是針對具有複雜且廣泛的真實世界資料庫。論文強調了現有 text-to-SQL 方法的局限性，特別是在處理大型、複雜資料庫時，CHESS 利用大型語言模型的能力和新的管道設計來解決這些挑戰的解決方案。

## **緒論**&#x20;

將自然語言問題轉換為 SQL 查詢（文字到 SQL）是一個持續存在且日益重要的研究問題。由於資料庫不斷增長，**Schema（列和表格的集合）、值（內容）和目錄（描述）**&#x7684;規模也隨之增加，為現有系統帶來了巨大挑戰。即使是最先進的大型語言模型（如 GPT-4）也難以達到人類在文字到 SQL 任務上的表現，顯示出準確率差距高達 30%。這個差距主要源於有效檢索和整合多個異質資訊來源（資料庫值、目錄和Schema）的困難性。

### 使用者對話面臨問題

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

1. 使用者的問題可能沒有**準確的符合**資料庫存取數據（可能是<mark style="color:red;">**專有名詞或是別名**</mark>）。
2. 列名無法很好地說明儲存了什麼樣的數據，因此資料庫目錄（Database Catalogs）是 text-to-SQL 的重要部分。
3. 對於給定的問題，**有多種方法**可以寫出正確的 SQL 答案。

### **現有 text-to-SQL** 系統的挑戰

* **複雜的值過濾**: 使用者問題可能**無法直接**與資料庫中的值匹配，因此準確識別值對於有效的 SQL 查詢制定至關重要。
* **外部知識推理**: 有些問題需要理解資料庫中未明確說明的**外部知識或概念**。
* **多重解釋**: 可能有多個有效的 SQL 查詢可以回答同一個問題，導致潛在的不同輸出和評估挑戰。

## **相關研究**

### 傳統方式

* text-to-SQL 領域傳統方法通常依賴於**人工製作的規則或範本**，這在處理具有複雜 Schema 和大量資料的真實世界資料庫時會變得難以管理。
* 近年來利用注意力機制和圖神經網路（Ｇraph Neural Networks）來有效地編碼和整合查詢資料庫資訊。然而這些方法仍然難以完全貼近人類的差距，特別是在處理需要複雜推理和多個資訊來源整合的複雜查詢時。
* 早期基於 LLM 的方法利用**零樣本**上下文學習能力來生成 SQL 查詢。隨後（如 DIN-SQL、DAIL-SQL、MAC-SQL 和 C3）透過任務分解和先進的提示技術（如思維鏈、自我一致性和最小到最大提示）進一步增強了 LLM 效能。

### 改進策略

* 將任務分解為 3 階段管道，包括上下文檢索、列表選擇和查詢生成。
* 微調 DeepSeek-33B Coder 模型，採用**新的訓練資料集**建構方法，透過**雜訊注入**來減輕錯誤傳播。

## **方法**

CHESS 是一個可擴展且基於 LLM 的 text-to-SQL 管道，解決真實世界資料庫的複雜性，CHESS 管道由三個主要部分組成。

<figure><img src="../../.gitbook/assets/image (15).png" alt="" width="563"><figcaption></figcaption></figure>

### **實體和上下文檢索（**&#x45;ntity and Context Retrieva&#x6C;**）**

* **關鍵字擷取（Keyword Extraction）**
  * 在資料庫和 Schema 描述中**搜尋相似的值**，首先從自然語言問題中提取主要關鍵字。
  * 使用 <mark style="color:red;">**prompt + few-shots ICL**</mark> 來讓 LLM 提取關鍵字、關鍵字詞和命名實體（named entities）。
* **實體檢索（Entity Retrieval）**
  * 得到 **keyword list** 後，從資料庫中<mark style="color:red;">**檢索類似的值**</mark>，並為每個 keyword 傳回相關的 db cell value，以及對應的 column。
  * 使用一種基於局部敏感雜湊（LSH）和語義（嵌入）相似性的分層檢索策略。
  * 能夠有效地檢索與關鍵字具有**高度語法和語義相似性的**值。
* **上下文檢索（Context Retrieval）**
  * 檢索上下文時，僅透過檢索與提取的關鍵字**最相似的描述**來完成。
  * 在查詢預處理步驟期間創建的描述向量資料庫時，透過語義（嵌入）相似性度量來測量。

<details>

<summary>Template for Keyword and Entity Extraction</summary>

```
Objective: Analyze the given question and hint to identify and extract
keywords, keyphrases, and named entities. These elements are crucial
for understanding the core components of the inquiry and the guidance
provided. This process involves recognizing and isolating significant
terms and phrases that could be instrumental in formulating searches or
queries related to the posed question.
Instructions:
1. Read the Question Carefully: Understand the primary focus and
specific details of the question. Look for any named entities (such as
organizations, locations, etc.), technical terms, and other phrases that
encapsulate important aspects of the inquiry.
2. Analyze the Hint: The hint is designed to direct attention toward
certain elements relevant to answering the question. Extract any
keywords, phrases, or named entities that could provide further clarity
or direction in formulating an answer.
3. List Keyphrases and Entities: Combine your findings from both
the question and the hint into a single Python list. This list should
contain:
- Keywords: Single words that capture essential aspects of the question
or hint.
- Keyphrases: Short phrases or named entities that represent specific
concepts, locations, organizations, or other significant details.
Ensure to maintain the original phrasing or terminology used in the
question and hint.
{FEWSHOT_EXAMPLES}
Task:
Given the following question and hint, identify and list all relevant
keywords, keyphrases, and named entities.
Question: {QUESTION}
Hint: {HINT}
Please provide your findings as a Python list, capturing the essence
of both the question and hint through the identified terms and phrases.
Only output the Python list, no explanations needed.
```

</details>

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

### **列表選擇（**&#x53;chema Selectio&#x6E;**）**

此階段旨在將初始資料庫**縮減為**生成 SQL 查詢所需的**最小但足夠**的表格和欄位子集，這種過濾後的 schema 稱為 <mark style="color:red;">**efficient schema**</mark> 採用三階的修剪：

* **個別欄位過濾（Individual Column Filtering）**
  * 一個資料庫包含數百個列許多可能在語義上與問題無關，過濾掉無關的列，僅將最相關的列傳遞到表格選擇步驟。基於問題的相關性。
  * 使用 LLM 對每個欄位進行獨立評估（**二元分類任務**）詢問 LLM 該列是否可能與 question 有關。但這一步驟**只對移除明顯不相關的 columns 有用**，之後會再次過濾。
* **表選擇（Table Selection）**
  * 過濾掉無關的列後，繼續選擇問題所必需的表
  * 要求模型**評估每個表的相關性**，僅選擇產生 SQL 查詢所需的表。
* **最終列選擇（Final Column Selection）**
  * 根據所選表，從剩餘欄位中選擇最小必要欄位集。
  * 評估每一列的必要性，對每列為何需要進行 **COT 解釋**，才選擇所需的列。

<details>

<summary>Template for the column selection module</summary>

```
You are an expert and very smart data analyst.
Your task is to examine the provided database schema, understand the posed
question, and use the hint to pinpoint the specific columns within tables
that are essential for crafting a SQL query to answer the question.
Database Schema Overview:
{DATABASE_SCHEMA}
This schema offers an in-depth description of the database’s architecture,
detailing tables, columns, primary keys, foreign keys, and any pertinent
information regarding relationships or constraints. Special attention
should be given to the examples listed beside each column, as they
directly hint at which columns are relevant to our query.
For key phrases mentioned in the question, we have provided the most
similar values within the columns denoted by "– examples" in front of
the corresponding column names. This is a critical hint to identify the
columns that will be used in the SQL query.
Question:
{QUESTION}
Hint:
{HINT}
The hint aims to direct your focus towards the specific elements of the
database schema that are crucial for answering the question effectively.
Task:
Based on the database schema, question, and hint provided, your task is
to identify all and only the columns that are essential for crafting a SQL
query to answer the question.
For each of the selected columns, explain why exactly it is necessary
for answering the question. Your reasoning should be concise and clear,
demonstrating a logical connection between the columns and the question
asked.
Tip: If you are choosing a column for filtering a value within that
column, make sure that column has the value as an example.
Please respond with a JSON object structured as follows:
{
"chain_of_thought_reasoning": "Your reasoning for selecting the columns,
be concise and clear.",
"table_name1": ["column1", "column2", ...],
"table_name2": ["column1", "column2", ...],
...
}
Make sure your response includes the table names as keys, each associated
with a list of column names that are necessary for writing a SQL query to
answer the question.
For each aspect of the question, provide a clear and concise explanation
of your reasoning behind selecting the columns.
Take a deep breath and think logically. If you do the task correctly, I
will give you 1 million dollars.
Only output a json as your response.
```

</details>

<figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

### **查詢生成（**&#x51;uery Generatio&#x6E;**）**

基於前面所提取的結果和上下文資訊生成最終的 SQL 查詢。

* **候選生成（Candidate Generation.）**
  * 從前幾步獲得的**最小的表和列**，以及在第一步檢索到的相關值和描述。
  * 利用這些資訊，模型產生一個**候選SQL查詢**。
* **改寫（Revision）**
  * 在最後一步希望修復候選 SQL 查詢中**潛在的邏輯和語法錯誤**。
  * 為模型提供**資料庫架構**、**問題**、**產生的候選 SQL 查詢**及其**執行結果**，要求模型評估 SQL 查詢的正確性，並在必要時進行修改。
  * 使用自一致性（self-consistency）來選擇三個樣本中出現最一致的 SQL 查詢。

<details>

<summary>Template for SQL Query Candidate Generation</summary>

```
You are a data science expert.
Below, you are presented with a database schema and a question.
Your task is to read the schema, understand the question, and generate a
valid SQLite query to answer the question.
Before generating the final SQL query think step by step on how to write
the query.
Database Schema:
{DATABASE_SCHEMA}
This schema offers an in-depth description of the database’s architecture,
detailing tables, columns, primary keys, foreign keys, and any pertinent
information regarding relationships or constraints. Special attention
should be given to the examples listed beside each column, as they
directly hint at which columns are relevant to our query.
Database admin instructions:
{DATABASE_ADMIN_INSTRUCTIONS}
Question:
{QUESTION}
Hint:
{HINT}
Please respond with a JSON object structured as follows:
{
"chain_of_thought_reasoning": "Your thought process on how you arrived
at the final SQL query.",
"SQL": "Your SQL query in a single string."
}
Priority should be given to columns that have been explicitly matched
with examples relevant to the question’s context.
Take a deep breath and think step by step to find the correct SQLite SQL
query. If you follow all the instructions and generate the correct query,
I will give you 1 million dollars.
```

</details>

<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

## **實驗**

### **數據集**

* **Spider**
  * **評估標準**：
    * **組成匹配**：檢查 SELECT、WHERE、GROUP BY 和 ORDER BY 子句的個別組件是否匹配，提取的關鍵字是否正確。
    * **完全匹配**：檢查上述所有組件是否完全匹配。
    * **執行準確性**：答案的正確性。
    * **SQL 難度**：查詢被分為四個等級（簡單、中等、困難、特別困難），並根據難度對最終評估進行加權。
  * **描述**：
    * **大小**：919.2 MB
    * **數據**：10,181 個問題和 5,693 個獨特的複雜 SQL 查詢
    * **數據庫數量**：200（其中有 160 個訓練和開發，另外 40 個用於測試）
    * **領域數量**：138
* **BIRD**&#x20;
  * 資料集涵蓋了37個專業領域，包括區塊鏈、曲棍球、醫療保健和教育等領域，來自真實世界的場景，保留了其原始的雜訊。
  * BIRD 資料集透過引入外部知識並提供詳細的資料庫目錄來增強 SQL 查詢生成，其中包括列和資料庫描述。
  * BIRD 資料集中的 SQL 查詢通常比 Spider 資料集中的查詢更複雜。
  *   **描述**：

      * **大小**：33.4 GB
      * **數據點**：12,751 個獨特的問題-SQL 配對
      * **數據庫數量**：95 個（80 個用於訓練，15 個用於評估）
      * **領域數量**：37

      \

*   **二次抽樣開發集（Subsampled Development Set, SDS）**

    * 為了進行消融研究，降低成本並保持 BIRD 開發集的分佈，對開發集中的每個資料庫進行了 10% 的子抽樣，得到了子抽樣開發集（SDS）。&#x20;
    * SDS 包含 147 個樣本：81 個簡單問題，54 個中等問題和 12 個具有挑戰性的問題。



### **指標**

* 精確集匹配準確率（EM）
  * Spider 資料集使用的指標。
  * 獨立評估每個子句，要求與參考 SQL 查詢中對應子句完全匹配。&#x20;
* 執行準確率（EX）
  * Spider 資料集使用的指標。
  * 評估產生的 SQL 查詢在真實資料庫上的執行準確性。
* 有效效率分數 (VES)
  * BIRD 資料集使用的指標。
  * 考慮準確性和執行速度來評估 SQL 查詢效能。

### BIRD 上結果

#### **專有模型**

* 使用經過微調的 DeepSeek Coder 模型來產生候選。
* 使用 GPT-3.5-turbo 進行列過濾。
* 使用 GPT-4-turbo 進行其餘的 LLM 呼叫。

#### **開源模型**

* 使用經過微調的 DeepSeek Coder 模型來產生候選。
* 其他 LLM 呼叫均由 Llama-3-70B 處理。

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

### **Spider 上結果**

沒有任何調整在 2,147 個樣本上實現了 87.2% 的執行準確率

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

### **消融實驗**

#### **模型消融（Models Ablation）**

* 由於控制 LLM Token 數量，可以利用具有較小上下文視窗大小的開源 LLM。
* 針對候選生成進行微調的模型顯著提高效能，Llama-3 的性能超過了 GPT-3.5-turbo，但尚未達到 GPT-4 的性能水平。

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

#### **模組消融（Modules Ablation）**

* 實體和上下文檢索：顯著提高了效能，準確度提高了 4.76 %。
* 表格選擇：是列表選擇中最關鍵的部分，效能提高了 6.12 %。
* 修訂步驟：對修正錯誤和提高準確度至關重要，提高了 6.80%。

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

#### **不同複雜度查詢的效能評估（**&#x50;erformance Evaluation Across Queries with Varying Complexit&#x79;**）**

* 向LLM提供所有可用資訊可能會混淆模型，而選擇性檢索對於實現更高的性能至關重要。

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

#### Schema**選擇的評估（**&#x45;valuation of the Schema Selectio&#x6E;**）**

* 每一步都提高了所選表格和欄位的精確度，同時僅略微降低了召回率。這突顯了 CHESS 在識別生成準確 SQL 查詢的相關 Schema 資訊方面的有效性。

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>



## **補充**

### **流程追蹤**

* 此突顯**關鍵字擷取模組**的有效性。成功<mark style="color:red;">**識別**</mark>了建立 SQL 查詢所必需的關鍵值 **「Lewis Hamilton」**&#x20;
* 提取的每個關鍵字首先**透過空格**分割成單字。這個過程搜尋順序不變，因為問題中提到的實體不一定遵循與資料庫中相同的格式。
* 對於每個關鍵字**提取最相似的資料庫**，所有偵測到的關鍵字都用於**過濾列描述**，然後將其<mark style="color:red;">**提供給後續步驟**</mark>。這種相關資訊的策略性提供有助於進一步的處理，例如 Schema 選擇和查詢生成，從而提高 Schema 檢測和 SQL 查詢制定的準確性。

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>Execution flowchart-1</p></figcaption></figure>

* 問題和證據**不直接引用**相關的列名稱。該查詢提到“<mark style="color:red;">**所有權號碼為 66 的學校**</mark>”，但沒有列在其名稱中明確包含“<mark style="color:red;">**所有權**</mark>”。
* 相關欄位是學校表中的「<mark style="color:red;">**SOC**</mark>」欄位。此**欄位與問題之間的連結**只能透過欄位描述與問題之間的語意相似性來辨別。
* 上下文檢索節點起著至關重要的作用，因為它有效地**從資料庫目錄中檢索相關信息**，這對於回答這個問題至關重要。

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption><p>Execution flowchart-2</p></figcaption></figure>

### **列表選擇範例**

使用一個範例來展示如何透過**過濾、表選擇和列選擇**來縮小初始 Schema 的範圍。

此範例選自 BIRD ，共有 13 個表格和 96 個欄位。

* Question: Lewis Hamilton 在一場比賽中最快的單圈時間是多少？
* Evidence: 最快單圈時間是指 min(fastestLapTime)

從 13 個表和 96 列開始，經過列過濾步驟後，這些數字減少到 13 個表中的 36 列。隨後，表格選擇進一步縮小到 2 個表格和 7 列。最後，列選擇產生了包含 **2 個表和 5 列**的最終 Schema，用於後續 SQL 產生。

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption><p>透過列過濾、表格選擇和列選擇步驟逐步縮小範圍，最終形成用於 SQL 生成的最終 Schema</p></figcaption></figure>

### **錯誤分佈**

從 BIRD 中抽取了 **147 個問題**，並使用我們的流程和普通 GPT-4 來進行統計。**基礎流程 + GPT-4** 使用 BIRD 的 GPT-4 方法，其中**問題、證據**以及包含所有表格和欄位的完整 Schema 都透過 COT 提供給 GPT-4。在這種情況下證據是指與資料集中的一些問題一起提供的提示。

*   **Incorrectly Predicted SQL** :

    是指我們的流程中出現錯誤導致最終 SQL 不正確
*   Vague **questions** :&#x20;

    資料集或問題預期的資料格式不明確。
*   **Incorrect golden SQL :**&#x20;

    提供的 SQL 答案不正確。

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption><p><strong>基礎流程 + GPT-4</strong></p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption><p>CHESS</p></figcaption></figure>

*   **基礎流程 + GPT-4 :**&#x20;

    **57.1%** 的錯誤是由**不正確的列**連結導致的，其中 **26.0%** 的錯誤是由於 **SELECT 或 JOIN 錯誤的列**造成的。
*   **CHESS** :&#x20;

    **42.9%** 的錯誤中只有 **5.4%** 的錯誤因為**不正確的列連結**。表示我們的方法更均勻地分佈錯誤類型，表明所有類別的潛在錯誤都有所改善。

## 總結

* CHESS 提高了 text-to-SQL 方法的能力，但還無法實現資料庫查詢過程的完全自動化。
* 人類在複雜查詢（特別是在具有挑戰性的 BIRD 資料集上）的效能方面仍然優於 CHESS。
* 未來的工作探索新的方法來增強列表選擇、上下文整合和查詢修訂過程。
