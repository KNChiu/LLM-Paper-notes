# Phi-3 / Phi-3-vision

## Phi-3 系列模型介紹與更新

### 1. 劃時代的性能與小型化

*   Phi-3 小型語言模型（SLMs）使用高質量數據訓練

    * 經過大量高質量數據訓練，包括人類反饋強化學習（RLHF）和自動化測試。
    * Phi-3 系列模型適用於各種語言理解、推理分析和生成任務。
    * 在資源有限和低延遲需求的場景中表現突出，甚至部署於手機上(Phi-3-mini 4-bit)。

    <figure><img src="../../.gitbook/assets/image (19).png" alt="" width="563"><figcaption></figcaption></figure>


*   Phi-3 性能優於同等和更大尺寸模型

    * 支持 4K /8K 和 128K 的上下文長度，適用於不同場景需求。
    * 在語言、推理、程式碼和數學等基準性能優於同等和更大尺寸模型，如 GPT-3.5T 和 Gemini 1.0 Pro。

    <figure><img src="../../.gitbook/assets/image (20).png" alt="" width="563"><figcaption></figcaption></figure>

### 2. 安全性評估

* 遵循微軟負責任 AI 標準，包含六項原則：
  * 問責制、透明度、公平性、可靠性和安全性、隱私和安全性、包容性。
* 經過嚴格安全測量和評估，包括：
  * 紅隊測試(red-teaming)、敏感用途審查(sensitive use review)。
* 使用高質量數據訓練
  * 經過強化學習和多種危害類別的自動化測試和人工紅隊測試。

### 3. 優化與支援性

* 使用 ONNX Runtime 優化模型
  * 支持 Windows DirectML 和跨平台支援。
  * 針對 NVIDIA GPU 和 Intel accelerators 進行優化。

### 4. 多模態的引入

*   Phi-3 系列新增 Phi-3-vision 模型

    * 結合語言和視覺能力，對於文本和圖像進行推理。
    * 在視覺推理任務、OCR、表格和圖表理解任務中性能超過更大尺寸模型。

    <figure><img src="../../.gitbook/assets/image (21).png" alt="" width="563"><figcaption></figcaption></figure>

### 5. 選擇合適的模型

* **Phi-3-mini**  (3.8B) : 支援 128K, 4K 兩種上下文長度(context lengths)。
*   **Phi-3-small** (7B)  : 在語言、推理、程式碼和數學測試中擊敗 GPT-3.5。

    <figure><img src="../../.gitbook/assets/image (22).png" alt="" width="563"><figcaption></figcaption></figure>
*   **Phi-3-medium** (14B) : 各項表現優於Gemini 1.0 Pro。

    <figure><img src="../../.gitbook/assets/image (23).png" alt="" width="563"><figcaption></figcaption></figure>
*   **Phi-3-vision** (4.2B) : 適合需要圖文結合推理的任務，如 OCR 和圖表理解，優於 Claude-3 Haiku 和 Gemini 1.0 Pro V 等較大模型。

    <figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>



    ### 筆記重點

    * Phi-3 系列模型強調高性能、小型化和成本效益。
    * 遵循微軟負責任 AI 標準，強調安全和責任心。
    * Phi-3-vision 引入多模態能力，擴展應用場景。
    * 根據不同需求選擇適當 Phi-3 模型，以實現最佳性能和成本平衡。
