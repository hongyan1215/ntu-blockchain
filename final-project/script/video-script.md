# Demo Video Script — Perp DEX 清算機制比較

**目標長度：** 8–9 分鐘
**形式：** Slidev presenter 模式 + 螢幕錄影 (OBS / QuickTime)
**畫面比例：** 16:9, 1080p
**聲音：** 單軌口白；不放背景音樂
**語言：** 繁體中文口白（投影片內容為英文）

每個區塊格式：`[時間] · 投影片編號 · 口白 · B-roll / 畫面提示`

---

## 00:00–00:25 · Cold open（先不切投影片）

> 「2025 年 3 月 26 日，一名攻擊者『刻意』把自己清算掉，從 Hyperliquid 拿走了數千萬美元。協議的自動安全網——HLP 金庫——完全照著設計運作，而這正是它賠錢的原因。今天我們要看的就是：三個最大的去中心化永續合約交易所，是怎麼用三種完全不同的方式，回答同一個問題——『當交易者付不出錢時，該由誰買單？』」

**B-roll：** JELLY 公開的價格走勢截圖，淡入到 Title slide。

---

## 00:25–00:45 · Slide 1 — Title

> 「我是黃泓諺，學號 R14922156。這支影片會帶大家走過 GMX、dYdX v4 與 Hyperliquid 三條鏈的清算引擎、各自做出的權衡，並從 JELLY 事件回頭檢視整個設計空間。」

---

## 00:45–01:05 · Slide 2 — Why this matters

> 「永續合約 DEX 每天結算的名目交易量高達數百億美元，而清算引擎是其中最安全攸關的元件。設計者面對的是一個三角難題：使用者體驗、系統安全、去中心化——三個角，每家平台只能挑一個方向偏靠。」

**畫面提示：** 點一下 reveal 底部的 callout 條。

---

## 01:05–01:20 · Slide 3 — Vocabulary

> 「先過一遍詞彙：觸發清算的是 mark price，不是最後成交價；initial margin 是開倉門檻、maintenance margin 是維持門檻；liquidator 是被付錢來清掉壞倉的角色；insurance fund 吸收損失；ADL（auto-deleveraging）則是最後手段——強制平掉獲利方來補壞帳。」

**節奏：** 快速念過，這頁只是工具書。

---

## 01:20–01:30 · Slide 4 — Part 1 section card

> 「先進入第一部分：三條鏈的架構地景。」

---

## 01:30–02:00 · Slide 5 — Three architectures

> 「三張圖、三種哲學。GMX 沒有訂單簿——交易者直接與被動流動性池對作，Chainlink 提供價格，keeper bot 是執行肌肉。dYdX v4 把完整 CLOB 塞進 Cosmos 鏈，驗證者『就是』撮合引擎也是預言機。Hyperliquid 兩者兼具——正常流量走 CLOB，再加一個叫 HLP 的社群金庫，同時扮演做市商與保險基金。」

**B-roll：** 依序指向三個 subgraph。

---

## 02:00–02:20 · Slide 6 — Cheat-sheet

> 「同樣資訊整理成表。請特別注意 counterparty 欄位：GMX 的對手方是池子裡的 LP；dYdX 是訂單簿另一側加上保險基金；Hyperliquid 則是訂單簿『加上』HLP。」

---

## 02:20–02:45 · Slide 7 — Leverage and margin

> 「槓桿反映的是風險胃口。GMX 開到 100×，dYdX 每個市場上限 50×，Hyperliquid 40×——但有個關鍵巧思：當帳戶總持倉變大時，最大槓桿會自動降低。BTC 從 40× 起跳，名目超過 1.5 億美元後降到 20×。激進 UX 中藏著一條對抗鯨魚連環爆倉的結構性防線。」

---

## 02:45–02:55 · Slide 8 — Part 2 section card

> 「第二部分：把三條鏈的清算瀑布攤開來比。」

---

## 02:55–03:30 · Slide 9 — Liquidation waterfall

> 「這是整支影片的核心。當帳戶撐不住時會發生什麼？GMX 把單子交給 keeper，直接打進池子，只有當未實現獲利會掏空池子時，才會 ADL 獲利方。dYdX 把清算單寫到公開訂單簿上，價格是 oracle 加上一個可被吃掉的點差；ADL 之前還有保險基金當緩衝。Hyperliquid 分三階：先 20% 走市價，剩下批發給 HLP，連 HLP 都吞不下才會啟動 ADL。」

**畫面提示：** 注意底部紅色 callout：『Tier 1 → 30 s 冷卻 → Tier 2 HLP → Tier 3 ADL』正是 JELLY 事件踩到的那條路徑。

---

## 03:30–04:00 · Slide 10 — Liquidation price formulas

