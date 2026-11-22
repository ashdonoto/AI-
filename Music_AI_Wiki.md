# Music AI Wiki

> **主題**：AI 如何理解並生成音樂？  
> **用途**：作為 12 頁 HTML 教學簡報的知識來源與引用索引。  
> **語言**：繁體中文  
> **資料規模**：6 份外部資料（S01–S06）  
> **整理方式**：按「音樂表示 → 音樂序列模型 → 音訊生成 → 管弦樂與結構控制」組織。  
> **資料確認日期**：2026-11-23

## 0. Wiki 使用方式與範圍

本文件把六份來源整理成可供 LLM 檢索的知識卡。各卡包含來源、研究問題、主要方法、關鍵概念、限制及對應的簡報頁數。引用時請使用 **S01–S06**，並優先回到原始文件核對；不要把以下教學示例當成論文實驗結果。

本 Wiki 以來源的公開摘要、官方說明頁及 CAST 的公開文章內容為基礎整理，**不是六篇論文的逐頁全文精讀或重新實驗**。特別是模型細節、數值、評估方式如要延伸，應再查閱論文全文。這六份資料中，S01 是技術規格摘要，其餘五份是研究論文。

### 核心區別

- **Audio（音訊）**：聲波訊號，包含實際音色、演奏與聲學表現。
- **MIDI（音樂控制事件）**：記錄 Note On、Note Off、力度、控制訊息等，並非聲音本身。MIDI 可控制音源，但 MIDI channel 不必然等於獨立樂器聲部。[S01]
- **Symbolic Music Generation（符號式音樂生成）**：產生或處理音符、節奏等結構化音樂事件；Music Transformer、MusicVAE、SymphonyGen 及 CAST 分別涉及此類相關問題。[S02][S03][S05][S06]
- **Audio Music Generation（音訊音樂生成）**：模型直接產生代表音訊的資訊並重建聲音；MusicGen 的輸出屬於此路線，不直接等於可逐音編輯的管弦樂 MIDI 分軌。[S04]
- **Harmony Skeleton（和聲骨架）**：用簡化而有結構的音樂條件引導後續生成；各論文所稱的 skeleton 並非完全相同的表示，不可逕自視為同一種資料格式。[S05][S06]

---

## S01 — MIDI 1.0 Messages：音樂如何被電腦表示？

**資料類型**：官方技術規格摘要（非研究論文）  
**來源**：MIDI Association, *Summary of MIDI 1.0 Messages*  
**原文**：https://midi.org/summary-of-midi-1-0-messages  
**主題標籤**：`MIDI` `Note On` `Note Off` `Velocity` `Channel` `Control Change`

### 主要問題

如何以標準化的訊息，讓電子樂器及軟體傳遞演奏與控制資訊？

### 主要內容

MIDI 1.0 的 Channel Voice Messages 包括 **Note On**、**Note Off**、**Polyphonic Key Pressure**、**Control Change**、**Program Change**、**Channel Pressure** 和 **Pitch Bend Change**。例如 Note On 表示開始按下某個琴鍵，攜帶音符編號及力度；Note Off 表示放開／結束音符。[S01]

MIDI 1.0 使用 16 個邏輯通道（規格訊息的編碼是 0–15，介面通常顯示 1–16）。**通道、音色設定與實際聲部不是一對一的保證關係**：樂器分軌通常還依賴檔案軌道結構、音源配置及編曲方式。

### 教學示例（非來源的實驗數據）

`Note On: pitch 60, velocity 90` 表示開始演奏 MIDI 音符 60（常見十二平均律對應中音 C），力度 90；相對應的 `Note Off: pitch 60` 代表音符結束。MIDI 事件本身不包含真實鋼琴錄音。

### 使用限制與注意

官方頁面明確指出它是**訊息概要而非完整規格**；若需要實作相容性或 MIDI 檔案解析細節，必須進一步參照完整規格。也不要將 MIDI 事件、MIDI 檔案格式及數位音訊混為一談。

**對應簡報**：第 1–4、10、12 頁。

---

## S02 — Music Transformer：長距離音樂結構

**資料類型**：研究論文  
**作者／年份**：Cheng-Zhi Anna Huang et al., 2018  
**論文**：*Music Transformer*  
**原文**：https://arxiv.org/abs/1809.04281  
**主題標籤**：`Transformer` `Relative Attention` `Long-term Structure` `Symbolic Music`

### 研究問題

音樂會在不同時間尺度上重複、變化與呼應：短動機、樂句乃至整個曲式。一般序列模型難以保留長時間的音樂關係，而直接計算相對位置注意力也會產生很高的記憶體成本。[S02]

