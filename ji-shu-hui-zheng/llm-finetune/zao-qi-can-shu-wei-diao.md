# 早期參數微調

### 早期參數微調方法

1. **Adapter (2019.02.02)**
   * 論文：《Adapter: Parameter-Efficient Transfer Learning for NLP》
   * 價值：作為最早的參數高效微調方法之一，Adapter 為後續研究奠定了基礎
   * 局限性：雖然新增參數少，但實驗顯示即使只有 1% 的新參數仍會顯著增加推理延遲
2. **Prefix-Tuning (2021.01.01)**
   * 論文：《Prefix-Tuning: Optimizing Continuous Prompts for Generation》
   * 重要性：引入了連續提示的概念，影響了後續大量工作
   * 缺點：前綴相對難以優化，且準確度難以隨前綴長度增長
3. **P-tuning (2021.03.18)**
   * 論文：《P-tuning: GPT Understands, Too》
   * 專注於讓 GPT 類模型理解更多任務類型
4. **Prompt Tuning (2021.04.18)**
   * 論文：《The Power of Scale for Parameter-Efficient Prompt Tuning》
   * 探討了參數規模對提示調整效果的影響
