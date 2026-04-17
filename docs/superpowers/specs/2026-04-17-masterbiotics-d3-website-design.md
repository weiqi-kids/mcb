# MASTERBiotics D3.js 互動網站設計規格

## 概述

將 MASTERBiotics™（日本億菌大師）產品簡報轉化為 6 個互動式一頁式網站，分為 B2B 和 B2C 兩種受眾，各 3 種視覺風格。

## 產出檔案

| 檔案 | 受眾 | 風格 |
|------|------|------|
| `b2c-a.html` | 消費者 | Scrollytelling（滾動敘事） |
| `b2c-b.html` | 消費者 | Dashboard（儀表板） |
| `b2c-c.html` | 消費者 | 產品展示型 |
| `b2b-a.html` | 品牌商/配方師 | Scrollytelling（滾動敘事） |
| `b2b-b.html` | 品牌商/配方師 | Dashboard（儀表板） |
| `b2b-c.html` | 品牌商/配方師 | 產品展示型 |

## 技術規格

- **框架**: 純 HTML + CSS + JavaScript，無前端框架
- **圖表庫**: d3.js v7，透過 CDN 引入
- **架構**: 每個 HTML 檔案完全 self-contained（CSS/JS 全部內嵌）
- **語言**: 全中文
- **響應式**: 支援桌面和手機瀏覽
- **離線**: CDN 載入後即可離線瀏覽

## 視覺設計

### 配色方案（沿用簡報風格）

- 主背景: `#1a1a3e` 深藍紫
- 次背景: `#2d1b69` 紫色漸層
- 強調色: `#d4a843` 金黃色（用於標題、重點數字）
- 文字: `#ffffff` 白色、`#ccccdd` 淺灰
- 輔助色: `#6b5ce7` 紫色、`#3a86c8` 藍色（圖表用）
- 正向/綠: `#4caf50`
- 警示/橘: `#ff9800`

### 字體

- 標題: system-ui, sans-serif, bold
- 內文: system-ui, sans-serif

## B2C 內容架構

### 核心素材

1. **市場痛點**: 未冷藏活菌數快速下降、加工儲存需 25°C、冷鏈成本高
2. **芽孢益生菌優勢**: 活菌數高且穩定、環境耐受性強、標示經得起考驗
3. **五大健康功效**:
   - 腸道健康（耐胃酸膽鹽、定殖力強）
   - 免疫優化（抗病毒、抗過敏、抗發炎三重功效）
   - 體重管理（激活 GLP-1 瘦瘦素）
   - 好心情（腦腸軸多巴胺分泌）
   - 抗氧化（SOD + Catalase 酵素）
4. **臨床實證**:
   - 排便改善：便秘改善 60-70%、腹瀉改善 10-20%
   - 蛋白質吸收提升 150-200%+
5. **產品形式**: 粉劑飲品、口服粉包、膠囊、錠劑、軟糖/果凍

### b2c-a.html — Scrollytelling

全屏垂直 section，IntersectionObserver 觸發進場動畫。

- **Section 1 - Hero**: 品牌名稱 + 標語「超高菌數 + 極致穩定」，背景粒子動畫模擬細菌
- **Section 2 - 痛點**: 三個問題卡片逐一滑入（活菌下降、冷藏條件、冷鏈成本）
- **Section 3 - 解方**: 「芽孢益生菌」大字浮現，三大優勢 icon 逐一亮起
- **Section 4 - 五大功效**: 圓形放射狀佈局，滾動時從中心向外展開 5 個功效節點，每個節點可點擊展開說明
- **Section 5 - 臨床實證**:
  - 左側：排便改善堆疊長條圖（d3.js 動畫從 baseline 到 5th day）
  - 右側：蛋白質吸收數字滾動計數器（200%+）
- **Section 6 - 產品 + CTA**: 產品形式 icon grid，底部行動呼籲

### b2c-b.html — Dashboard

上方 hero 區塊，下方 2x2 或 3 列面板佈局。

- **Hero Banner**: 品牌名 + 核心數字（500 億 + 2000 億）
- **面板 1 - 功效卡片**: 5 張卡片 grid，hover 時翻轉顯示詳細說明
- **面板 2 - 臨床數據**: d3.js 互動圖表，下拉選單可切換蛋白質類型（乳清/碗豆/膠原），顯示糞便氮和 FAN 數據
- **面板 3 - 排便改善**: Before/After 堆疊長條圖（便秘/腹瀉/健康比例），可切換 B.coagulans 或 B.subtilis
- **面板 4 - 產品形式**: Icon grid + 簡短說明