> 「來看數學。GMX 永遠用對交易者不利那一側的 oracle 點差成交，保護池子。dYdX 的可成交價公式裡，特別注意這個 s——s 等於正 1 是多單清算，成交價會低於 oracle；s 等於負 1 是空單清算，成交價會高於 oracle，方向永遠不利於被清算者。Hyperliquid 用 margin available 與 tier MM leverage 推回一個清算價，雖然沒有顯式手續費，但只要 HLP 接走你的倉，你的維持保證金緩衝就歸零了。」

**畫面提示：** 用游標圈出 dYdX 公式裡的 `s` 與右側的 sign convention 註腳。

---

## 04:00–04:25 · Slide 11 — Oracles

> 「Oracle 是永續合約最大的攻擊面。GMX 信任 Chainlink Data Streams 的 operator network。dYdX 把預言機職責綁進驗證者集合，透過 vote extensions 投票回報交易所價格。Hyperliquid 同樣信任驗證者，但額外把 CEX 流動性權重納入計算——Binance 算三倍、OKX 兩倍——作為 sybil-resistance 的啟發式。共同主題是：三家都拒絕用內部薄 AMM 當 oracle，這是 2021–2022 那波 oracle 攻擊的血淚教訓。」

---

## 04:25–04:45 · Slide 12 — Volatility defenses

> 「極端波動的工具差異很大。GMX 用 price-impact caps 與 `MAX_PNL_FACTOR` 限制集體獲利上限。dYdX 沒有顯式熔斷，靠社群保險基金。Hyperliquid 則用 open-interest caps，加上『掛單距離 oracle 不得超過 1%』的反 spoofing 規則。」

---

## 04:45–05:00 · Slide 13 — Funding rates

> 「資金費率簡述：GMX 依未平倉失衡動態調整、連續取樣；dYdX 用 premium + interest 模型，每分鐘取樣、每小時結算；Hyperliquid 每 5 秒取樣 premium，每小時付一次，並有 4%/h 上限。注意一個細節：Hyperliquid 的 funding 是基於 oracle price 計算，不是 mark price，這是刻意設計的反操縱選擇。」

---

## 05:00–05:10 · Slide 14 — Part 3 section card

> 「第三部分：放大鏡看 GMX。我選 GMX 是因為它的『池子當對手方』模型，能最清楚講出 LP 究竟承擔了什麼。」

---

## 05:10–05:55 · Slide 15 — GMX flow diagram

> 「Keeper 持續監控帳戶，當擔保品跌破 0.25–1% 維持門檻，keeper 抓一份簽名過的 Chainlink Data Streams 報告，送出清算交易。注意琥珀色 callout 標出的路徑：commit-execute 兩階段執行——oracle 價格在第 N 區塊先 commit，倉位在 N+1 區塊才執行，這是反 MEV 的關鍵。倉位用對交易者不利那一側的 oracle 點差成交，扣 0.2–0.45% 名目作為手續費，手續費 63% 回流 GM 池 LP、37% 進協議金庫。最後若所有開倉的未實現獲利突破 `MAX_PNL_FACTOR_FOR_ADL`，GMX 會強制平掉獲利最多的倉位——這就是合成市場最常用到的 ADL 安全閥。」

**B-roll：** 用游標沿著琥珀色 callout 描繪整條路徑。

---

## 05:55–06:25 · Slide 16 — Worked example

> 「換成具體數字：100 萬美元、10× 槓桿做多 ETH，價格跌 9% 到 2,730。交易者 PnL 是負 9 萬，清算手續費 2,000 美元。池子拿到 9 萬 WETH 加上 1,260 美元手續費分潤。交易者剩約 8,000 美元離場。重點是：GM 池在結構上是『空 vol』——它從清算事件賺錢，卻在長期單邊行情中被交易者吃走。可以把它想成自動寫選擇權的策略。」

---

## 06:25–06:40 · Slide 17 — v1 → v2 evolution

> 「GMX v2 把每個市場隔離成獨立的池，部分原因就是 v1 的共享 GLP basket 在 2022 年被人用 AVAX 薄現貨操縱過。模式很一致：每一次重大升級背後都對應一個具體事件。」

---

## 06:40–06:50 · Slide 18 — Part 4 section card

> 「第四部分：從戰場學到的教訓。」

---

## 06:50–07:05 · Slide 19 — Real incidents → real design changes

> 「Black Thursday 教會我們需要 backstop bidder；LUNA 教會我們資產上限；FTX 驗證了非託管的論點；而 JELLY，在 2025 年 3 月，揭露了自動化保險金庫的極限。」

---

## 07:05–07:30 · Slide 20 — JELLY case

> 「JELLY 值得仔細看，因為它是目前最新、最完整的一課。攻擊者透過自成交把一個低市值代幣推到可清算狀態，HLP 完全照著設計接下倉位，但底層現貨太薄，HLP 根本沒辦法平倉，於是金庫吃下虧損。Hyperliquid 最後下架 JELLY 並補償用戶，但那必須透過『治理介入』。系統性的教訓是：任何保險原語都需要『分資產』的風險預算，而不是只追蹤整體償付能力。」

