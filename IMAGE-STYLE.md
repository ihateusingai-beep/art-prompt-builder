# art-prompt-builder · 圖卡風格 canon（統一出圖）

> **用途**：所有 `who` / `act` / `style` 卡面 JPG、補圖、重產，必須跟呢份。  
> **對象**：SEN 中度智障圖卡（將軍澳培智）· 學生主看圖 + 下方繁中標。  
> **Live**：https://ihateusingai-beep.github.io/art-prompt-builder/  
> **路徑**：`img/*.jpg`（中秋 flat）· `img/christmas|spring-festival|dragon-boat/*.jpg`

---

## 1. 一句 lock

**兒童圖卡插畫 · 單一主體置中 · 柔和粉彩 · 大膽簡單形狀 · 溫暖安全 · 畫面絕對無字。**

唔係吉卜力電影分鏡、唔係寫實棚拍、唔係複雜構圖海報。

---

## 2. 硬規格（必守）

| 項 | 規格 |
|---|---|
| 比例 | **1:1 square** |
| 落盤 | center-crop → **1024×1024 JPEG** · quality ≈ 85 · optimize |
| 主體 | **單一清晰主體**置中；背景乾淨、淺、少雜物 |
| 形狀 | 大塊、圓潤、友善；遠睇 1 秒識得係咩 |
| 色調 | soft pastel / 溫暖；節日可略加主題色，但唔可陰暗 |
| 光 | 柔和散射；禁硬陰影、禁恐怖對比 |
| 文字 | **畫面 0 文字**：無中文、無英文、無字母、無數字、無招牌、無標語、無水印、無 logo、無 caption |
| 標籤 | 繁中寫喺 **HTML `zh:`**，唔燒入圖 |
| color 類 | **只 CSS gradient** — 唔產 color JPG |
| 宗教 | 聖誕：**禁**十字架、教堂彩窗；新年／端午：禁無關西方節日元素 |

### 禁止（negative · 全主題共用核心）

- 恐怖 / 黑暗 / 血腥 / 怪核
- 模糊臉、擁擠微細、雜亂背景
- 浮水印、logo、簽名
- **任何可讀文字／字母／英文／數字／typography**
- 宗教符號（尤其聖誕十字架、stained glass）
- 跨節日元素（例如新年圖入聖誕樹）

節日專屬 negative 以 `holiday-prompt-builder.html` 內 `NEG_*` 為 runtime 準；補圖時同步跟。

---

## 3. Prompt 模板（產卡面 JPG 用）

### 3.1 中文骨架（Grok / MiniMax 皆可）

```
兒童圖卡插畫，單一主體置中，柔和粉彩背景，溫暖安全，大膽簡單形狀：
【主題描述】。
絕對無文字、無字母、無英文、無數字、無招牌、無水印、無 logo。
no text no letters no English no numbers no watermark no logo
```

### 3.2 英文骨架（skill / batch 用）

```
Simple cute children picture-book illustration for SEN education flashcard,
single clear subject centered, soft pastel background, big friendly shapes,
warm safe mood: 【SUBJECT】,
no text, no letters, no English, no numbers, no logos, no watermark, no captions, square
```

### 3.3 主題描述寫法

- **who**：角色 alone 或清晰一組（最多家庭級 2–3 人，仍要主體清楚）
- **act**：一個動作 + 最少道具（例：掛襪子喺壁爐旁；拎紅燈籠）
- **style**：畫風樣本 — 寫「……風格樣張／texture feel」，仍要單一可辨主體或明確風格示意
- 易出事物（揮春、海報、貼紙、包裝、燈籠面）：**額外強調「紙面／表面只有空白或純花紋，無可讀文字」**

### 3.4 好 vs 差

| ✅ | ❌ |
|---|---|
| 「微笑薑餅人，糖霜純裝飾曲線，無字母」 | 「薑餅人身上寫 Merry Christmas」 |
| 「門上紅色揮春紙，只有簡單花紋，無字」 | 「揮春寫福字／英文」 |
| 「紅燈籠，燈籠表面無字」 | 「燈籠印英文 brand」 |
| 單主體 + 淺底 | 大場面多人微細、夜黑恐怖 |

---

## 4. 出圖 pipeline

1. **MiniMax 優先**（`image-01` · `aspect_ratio: 1:1`）
2. 額滿 **1008 / 2056** → 明說「MiniMax 額滿，用 Grok image 頂」→ `image_generate` · `aspect_ratio=square`
3. 原檔通常在 `~/.hermes/cache/images/`
4. PIL：center-crop 最短邊 → resize **1024×1024** → JPEG q85
5. 寫入 `img/<path>`，HTML `img:` 路徑對齊
6. `node --check` 抽 script；commit；push；curl live 圖 **HTTP 200**

批次：Grok ~4 張 parallel OK；MiniMax 間隔 ~2s。

---

## 5. 與 UI 文案分工

| 層 | 規則 |
|---|---|
| **圖像素** | 無字（本文件 §2–3） |
| **卡面 label** | 全 **繁中** 2–3 字；`zh:` **0 拉丁字母**（例：BB→寶寶） |
| **簡/英** | 只在結果 prompt tab（`zhCn` / `en`），唔印上圖卡 |
| **學生 AI 出圖 negative** | HTML `NEG_ZH` / `NEG_CN` / `NEG_EN` + 節日變體 |

---

## 6. 檔名同目錄

```
img/who-*.jpg | act-*.jpg | style-*.jpg          ← 中秋（歷史 flat）
img/christmas/{who,act,style}-*.jpg
img/spring-festival/{who,act,style}-*.jpg
img/dragon-boat/{who,act,style}-*.jpg
```

- 一詞一圖；禁長期 reuse 錯節日圖
- color：**無** JPG

---

## 7. QA checklist（每張／每批）

- [ ] 1:1 · 1024 JPG
- [ ] 主體 1 秒可辨、置中
- [ ] soft pastel / 溫暖，唔陰唔驚
- [ ] **像素內無任何字／字母／水印**
- [ ] 無十字架／錯節日符號
- [ ] 路徑已 wire；live 200
- [ ] 對應 `zh:` 繁中、無拉丁

Vision 不可用時：高風險類（揮春、海報、貼紙、包裝、燈籠面）優先重產 + 加強「空白/花紋無字」prompt。

---

## 8. 一鍵 copy prompt（補圖）

```
兒童圖卡插畫，單一主體置中，柔和粉彩背景，溫暖安全，大膽簡單形狀：【填主題】。畫面絕對無文字無字母無英文無數字無招牌無水印。no text no letters no English no numbers no watermark no logo square
```

---

## 9. 相關

- App：`holiday-prompt-builder.html`（`NEG_*` · THEMES data · `img:`）
- Skill：`sen-festival-prompt-builder` + `minimax-image-generation` → `references/sen-holiday-flashcard-batch.md`
- 擴詞流程：skill `references/vocab-expand-playbook.md`

**改風格 = 先改本 MD，再改 skill ref 同出圖 prompt，最後先補新圖。**