### 核心方法

研究提出改善的 **relative self-attention（相對注意力）** 計算方式，使中間相對位置資訊的記憶體需求由序列長度的平方量級降低至線性量級。模型因此能處理更長的音樂事件序列。[S02]

### 論文報告的能力

- 生成分鐘級、包含數千步的音樂作品。
- 依據既有音樂動機進行具連貫性的續寫。
- 在 sequence-to-sequence 設定下，依旋律生成伴奏。
- 在 JSB Chorales 和 Piano-e-Competition 資料集上進行評估。[S02]

### 重要辨析

Music Transformer 的重點是**音樂序列與長距離依賴**，並非「輸入一段文字就輸出完整音訊」，也不代表它天然具備樂器分軌正確或實際演奏可行性的保證。

**對應簡報**：第 2、5、6、12 頁。

---

## S03 — MusicVAE：用潛在空間表示音樂

**資料類型**：研究論文  
**作者／年份**：Adam Roberts, Jesse Engel, Colin Raffel, Curtis Hawthorne, Douglas Eck, 2018  
**論文**：*A Hierarchical Latent Vector Model for Learning Long-Term Structure in Music*  
**發表**：ICML 2018, PMLR 80:4364–4373  
**原文**：https://proceedings.mlr.press/v80/roberts18a.html  
**主題標籤**：`VAE` `Latent Space` `Hierarchical Decoder` `Interpolation`

### 研究問題

一般 VAE 在處理長序列時，可能難以學到具有長期結構的有意義潛在表示，並出現 **posterior collapse（後驗塌縮）** 等問題。[S03]

### 核心方法

MusicVAE 使用**階層式解碼器（hierarchical decoder）**：先產生各子序列的 embedding，再利用這些 embedding 生成各段子序列，鼓勵模型實際利用潛在變數中的資訊。[S03]

### 論文報告的能力

在音符序列建模的實驗中，作者報告階層式方法相較於平坦（flat）基準模型，在**取樣、插值和重建**等方面有顯著改善。[S03]

### 教學理解

可以把 MusicVAE 想成將一段音樂編碼到連續的潛在空間，再從其中的點生成或重建音樂；兩個點之間也可進行插值。但「插值後必然形成好聽的過渡」不是保證。

### 與 Music Transformer 的區別

Music Transformer 著重序列中的長距離關聯；MusicVAE 著重利用有意義的**潛在表示**及階層式解碼處理長期結構。它們不是同一模型的不同名稱。

**對應簡報**：第 6、12 頁。

---

## S04 — MusicGen：從文字或旋律條件生成音訊

**資料類型**：研究論文  
**作者／年份**：Jade Copet et al., 2023（arXiv 2024 修訂）  
**論文**：*Simple and Controllable Music Generation*  
**發表**：NeurIPS 2023  
**原文**：https://arxiv.org/abs/2306.05284  
**主題標籤**：`MusicGen` `Text-to-Music` `Audio Tokens` `Conditional Generation`

### 研究問題

如何在給定文字描述或旋律特徵的條件下，有效率地生成具可控性的音樂音訊？[S04]

### 核心方法

MusicGen 使用單階段 Transformer 語言模型，處理多個壓縮離散音樂表示（audio token）串流，透過有效率的 token interleaving（交錯）模式避免必須串接多個階段式生成模型。[S04]

### 輸入與輸出

- **條件輸入**：文字描述或旋律特徵。
- **主要輸出**：音樂音訊，可產生單聲道或立體聲音樂樣本。[S04]

### 常見誤解

**MusicGen 不等於直接產生可逐音編輯的多軌 MIDI。** 即使生成的音訊聽起來像管弦樂，也不表示模型已經產生每種樂器獨立、聲部合理的 MIDI 軌道。若要獲得分軌符號資料，通常還需要其他模型或轉譯處理。

### 與符號式生成的差異

符號式方法重視音符事件、時值、樂器及結構是否可編輯；音訊生成則直接關心聲音成果。兩者的目標與表示方法不同，不能僅用「哪個較進階」比較。

**對應簡報**：第 3、7、11、12 頁。

---

## S05 — SymphonyGen：分層管弦樂生成與 Harmony Skeleton

**資料類型**：研究論文  
**作者／年份**：Xuzheng He et al., 2026  
**論文**：*SymphonyGen: 3D Hierarchical Orchestral Generation with Controllable Harmony Skeleton*  
**狀態**：arXiv 頁面註明已獲 ISMIR 2026 接受  
**原文**：https://arxiv.org/abs/2604.25498  
**主題標籤**：`Orchestral Generation` `Hierarchy` `Harmony Skeleton` `Bar` `Track` `Event`

