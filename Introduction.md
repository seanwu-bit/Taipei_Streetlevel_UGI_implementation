# Spatial Decision Support System (SDSS) for Urban Heat & Greening Vulnerability

這是一個專為高密度城市（以台北市為核心研究對象）設計的**微觀街廓尺度空間決策支援系統 (SDSS)**。本專案透過多源資料介接、遙測影像計算、主成分分析 (PCA) 以及整合 Fuzzy-AHP 與 TOPSIS 的決策矩陣，實現從「巨觀熱點篩選」到「微觀街路綠化策略精準派發」的完整科學分析流程。

## 📌 系統工作流整合架構 (Workflow Architecture)

本專案的程式腳本分為兩大核心階段：**第一階段：多源資料處理與特徵萃取**，以及**第二階段：核心空間統計分析與策略派發模型**。
[ 1. 資料處理階段 (Data Preprocessing) ]
├── CWA_AQI介接.ipynb ──────────────────> 收集大氣環境背景變數 (AQI, PM2.5, PM10, SO2)
├── TaipeiTrees_Cleaning... ────────────> 萃取街路網連續體、都市綠地與既有樹木密度
└── EPM_Satellite_images.ipynb ─────────> 計算遙測指標 (LST, NDBI, NDVI/NDVI_inv)
│
▼ (特徵整合)
[ StreetLevel_Integrate.ipynb ]
│
▼ (巨觀篩選)
[ Citywide_PCA.ipynb ]
│
▼ (微觀派發)
[ FUZZYAHP_TOPSIS_Allocationmx.ipynb ]
## 🛠️ 各模組運作流程與說明 (Notebook Descriptions)

### 📂 Phase 1: Data Preprocessing & Feature Extraction (資料處理階段)

#### 1. `CWA_AQI介接.ipynb`
* **功能運作**：透過中央氣象署 (CWA) API 介接大氣環境觀測資料。
* **關鍵輸出**：處理並統計台北市各測站之長周期大氣指標（如 $AQI$、$\text{PM}_{2.5}$、$\text{PM}_{10}$、$\text{SO}_2$ 均值），建立 macro 尺度的大氣環境背景特徵。

#### 2. `TaipeiTrees_Cleaning&OSM_roadnetworkandparks.ipynb`
* **功能運作**：
    * 清洗並網格化台北市既有行道樹點位資料，計算空間鄰近性與微觀樹木密度。
    * 利用 OpenStreetMap (OSM) 撈取高解析度街路網路 (Street-segment network) 與公園綠地邊界，將都市實體基盤解構為連續的微觀網格單元。
* **關鍵輸出**：基於路網單元的既有生態資產圖層（樹木密度、公園可達性特徵）。

#### 3. `EPM_Satellite_images.ipynb`
* **功能運作**：處理地球觀測衛星影像（如 Landsat/Sentinel），進行輻射校正與大氣校正，精準萃取地表實體物理特徵。
* **關鍵輸出**：地表溫度 ($LST$)、不透水層指數 ($NDBI$)、植被指數 ($NDVI$) 及其逆向指標（綠色赤字 $NDVI_{inv}$）的微觀網格圖層。

---

### 📂 Phase 2: Core Analytical Pipeline (核心分析流程)

#### 4. `StreetLevel_Integrate.ipynb`
* **功能運作**：作為「資料過渡與空間對齊樞紐」。本腳本利用空間連接 (Spatial Join) 演算法，將 Phase 1 萃取出的衛星遙測、大氣測站、行道樹資產以及人口統計資料，全面對齊並整合至村里行政單元與其內部的微觀街路網絡中。
* **關鍵輸出**：一份包含 11 個核心環境與社會經濟變數的綜合空間特徵矩陣 (Master Feature Matrix)。

#### 5. `Citywide_PCA.ipynb`
* **功能運作**：
    * 執行全面性的**主成分分析 (PCA)**，探索都市環境脆弱因子的空間耦合效應。
    * **矩陣純化 (Purification)**：辨識出第一版 PCA 中巨觀空污變數產生的尺度錯置（Scale Mismatch），並透過精簡指標排除共線性，純化出真正反映「高溫、硬化、缺乏綠地、人口暴露」的純淨 $PC1$。
* **關鍵輸出**：精確篩選出台北市前 **Top 20% 高熱脆弱度核心熱點村里**，鎖定微觀路網分析的邊界邊界。

#### 6. `FUZZYAHP_TOPSIS_Allocationmx.ipynb`
* **功能運作**：本專案的最核心決策大腦，整合三大演算法：
    1.  **Fuzzy-AHP**：整合專家群體決策，通過一致性檢驗 ($\text{CR} < 0.1$)，確立生態功能導向的權重（樹木密度全域權重達 0.6799）。
    2.  **Opposite TOPSIS**：將權重帶入改造後的反向 TOPSIS 演算法，將 Building Ratio/NDBI 視為效益（環境越差分越高），計算出微觀街道綠化迫切度得分 (Greening Urgency Score)。
    3.  **IF-THEN 決策樹**：建構防呆派發邏輯，依據實體空間限制（建物密度）、既有綠化量等門檻，精準指派五大自適應綠化策略（如立體綠化、水綠共生基礎設施等），徹底避免邏輯衝突。
* **關鍵輸出**：各街段的決策分派統計、Top 10 村里策略結構堆疊圖、以及因地制宜的智慧治理派發圖誌。

---

## 📊 決策變數與權重矩陣摘要 (Variables & Weight Reference)

本系統最終分析所採用之底層 Fuzzy-AHP 全域權重配置如下，直接引導後續路網排序：

| 準則層 (Criteria) | 變數層 (Variables) | 全域權重 (Global Weight) | 內部邏輯一致性 ($\text{CR}$) |
| :--- | :--- | :---: | :---: |
| **固碳量 (0.4293)** | 樹木密度 (Tree Density)<br>固碳效能 (Carbon Sequestration) | **0.6799**<br>**0.0321** | Passed ✅ |
| **微氣候 (0.1842)** | 植被均值 (NDVI_mean)<br>建物密度 (Building Ratio)<br>不透水層 (NDBI_mean) | **0.1065**<br>**0.0656**<br>**0.0599** | Passed ✅ |
| **生物多樣性 (0.3865)**| 物種豐富度 (Shannon Index) | **0.0560** | Passed ✅ |

---