---

## 07:30–08:10 · Slide 21 — JELLY 量化攻擊模型

> 「這一頁把報告裡的硬核推導搬上來。左邊是微結構不等式：當 `l·c > L_exit` 時訂單簿吃不下，同時 `l·c ≲ V` 時 HLP 又剛好富到可以全吞——這兩個條件同時成立，攻擊才成立。BTC 上 `L_exit ≫ V`，所以不可能；新上市的薄幣種兩個不等號剛好翻轉，HLP 就成了傾倒對象。右邊是雙帳戶模型的期望提取：`E[extraction] ≈ min(N·(p1−p0)/p0, V) − c`。把 JELLY 的數字代進去——N 約 7,000 萬、價格 4 倍跳、V 約 2.4 億、成本 c 約 350 萬——得到大約 2.06 億美元的理論上限。獲利方地址 A2、A3⋯ 在 HLP 還來不及反向平倉之前，就把乾淨的 USDC 提走了，時間落差就是這場攻擊的『租金』。」

**B-roll：** 框出 `cases` 不等式，再框出 `$206M` 的方框。

---

## 08:10–08:20 · Slide 22 — Part 5 section card

> 「最後一部分：未解問題與未來方向。」

---

## 08:20–08:40 · Slide 23 — Open risks

> 「八個共通風險：oracle 操縱、insurance fund 容量、ADL 信號模糊、跨鏈 wormhole、治理捕獲、MEV 從 keeper 套利擴散、跨保證金的隱性相關性、以及資金費率引發的連環爆倉。前六項是大家熟悉的；後兩項過去討論不夠，下一頁我們專門講一個。」

---

## 08:40–09:10 · Slide 24 — Cross-margin: the hidden correlation tax

> 「跨保證金的隱形稅。維持保證金的精確算法要走 `MM_req = Σ wi·MMi + 相關項`，組合風險的方差是 `Var(E) = ΣΣ wi·wj·σi·σj·ρij`。在 ρ=1 的尾部行情下，方差會用『平方和』而非『單獨加總』的方式爆炸。複雜度更殘酷：朴素演算法 O(n²)，加上 cross-asset solvency check 變 O(n³)，n 一旦上千鏈上根本算不動。表格中綠色那一格才是工程出路——把資產分成 K 個 correlation group（K 遠小於 n），複雜度可以壓到 O(n + K)，這正是下一代風控引擎正在走的方向。」

**B-roll：** 圈出表格綠色那一列。

---

## 09:10–09:40 · Slide 25 — Future directions

> 「往前看六個方向，每一個我都標了紅色的工程代價：ZK 預言機可以降低單一信任，代價是 prover 延遲 200 ms 到 2 秒；意圖式清算拍賣把 MEV 還給用戶，代價是拍賣視窗 τ 越長、壞帳成長 ΔD 約等於 |S|·v·τ；RFQ 報價回退會增加攻擊面；分資產保險 tranche 會切碎資本效率；形式化驗證會降低開發速度；可程式化熔斷則要面對波動偵測器的治理難題。設計者要做的是把這些原語混搭，去『打破』UX 與安全的權衡——不是只在三角形裡繞圈。」

**B-roll：** 強調每一條紅色 `· cost:` 註腳。

---

## 09:40–09:55 · Slide 26 — Conclusion

> 「三個平台、同一個問題、三種答案：GMX 信任 LP、dYdX 信任驗證者加保險基金、Hyperliquid 信任社群金庫。沒有任何一種設計能完勝其他兩種——而正因如此，這個設計空間到今天還是開放的。」

---

## 09:55–10:00 · Slide 27 — Thank you

> 「以上是我的報告，謝謝大家。」

**B-roll：** 最後一頁停 2 秒，淡出黑屏。

---

## Recording checklist

- [ ] Slidev presenter mode：`pnpm slidev --presenter slides.md`
- [ ] Mermaid 圖先 render 成 PNG（`mmdc -i ... -o ... -w 1800 -H 1400`）
- [ ] OBS 音訊：48 kHz 單聲道、−12 dB peak
- [ ] 只擷取 slide 視窗，關閉通知
- [ ] 每個區塊乾淨一鏡到底，剪接時再切
- [ ] 上傳 YouTube 設定為 Unlisted，連結貼到繳交頁

## Post-production

- 剪掉口頭禪、把超過 0.8 秒的停頓收緊
- 輕度 EQ：100 Hz high-pass、3 kHz 微抬升提升人聲清晰度
- 字幕：自動產生後手動修正 ticker（HLP、GLV、JELLY 等）
- 縮圖：JELLY 走勢圖 + 標題文字