### 研究問題

交響樂生成不只要處理音符順序，也要同時控制整體形式、密集的多軌配器和多種樂器的音樂紋理。論文將此問題描述為可擴展性與控制能力之間的失衡。[S05]

### 核心方法

SymphonyGen 採用 **3D hierarchical framework（三維階層式架構）**，將生成流程拆成 **bar（小節）、track（軌道）、event（事件）** 三個結構軸，以級聯的解碼器降低相較於平坦 token 串流的記憶體需求，並允許在多個結構層級上加入條件。[S05]

論文以**節拍量化、多音高的和聲骨架（beat-quantized multi-pitch harmony skeleton）**當作簡譜／短譜式的條件。這個骨架可以由使用者撰寫、由音樂分析取得，或由模型生成，用來引導整體和聲輪廓，同時生成較完整的管弦樂紋理。[S05]

研究亦描述了使用 CLaMP 3 音訊嵌入的跨模態聲學獎勵進行強化學習微調，以及在推論時抑制非預期音調衝突的取樣演算法。[S05]

### 研究報告與限制

作者報告在客觀指標上減少不協和情況、保留獨立的旋律指標，並在主觀評價中得到優於比較系統的結果。不過，**這不等於對所有生成結果的樂器分軌、聲部數量、演奏可能性或配器合理性提供保證**；如果要研究這些問題，仍需額外的樂器與聲部層級檢查。

### 教學示例（非論文的固定輸入）

`C → F → G → C` 可以當作「先有和聲大綱，再填上旋律與配器紋理」的簡易例子。但 SymphonyGen 所使用的 skeleton 並非單純只有四個和弦名稱，不能將示意圖誤認為完整技術表示。

**對應簡報**：第 1、2、8、9、10、11、12 頁。

---

## S06 — CAST：先規劃結構，再生成音樂紋理

**資料類型**：研究論文  
**作者／年份**：Yang Yalan, 2026  
**論文**：*Enhancing long-term structure in symbolic music generation via a cascaded Skeleton-to-texture framework*  
**發表**：Scientific Reports 16, Article 17420（2026-04-14）  
**原文**：https://www.nature.com/articles/s41598-026-46750-0  
**主題標籤**：`CAST` `Skeleton-to-texture` `MusicBERT` `MuseFormer` `Long-term Coherence`

### 研究問題

在長篇符號式音樂生成中，純粹依靠隱含的機率關係，可能導致整體曲式或和聲走向失去連貫性，尤其是音樂序列變長之後。[S06]

### 核心方法

CAST 將音樂生成分成**宏觀和聲規劃（macro-harmonic planning）**與**微觀紋理填充（micro-texture filling）**兩個子工作。研究使用 MusicBERT 擷取語意骨架，再透過 cross-attention 讓骨架條件影響 MuseFormer 生成器，以更明確地建模長距離依賴。[S06]

### 論文報告的結果

在作者的實驗設定下，模型生成長度可達 1000 tokens；摘要報告結構誤差由基準的 **0.58 降到 0.22**，和弦生成準確率達 **96%**。這些是該研究的評估結果，**不可推廣成所有音樂生成任務或所有資料集的預期表現**。[S06]

### 與 SymphonyGen 的關係

兩者都凸顯「先有結構條件，再生成細節」的價值，但任務、模型與骨架定義並不完全相同。CAST 更強調符號式音樂的長期結構與紋理生成；SymphonyGen 著重多樂器管弦樂及小節／軌道／事件層級的生成控制。此處為根據兩篇研究的**比較性整理**，不是作者互相驗證的結論。[S05][S06]

**對應簡報**：第 9、10、12 頁。

---

## 7. 六份資料的對照與知識關係

| 來源 | 核心問題 | 主要表示／方法 | 學習重點 |
|---|---|---|---|
| S01 MIDI | 如何表示演奏控制訊息？ | MIDI Channel Voice Messages | 音符事件不等於音訊 |
| S02 Music Transformer | 如何處理音樂的長距離依賴？ | 改良相對注意力的序列模型 | 動機、重複、續寫與結構 |
| S03 MusicVAE | 如何學習可用的音樂潛在表示？ | VAE＋階層式解碼器 | 生成、重建與插值 |
| S04 MusicGen | 如何依文字／旋律生成音樂音訊？ | 壓縮離散音訊 token＋Transformer | 音訊生成不等於 MIDI 分軌 |
| S05 SymphonyGen | 如何生成可控制的多軌管弦樂？ | bar／track／event 分層＋和聲骨架 | 多軌結構與條件控制 |
| S06 CAST | 如何改善長篇符號音樂的結構一致性？ | 骨架規劃＋紋理填充 | 明確結構引導 |