### b2c-c.html — 產品展示型

全屏視差 section，強調視覺衝擊力。

- **Section 1 - 全屏 Hero**: 品牌大字，菌數數字動畫跳動（從 0 跳到 500 億 / 2000 億）
- **Section 2 - 消化道旅程**: SVG path 動畫，模擬孢子從口腔→胃→小腸→大腸的旅程，每到一站顯示說明
- **Section 3 - 功效輪播**: 自動/手動切換的全屏卡片，每張展示一個功效 + 對應 icon
- **Section 4 - 數據亮點**: 大數字區塊（改善 70%、吸收提升 200%+），滾動觸發計數動畫
- **Section 5 - 產品陣列**: 產品形式的水平滾動展示

## B2B 內容架構

### 核心素材

1. **菌株規格**:
   - 凝結芽孢桿菌 DSM 35067: 500 億/克 (5x10^10)，另有 150 億規格
   - 納豆芽孢桿菌 DSM 35068: 2000 億/克 (2x10^11)
   - 經德國 DSMZ 菌庫認證
2. **實驗數據**:
   - 胃酸膽鹽耐受性：6 小時存活率接近 100%
   - 腸道定殖力測試：與傳統益生菌相同，加 BrilliaNest™ 燕窩酸後大幅提升
   - 免疫指標：IFN-γ、IFN-β、TGF-β、IL-4、IFN-γ/IL-4 比值
   - GLP-1 激活：芽孢型態優於生長型態，優於傳統 LA/BL
   - 多巴胺激活：菌株裂解物刺激能力優於無性及孢子型態
   - 抗氧化酵素：Catalase 和 SOD 活性（U/mL），MCB 菌株領先競品
3. **臨床試驗**:
   - 150 人隨機雙盲，9 組（A-I），3 種蛋白質 x 3 種條件
   - Bristol Stool Chart 排便改善數據
   - 糞便氮(TN) + 游離胺基酸氮(FAN) 吸收率數據
4. **市場比較**: MCB vs B.coagulans vs B.subtilis vs Lactobacillus/Bifidobacterium
5. **熱銷配方**: 體重控制、提升免疫、蛋白增肌、情緒穩定、促進消化、降低發炎
6. **公司背景**: 1987 年成立，30+ 年歷史，FSSC 22000/HACCP/ISO 22000/FDA/Halal

### b2b-a.html — Scrollytelling

專業故事線，從市場需求帶到產品解決方案。

- **Section 1 - 市場洞察**: 消費者調查 bar chart（活菌數 vs 功能性 vs 品牌），d3.js 動畫柱狀圖
- **Section 2 - 產品定位**: 「超高菌數 + 極致穩定 = 制霸益生菌市場」，菌株規格雙欄卡片
- **Section 3 - 科學驗證故事線**: 六站式滾動（每站一個實驗），每站包含標題 + d3.js 圖表：
  - 站 1: 胃酸膽鹽耐受（折線圖，6h 存活率）
  - 站 2: 腸道定殖（分組 bar chart，對照組 vs 燕窩酸組）
  - 站 3: 免疫三重功效（多圖組合：IFN-β、IFN-γ、TGF-β、IL-4）
  - 站 4: GLP-1 激活（bar chart，BC-V/BC-S/BS-V/BS-S/LA/BL）
  - 站 5: 多巴胺激活（bar chart）
  - 站 6: 抗氧化酵素（雙圖：Catalase + SOD）
- **Section 4 - 臨床試驗**: 互動面板，可選擇蛋白質類型 + 查看 Bristol / TN / FAN 數據
- **Section 5 - 市場優勢**: 比較表 + 熱銷配方 grid
- **Section 6 - 關於 MCB**: 公司歷史 + 認證 logo 列

### b2b-b.html — Dashboard

左側固定導航，右側內容面板切換。

- **左側 Nav**: 5 個 tab icon + 文字
- **Tab 1 - 菌株規格**: 雙欄卡片（DSM 35067 / DSM 35068），菌數、規格、DSMZ 認證說明
- **Tab 2 - 實驗數據**: 6 個子 tab 切換圖表
  - 2a: 胃酸膽鹽耐受折線圖（0h→1.5h→3h→4.5h→6h）
  - 2b: 腸道定殖 bar chart（4 菌種 x 對照/燕窩酸組）
  - 2c: 免疫指標（4 張 bar chart: IFN-β, IFN-γ, TGF-β, IL-4 + 1 張 ratio chart）
  - 2d: GLP-1 bar chart（6 組）
  - 2e: Dopamine bar chart（6 組）
  - 2f: 抗氧化 Catalase + SOD 雙圖（MCB vs 競品）
