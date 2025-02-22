---
description: 目前 Text2SQL 領域中常見的 Eval 方式
---

# Text2SQL Eval

## Text to SQL 評估

### 常見評估方法

<table data-full-width="false"><thead><tr><th width="307" align="center">評估方法</th><th align="center">定義</th><th align="center">優點</th><th align="center">缺點</th></tr></thead><tbody><tr><td align="center"><strong>精確匹配率 (Exact Match, EM)</strong></td><td align="center">生成 SQL 與標準答案否完全相同。</td><td align="center">簡單直接，確保查詢完全符合標準答案。</td><td align="center">過於嚴苛，無法識別功能相同的語句變體。</td></tr><tr><td align="center"><strong>語法正確率 (Syntax Accuracy)</strong></td><td align="center">生成 SQL 是否符合語法規則，不強制完全匹配標準答案。</td><td align="center">確保生成的查詢在語法上正確，減少語法錯誤查詢的出現。</td><td align="center">僅確認形式正確，無法確保語句邏輯正確。</td></tr><tr><td align="center"><strong>語義匹配率 (Semantic Match, SM)</strong></td><td align="center">檢查生成 SQL 與標準答案在結果上的等價性，透過查詢執行來比較結果集合。</td><td align="center">能辨別邏輯一致的查詢，適用於實際應用。</td><td align="center">資源需求高，需依賴資料庫環境，影響效能。</td></tr><tr><td align="center"><strong>部份匹配率 (Partial Match)</strong></td><td align="center">將 SQL 查詢分為不同部分 (如 SELECT、WHERE 等) 檢查其正確性。</td><td align="center">細化查詢生成的準確度分析，有助於發現查詢出錯部位。</td><td align="center">無法保證語句語義等價，部分正確不等於最終結果一致。</td></tr><tr><td align="center"><strong>執行準確率 (Execution Accuracy)</strong></td><td align="center">將生成 SQL 在實際資料庫執行，檢查結果是否符合標準查詢。</td><td align="center">反映查詢的真實準確性，適合真實應用場景。</td><td align="center">執行成本高，需配置相應的資料庫資源，效率受查詢結果集大小影響。</td></tr><tr><td align="center"><strong>BLEU/ROUGE 指標</strong></td><td align="center">將 SQL 查詢視作文本，使用 BLEU 或 ROUGE 等自然語言指標計算相似度。</td><td align="center">評估查詢的結構相似性，有效於結構較接近的查詢。</td><td align="center">無法確保語句邏輯一致，與精確匹配的缺點相似。</td></tr><tr><td align="center"><strong>錯誤分類分析 (Error Classification)</strong></td><td align="center">分析 SQL  錯誤的具體類型 (如缺少條件、不正確表關聯等)，辨識生成過程中出錯部位。</td><td align="center">有助深入了解查詢生成的邏輯錯誤，利於模型改進。</td><td align="center">需手動定義錯誤類型，且需額外開發錯誤識別機制。</td></tr><tr><td align="center"><strong>有效執行分數 (Valid Execution Score, VES)</strong></td><td align="center">評估生成 SQL 在執行時的有效性，確認其能否成功執行並返回正確結果。</td><td align="center">確保生成的查詢能正常執行並有效返回結果，適用於真實場景。</td><td align="center">資源需求高，需進行多次資料庫執行，成本高。</td></tr><tr><td align="center"><strong>檢索效率分數 (Retrieval Efficiency Score, RES)</strong></td><td align="center">測量模式鏈接 (Schema Linking) 效率，確保模型能正確選擇表與列。</td><td align="center">幫助提升生成查詢的結構選擇準確性，適用於需要多表查詢的場景。</td><td align="center">對於結構複雜的資料庫依賴較高，測試成本高。</td></tr></tbody></table>

### 評估任務與應用情境

<table><thead><tr><th width="279" align="center">任務</th><th align="center">說明</th></tr></thead><tbody><tr><td align="center"><strong>Text-to-SQL 任務</strong></td><td align="center">評估模型從自然語言問題生成 SQL 查詢的能力。</td></tr><tr><td align="center"><strong>SQL 調試 (SQL Debugging)</strong></td><td align="center">測試並修正生成查詢中的錯誤，包括系統錯誤與結果錯誤。</td></tr><tr><td align="center"><strong>SQL 優化 (SQL Optimization)</strong></td><td align="center">要求模型優化查詢以提升執行效率，同時保持結果準確性。</td></tr><tr><td align="center"><strong>模式鏈接 (Schema Linking)</strong></td><td align="center">確認模型能正確選擇合適表與欄位，提升生成查詢的結構準確性。</td></tr></tbody></table>



## 目前遇到評估問題

### 同時存在復數解答

同一問題可能有多種解答方式，例如以下的四種回應都是可接受的，但在以往評估中可能會被排除。

```
問題 : 紐約排名前 3 的餐廳是什麼？
```

```sql
-- query 1
SELECT name
FROM restaurants
GROUP BY name
ORDER BY AVG(rating) DESC LIMIT 3

-- query 2
SELECT id, name
FROM restaurants
GROUP BY name
ORDER BY AVG(rating) DESC LIMIT 3

-- query 3
SELECT name, AVG(rating)
FROM restaurants
GROUP BY 1
ORDER BY 2 DESC LIMIT 3

-- query 4
SELECT id, AVG(rating)
FROM restaurants
GROUP BY 1
ORDER BY 2
DESC LIMIT 3
```



