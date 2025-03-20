---
description: >-
  對於 LLM Fine-Tune 常用方式中，LoRA（Low-Rank Adaptation）系列技術因參數效率高、不增加推理延遲等優勢，成為業界 LLM
  微調常用的手段。
---

# LoRA 系列

### LoRA 系列論文

1. LoRA 原始論文 (2021.06.17)
   * 論文：《LoRA: Low-Rank Adaptation of Large Language Models》
   * 關鍵貢獻：提出了低秩矩陣分解適應方法，大幅減少微調參數數量
   * 核心優勢：
     * 不增加推理延遲，是「無痛漲點」的微調方法
     * 顯著減少顯存使用量（最多減少 2/3）
     * 在 GPT-3 175B 上，訓練過程中的顯存消耗從 1.2TB 降至 350GB
     * 訓練速度比全參數微調提高約 25%
   * 實現方式：假設預訓練權重參數為 W₀，通過低秩矩陣 B 和 A 的乘積更新參數，其中 W = W₀ + BA
2. AdaLoRA: Adaptive Low-Rank Adaptation (時間大約在2023年)
   * 在 LoRA 的基礎上，AdaLoRA 探索了如何自動調整低秩矩陣的秩，以便在不同任務與資源限制下獲得更靈活且高效的微調效果。
   * 這篇論文進一步完善了原始 LoRA 方法，對於追求最佳效能與資源利用率的應用非常重要。
3. **QLoRA: Efficient Finetuning of Quantized LLMs (2023/05/23)**
   * QLoRA 則是將 LoRA 方法延伸到量化（quantization）的場景
   * 讓你在低精度（例如 4 位）的情況下仍能高效微調大型語言模型。
   * 這對於資源受限的環境（例如單張 GPU）來說，極具實用價值，並能顯著降低記憶體與運算需求。
4. DoRA: Decomposed Low-Rank Adaptation
   * 基於 LoRA 的改進方法
   * 核心思想：將預訓練權重分解為幅度（magnitude）和方向（direction）兩個部分，並分別微調
   * 創新點：通過權重分解分析探究 FT (Full Fine-tuning) 和 LoRA 之間的內在差異
   * 優勢：使 LoRA 的學習能力更接近全參數微調（FT）