- **Tab 3 - 臨床試驗**:
  - 上方：試驗方法摘要
  - 中間：Bristol Stool 堆疊圖（可選 A-I 組）
  - 下方：TN/FAN 數據圖（可選蛋白質類型）
- **Tab 4 - 配方建議**: 6 大方向 grid 卡片，點擊展開配方成分
- **Tab 5 - 公司 + 比較**: 比較表格 + MCB 公司簡介 + 認證

### b2b-c.html — 產品展示型

高端感全屏 section，強調品牌質感。

- **Section 1 - Hero**: MASTER Biotics™ 品牌 + MCB logo，tagline「For Food & Nutrition, we always do more than expected」
- **Section 2 - 菌株卡片**: 兩張全屏卡片（hover 翻轉顯示規格細節），DSM 35067 + DSM 35068
- **Section 3 - 數據亮點**: 大數字計數器區塊（500 億、2000 億、100% 存活率、200%+ 吸收提升、70% 排便改善）
- **Section 4 - 實驗圖表輪播**: 左右箭頭切換 6 張 d3.js 圖表，每張附簡要結論
- **Section 5 - 比較優勢**: 動態比較表（hover 行高亮）
- **Section 6 - 配方 grid**: 6 大方向卡片
- **Section 7 - MCB**: 公司背景 + 1987 年歷史 timeline + 認證 logo 列

## D3.js 圖表清單

### 共用圖表元件

| 圖表 ID | 類型 | 用途 | 數據來源 |
|--------|------|------|---------|
| chart-acid-bile | 折線圖 | 胃酸膽鹽 6h 耐受性 | PDF p.7 |
| chart-colonization | 分組 bar chart | 腸道定殖力（對照 vs 燕窩酸） | PDF p.9 |
| chart-ifn-beta | Bar chart | IFN-β 抗發炎 | PDF p.11 |
| chart-ifn-gamma | Bar chart | IFN-γ 抗病毒 | PDF p.11 |
| chart-tgf-beta | Bar chart | TGF-β 抗發炎 | PDF p.11 |
| chart-il4 | Bar chart | IL-4 抗過敏 | PDF p.11 |
| chart-ifn-il4-ratio | Bar chart | IFN-γ/IL-4 比值 | PDF p.11 |
| chart-glp1 | Bar chart | GLP-1 激活 | PDF p.13 |
| chart-dopamine | Bar chart | Dopamine 激活 | PDF p.15 |
| chart-catalase | Bar chart | Catalase 酵素活性 | PDF p.18 |
| chart-sod | Bar chart | SOD 酵素活性 | PDF p.18 |
| chart-consumer-survey | Bar chart | 消費者關注特點 | PDF p.3 |
| chart-bristol-whey | 堆疊 bar | 乳清蛋白排便改善 | PDF p.26 |
| chart-bristol-pea | 堆疊 bar | 碗豆蛋白排便改善 | PDF p.27 |
| chart-bristol-collagen | 堆疊 bar | 膠原蛋白排便改善 | PDF p.28 |
| chart-tn-fan-whey | 分組 bar | 乳清蛋白 TN/FAN | PDF p.29 |
| chart-tn-fan-pea | 分組 bar | 碗豆蛋白 TN/FAN | PDF p.30 |
| chart-tn-fan-collagen | 分組 bar | 膠原蛋白 TN/FAN | PDF p.31 |
| chart-comparison | 表格 | 市場原料比較 | PDF p.35 |

### 互動行為

- **動畫進場**: IntersectionObserver 觸發，d3 transition duration 800ms
- **Hover**: tooltip 顯示精確數值
- **點擊切換**: 部分圖表可切換數據組
- **數字計數器**: requestAnimationFrame 驅動的數字滾動動畫

## 數據硬編碼

所有數據直接寫在 JavaScript 中，不需外部 JSON 檔案：

### 胃酸膽鹽耐受（chart-acid-bile）
```javascript
const acidBileData = [
  { time: "0h", bs: 100, bc: 100 },
  { time: "1.5h", bs: 101, bc: 100 },
  { time: "3h", bs: 102, bc: 100 },
  { time: "4.5h", bs: 103, bc: 101 },
  { time: "6h", bs: 104, bc: 102 }
];
// bs = 納豆芽孢桿菌 DSM35068
// bc = 凝結芽孢桿菌 DSM35067
```