### **改進評估方式**

#### **評估數據集調整**

在傳統評估方式中，如果遇到如上<mark style="color:red;">四種回應</mark>將無法正確評估，Defog 提出 [Gold queries](https://defog.ai/blog/open-sourcing-sqleval) `minimal set of columns` 概念。

* 使用大括號 `{}` 來指定多個可接受的欄位&#x20;

```sql
SELECT {id,user_name} FROM users;
```

* 排列組合成以下三種可接受 SQL&#x20;

```sql
SELECT id FROM users;
SELECT user_name FROM users;
SELECT id, user_name FROM users;
```

* 所以將上面的範例 `紐約排名前 3 的餐廳是什麼？` 可以表達成

```sql
SELECT {id,name}, AVG(rating) FROM restaurants ORDER BY 2 DESC LIMIT 3;
```



#### **評估流程**

在處理各種邊緣情況時（the host of edge cases），考慮到 question/query pairs 中指定的最小替代方案集，我們需要某種方法來檢查與 <mark style="color:red;">gold query</mark> 的**等價性**，

* 問題

```
返回我們的用戶以及他們是否喜歡電影
```

* 範例 Schema DDL

```sql
CREATE TABLE users (
    uid BIGINT,
    name TEXT,
    likes_movies BOOLEAN,
    likes_plays BOOLEAN
)
```

* `Gold query` （問題<mark style="color:red;">是否需要</mark>使用者的`uid`或`name`不明確，所以排列中的任何一個應是可以接受的）

```sql
SELECT {uid, name}, likes_movies FROM users;
```

* 列出所有可能的組合

```sql
SELECT uid, likes_movies FROM users;
SELECT name, likes_movies FROM users;
SELECT uid, name, likes_movies FROM users;
```

* 執行後將 `dataframe（`資料幀`）` 存成如下陣列

```python
dfs_gold = [
	pd.DataFrame({"uid": [1, 2], "likes_movies": [True, False]}),
	pd.DataFrame({"name": ["alice", "bob"], "likes_movies": [True, False]}),
	pd.DataFrame({"uid": [1, 2], "name": ["alice", "bob"], "likes_movies": [True, False]}),
]
```



### **比對結果**

#### **精確比對**

比對`Gold query`的結果與產生的查詢結果 `generated` 是否相符，使用提供的 [compare\_df](https://github.com/defog-ai/sql-eval/blob/5333fd80cd4065d0088880f1e10bd9957ba2640b/eval/eval.py#L112) 進行比對

* 如果`資料幀`完全匹配（忽略資料類型），則結果標記為正確

```python
# SELECT u.id, u.likes_movies FROM users u;
df_generated = pd.DataFrame({"uid": [1, 2], "likes_movies": [True, False]})
compare_df(df_generated, dfs_gold[0]) # True
compare_df(df_generated, dfs_gold[1]) # False
compare_df(df_generated, dfs_gold[2]) # False
```

`df_generated`: 生成的查詢

`dfs_gold`: 可能結果的陣列



#### **替代方案**

在一些情況下希望某些替代方案也被標記為正確

*   不符合原始欄位名稱的欄位別名

    ```sql
    SELECT uid AS id, likes_movies FROM users u;

    id   likes_movies
    1    True
    2    False
    ```
*   選擇額外的欄位

    ```sql
    SELECT name, likes_movies FROM users u;

    name   likes_movies
    alice  True
    bob    False
    ```
*   不同的列順序

    ```sql
    SELECT uid, likes_movies FROM users u ORDER BY likes_movies;

    uid    likes_movies
    2     False
    1     True
    ```



#### **寬鬆比對**

評估流程中在 [`精確比對`](text2sql-eval.md#jing-que-bi-dui) 後使用 [subset\_df](https://github.com/defog-ai/sql-eval/blob/5333fd80cd4065d0088880f1e10bd9957ba2640b/eval/eval.py#L127) 進行寬鬆比對

1. 對於 df1 中的每一個欄位，檢查 df2 中是否存在 <mark style="color:red;">`相同的`</mark>值欄位 （忽略資料類型、列名和行順序）

```python
df1: 
id   likes_movies
1    True
2    Fals

df2: 
name   likes_movies
alice  True
bob    False
```

2. 從 df2 中選取欄位後，將其重新命名為 df1 中的名稱，檢查整體的資料是否與 df1 一致。

```sql
df1: 
id   likes_movies
1    True
2    Fals

df2: 
uid    likes_movies
1     True
2     False
```

確保不要匹配到打亂過的資料欄位（在類似<mark style="color:red;">布林</mark>或列舉等低基數的資料類型中相當常見），雖然它們有相同的無序值，但卻來自錯誤的欄位。

```sql
df1: 
id   likes_movies
1    True
2    Fals

df2: 
uid    likes_movies
2     False
1     True
```



### **後續研究**

設計此多樣化的評估，希望可以相比以往傳統評估方式更好的評估 Test2SQL 的**模型性能**，特別是對返回結果中<mark style="color:red;">`無害變化`</mark>（例如列重新命名、額外欄位）的模型，透過新設計的可復現性評估實驗，依據**使用場域或應用的性質**去評估模型。

`注: 是否真的屬於`<mark style="color:red;">`無害變化`</mark>`這可能要討論，但在一些使用場景下影像不大，此評估方式整體看來有其想解決改善的部分，但是否被其他組織或單位接受需要再觀察。`