### 建議閱讀順序

**S01 → S02／S03 → S04 → S05／S06**。先掌握資料表示，再學習不同生成方法，最後理解多軌管弦樂與長期結構控制問題。此閱讀路線是本 Wiki 的**教學安排**，不是原始論文提出的技術演化順序。

### Wiki → 12 頁 HTML 簡報對應表

| 頁次 | 教學內容 | 主要來源 |
|---|---|---|
| 01 | 從 MIDI、Transformer 到 AI 管弦樂 | S01、S05 |
| 02 | 課程地圖：表示、學習、配器 | S01、S02、S05 |
| 03 | Audio ≠ MIDI | S01、S04 |
| 04 | MIDI 事件示例 | S01 |
| 05 | AI 如何學習時間結構 | S02 |
| 06 | Music Transformer vs MusicVAE | S02、S03 |
| 07 | MusicGen：文字到音訊 | S04 |
| 08 | 從旋律到多樂器聲部 | S05 |
| 09 | Harmony Skeleton：先有骨架 | S05、S06 |
| 10 | 輸入、條件、生成、驗證 | S01、S05、S06 |
| 11 | 互動測驗：MusicGen 的輸出 | S04、S05 |
| 12 | 總結與延伸 | S01–S06 |

> **教學圖例標註規則**：`C → F → G → C`、波形、Piano Roll、多軌圖以及曲式 `A → B → A′` 主要是自製概念示意，並非引用論文的實際模型輸出或實驗圖。

---

## 8. 給 LLM 的查詢與引用規則

1. 回答前先判斷問題屬於 **表示、長期結構、潛在表示、音訊生成、管弦樂生成、骨架控制** 哪一類。
2. 依上方索引選取來源，產生重點時標出 `[Sxx]`，必要時提供原文 URL。
3. 對不同模型應明確區分 **輸入／輸出**、**訓練或生成方法**、**論文實際報告的結論**。
4. 不得把「音訊」和「可編輯多軌 MIDI」視為同樣的輸出；不得把 MIDI channel 當作確定的樂器聲部。
5. 自製教學例子需標成**示意**，不可冒充論文中的實驗、數據或模型架構圖。
6. 若原始來源未提供某項細節，標為「此份資料未確認」，回到原文查核，不可自行補造。
7. 產生 HTML 簡報後，至少檢查：**每頁是否有來源**、**引用 URL 是否符合標題**、**關鍵敘述是否被來源支持**、**是否混淆音訊與 MIDI**。

## 9. 本次整理與驗證紀錄

- **資料清單**：S01–S06，6 份不同外部資料；其中 1 份官方規格摘要、5 份學術研究。
- **資料確認**：已開啟六個來源的公開網頁，核對各來源標題、摘要或官方技術內容，並記錄原文連結。
- **知識整理**：依教學順序建立六份獨立知識卡，標記來源、關鍵概念、研究限制與簡報頁數。
- **交叉檢查重點**：特別區分 MIDI 與 Audio；Music Transformer／MusicVAE／MusicGen 的不同任務；SymphonyGen 與 CAST 的 skeleton 定義差異。
- **驗證範圍**：以上是來源頁面與摘要層級的整理，**不是**已執行原始模型、重現實驗或完整逐頁審查 PDF。
- **後續檢查**：如要引用模型架構的細部設計、精確評估值或完整技術限制，應直接查閱原始論文全文。

---

## 10. 原始資料連結索引

- **[S01]** MIDI Association — https://midi.org/summary-of-midi-1-0-messages
- **[S02]** Huang et al., *Music Transformer* — https://arxiv.org/abs/1809.04281
- **[S03]** Roberts et al., *A Hierarchical Latent Vector Model for Learning Long-Term Structure in Music* — https://proceedings.mlr.press/v80/roberts18a.html
- **[S04]** Copet et al., *Simple and Controllable Music Generation* — https://arxiv.org/abs/2306.05284
- **[S05]** He et al., *SymphonyGen: 3D Hierarchical Orchestral Generation with Controllable Harmony Skeleton* — https://arxiv.org/abs/2604.25498
- **[S06]** Yang, *Enhancing long-term structure in symbolic music generation via a cascaded Skeleton-to-texture framework* — https://www.nature.com/articles/s41598-026-46750-0