### 腸道定殖（chart-colonization）
```javascript
const colonizationData = [
  { strain: "凝結芽孢桿菌\nDSM35067", control: 5, withBrilliaNest: 18 },
  { strain: "納豆芽孢桿菌\nDSM35068", control: 7.5, withBrilliaNest: 21 },
  { strain: "嗜酸乳桿菌", control: 5, withBrilliaNest: 14 },
  { strain: "雷特式B菌", control: 6, withBrilliaNest: 17 }
];
```

### 消費者調查（chart-consumer-survey）
```javascript
const surveyData = [
  { factor: "活菌數", value: 70 },
  { factor: "功能性", value: 15 },
  { factor: "品牌", value: 10 }
];
```

### GLP-1（chart-glp1）
```javascript
const glp1Data = [
  { group: "Control", value: 100 },
  { group: "BC-V", value: 480 },
  { group: "BC-S", value: 450 },
  { group: "BS-V", value: 500 },
  { group: "BS-S", value: 470 },
  { group: "LA", value: 250 },
  { group: "BL", value: 240 }
];
```

### Dopamine（chart-dopamine）
```javascript
const dopamineData = [
  { group: "Control", value: 20 },
  { group: "BC-V", value: 130 },
  { group: "BC-S", value: 130 },
  { group: "BC-L", value: 230 },
  { group: "BS-V", value: 130 },
  { group: "BS-S", value: 130 },
  { group: "BS-L", value: 230 }
];
```

### 免疫指標（chart-immune）
```javascript
const immuneData = {
  ifnBeta: [
    { group: "Control", value: 130 },
    { group: "BC", value: 140 },
    { group: "BS", value: 135 },
    { group: "LA", value: 130 },
    { group: "BL", value: 125 }
  ],
  ifnGamma: [
    { group: "Control", value: 100 },
    { group: "BC", value: 500 },
    { group: "BS", value: 480 },
    { group: "LA", value: 450 },
    { group: "BL", value: 420 }
  ],
  tgfBeta: [
    { group: "Control", value: 100 },
    { group: "Pathogen", value: 120 },
    { group: "BC", value: 200 },
    { group: "BS", value: 190 },
    { group: "LA", value: 160 },
    { group: "BL", value: 150 }
  ],
  il4: [
    { group: "Control", value: 100 },
    { group: "Allergen", value: 210 },
    { group: "BC", value: 120 },
    { group: "BS", value: 110 },
    { group: "LA", value: 150 },
    { group: "BL", value: 140 }
  ],
  ifnIl4Ratio: [
    { group: "Control", value: 100 },
    { group: "BC", value: 200 },
    { group: "BS", value: 190 },
    { group: "LA", value: 100 },
    { group: "BL", value: 80 }
  ]
};
```

### 抗氧化酵素（chart-antioxidant）
```javascript
const antioxidantData = {
  catalase: [
    { strain: "日本億菌大師\nDSM35067", value: 8000 },
    { strain: "它牌凝結芽\n孢菌S", value: 3500 },
    { strain: "它牌凝結芽\n孢菌B", value: 14000 },
    { strain: "日本億菌大師\nDSM35068", value: 9500 },
    { strain: "它牌納豆芽\n孢菌H", value: 7000 },
    { strain: "它牌納豆芽\n孢菌S", value: 7000 }
  ],
  sod: [
    { strain: "日本億菌大師\nDSM35067", value: 800 },
    { strain: "它牌凝結芽\n孢菌S", value: 200 },
    { strain: "它牌凝結芽\n孢菌B", value: 350 },
    { strain: "日本億菌大師\nDSM35068", value: 650 },
    { strain: "它牌納豆芽\n孢菌H", value: 550 },
    { strain: "它牌納豆芽\n孢菌S", value: 550 }
  ]
};
```

