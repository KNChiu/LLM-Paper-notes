---
description: 'phi-1: Textbooks Are All You Need，高品質數據集對小模型影響與合成數據概念'
---

# Phi-1

### 1. 介紹

* phi-1 是一個針對程式碼生成任務的大型語言模型，具有1.3B參數，訓練在高品質數據(教科書等級品質)上。
* 模型在 HumanEval 上達到 50.6% 的 pass@1 準確率，在 MBPP 上達到 55.5% 的 pass@1 準確率。

### 2. 訓練細節及高品質數據的重要性

#### 2.1 過濾現有的程式碼數據集

* 使用 Transformer-based 分類器來過濾數據。
* GPT-4 用來標註數據質量，篩選出高教育價值的程式碼片段。

#### 2.2 創建合成的教科書品質數據集

* 使用 GPT-3.5 生成 Python 教科書數據，確保數據集的多樣性和非重複性。
* 合成數據集包括約1B tokens的教科書數據和約180M tokens的練習數據。

#### 2.3 模型架構與訓練過程

* phi-1 和 phi-1-base 的預訓練使用 CodeTextbook 數據集(phi-1-base 29%)，微調使用 CodeExercises 數據集(phi-1 51%)。
* 模型參數小與架構簡單，但數據質量有顯著提高模型性能，且解鎖了意想不到的程式碼生成功能。

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption><p>增加計算時間(135 -> 1090 GPU hr)與數據(26B -> 76B）或增加模型參數（350M -> 1.3B），且在 CodeTextbook 資料集上訓練的基於 phi-1 的模型僅用 1.3B 參數模型即可實現 29% 的 HumanEval 效能。</p></figcaption></figure>

### 3. 湧現(emergence)性質

* 比較 phi-1 和 phi-1-small 發現模型參數對湧現能力的重要性。
* phi-1 展現出許多 phi-1-base 沒有的能力，如更高的程式碼準確性和靈活性。

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### 4. 替代基準測試

* CodeExercises 數據集可能會產生記憶(訓練數據污染)
* 為了最大限度地減少資料外洩（data leakage) 使用其他測試方法如不同的程式碼生成任務來評估模型的性能。

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>對 50 個非常規程式問題的理解進行評分</p></figcaption></figure>

### 5. 訓練數據污染的研究

* 分析 HumanEval 基準測試中的數據污染問題。
* 強調訓練數據的透明度和可靠性對於模型性能的重要性。

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption><p>移除 CodeExercises 資料集中與 HumanEval「相似」的內容</p></figcaption></figure>

### 重點整理

1. **什麼是 phi-1 模型？**
   * phi-1 是一個針對編碼任務的大型語言模型，具有1.3B參數，訓練在高品質數據上，展示出優越的程式碼生成能力。
2. **phi-1 模型的主要優勢是什麼？**
   * phi-1 模型在基準測試中表現優異（HumanEval: 50.6% pass@1，MBPP: 55.5% pass@1），並且展示出突現性質，高效且靈活。
3. **phi-1 模型的主要劣勢是什麼？**
   * phi-1 模型訓練成本高，依賴高品質數據，且可能存在數據污染風險，影響模型的公平性和準確性。
4. **什麼是合成數據？**
   * 合成數據是通過人工方法或生成模型創建的數據，模擬真實世界數據但可精確控制其特性和質量，無敏感信息且生成快速。

***

### 簡短重點

* **優勢**: 高性能模型 高品質數據 突現性質
* **劣勢**: 數據污染風險 訓練成本高 依賴高品質數據
* **教科書數據**: 合成教科書數據 合成練習數據 數據過濾與標註
* **合成數據**: 人工生成數據
* **優勢**: 控制性強 無敏感信息 快速生成
* **取得手法**: 數據生成模型 數據擴充 數據過濾與優化