### 臨床試驗 Bristol（堆疊 bar chart）
```javascript
// 乳清蛋白組
const bristolWhey = {
  control: [ // A 組
    { day: "Baseline", constipation: 20, diarrhea: 20, healthy: 60 },
    { day: "1st", constipation: 20, diarrhea: 20, healthy: 60 },
    { day: "2nd", constipation: 15, diarrhea: 20, healthy: 65 },
    { day: "3rd", constipation: 15, diarrhea: 15, healthy: 70 },
    { day: "4th", constipation: 15, diarrhea: 15, healthy: 70 },
    { day: "5th", constipation: 15, diarrhea: 15, healthy: 70 }
  ],
  bCoagulans: [ // B 組
    { day: "Baseline", constipation: 25, diarrhea: 15, healthy: 60 },
    { day: "1st", constipation: 15, diarrhea: 15, healthy: 70 },
    { day: "2nd", constipation: 10, diarrhea: 10, healthy: 80 },
    { day: "3rd", constipation: 8, diarrhea: 7, healthy: 85 },
    { day: "4th", constipation: 7, diarrhea: 5, healthy: 88 },
    { day: "5th", constipation: 7, diarrhea: 5, healthy: 88 }
  ],
  bSubtilis: [ // C 組
    { day: "Baseline", constipation: 25, diarrhea: 15, healthy: 60 },
    { day: "1st", constipation: 15, diarrhea: 12, healthy: 73 },
    { day: "2nd", constipation: 10, diarrhea: 10, healthy: 80 },
    { day: "3rd", constipation: 8, diarrhea: 7, healthy: 85 },
    { day: "4th", constipation: 7, diarrhea: 5, healthy: 88 },
    { day: "5th", constipation: 7, diarrhea: 5, healthy: 88 }
  ]
};
```

### 糞便氮 TN/FAN
```javascript
const tnFanWhey = {
  control: { tn: 10.23, fan: 0.65, ratio: 0.06 },
  bCoagulans: { tn: 5.98, fan: 1.20, ratio: 0.20 },
  bSubtilis: { tn: 5.91, fan: 1.28, ratio: 0.22 }
};
const tnFanPea = {
  control: { tn: 11.01, fan: 0.59, ratio: 0.05 },
  bCoagulans: { tn: 6.19, fan: 1.22, ratio: 0.20 },
  bSubtilis: { tn: 6.24, fan: 1.38, ratio: 0.22 }
};
const tnFanCollagen = {
  control: { tn: 7.45, fan: 0.80, ratio: 0.10 },
  bCoagulans: { tn: 5.90, fan: 1.32, ratio: 0.22 },
  bSubtilis: { tn: 5.93, fan: 1.34, ratio: 0.23 }
};
```

### 蛋白質排出量（糞氮轉化）
```javascript
const proteinExcretion = {
  baseline: { whey: 15.30, pea: 15.41, collagen: 15.45 },
  whey: { none: 38.36, bCoagulans: 22.43, bSubtilis: 22.16, intake: 50, excreted: 23.06 },
  pea: { none: 41.29, bCoagulans: 23.21, bSubtilis: 23.40, intake: 50, excreted: 25.88 },
  collagen: { none: 27.94, bCoagulans: 22.13, bSubtilis: 22.24, intake: 50, excreted: 12.49 }
};
```

## 比較表數據

```javascript
const comparisonTable = [
  {
    feature: "原料規格",
    masterBiotics: "B.Coagulans 500億\nB.Subtilis 2000億",
    bCoagulans: "150億",
    bSubtilis: "1000億",
    lactoBifido: "100-200億"
  },
  {
    feature: "儲存條件",
    masterBiotics: "無需冷藏與冷鏈運輸",
    bCoagulans: "無需冷藏與冷鏈運輸",
    bSubtilis: "無需冷藏與冷鏈運輸",
    lactoBifido: "儲存溫度不能超過28°C"
  },
  {
    feature: "應用範圍",
    masterBiotics: "保健食品(含錠劑)\n烘焙及糖果食品",
    bCoagulans: "健康補充劑(含錠劑)\n烘焙及糖果食品",
    bSubtilis: "健康補充劑(含錠劑)\n烘焙及糖果食品",
    lactoBifido: "膠囊和粉劑保健品"
  },
  {
    feature: "健康功能",
    masterBiotics: "腸道、免疫、體重管理\n和認知健康多重功能",
    bCoagulans: "視供應商而定",
    bSubtilis: "視供應商而定",
    lactoBifido: "視供應商而定"
  }
];
```

## 熱銷配方數據

```javascript
const formulaData = [
  { name: "體重控制", ingredients: "羥基檸檬酸、生物類黃酮、EGCG、MCT油、白藜蘆醇、綠原酸" },
  { name: "提升免疫", ingredients: "多醣體、薑黃素、維生素C&D、母乳寡糖、蜂膠" },
  { name: "蛋白增肌", ingredients: "乳清蛋白、素食蛋白、奶昔代餐、特膳食品" },
  { name: "情緒穩定", ingredients: "GABA、蘑菇多醣體、迷迭香酸、菸胺酸" },
  { name: "促進消化", ingredients: "加州梅、纖維、油粉、益生元" },
  { name: "降低發炎", ingredients: "薑黃素、麩胱甘肽、維生素C、多酚" }
];
```
