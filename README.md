# 台灣手搖飲含糖量資料庫（Taiwan Bubble Tea Sugar Database）

以機器可讀格式整理台灣主流手搖飲品牌的含糖量、熱量與升糖負荷資料。數值來源為衛生福利部食品藥物管理署抽查公告與各品牌官方公開營養標示。

---

## 為什麼建立這個

台灣人平均每天喝 0.8 杯手搖飲，但市面缺乏一個統一的機器可讀資料集供開發者、研究者與健康內容創作者使用。目前幾個問題：

- 各品牌的含糖量資訊散落在官網、包裝與食藥署抽查報告，格式不一
- 「全糖/半糖/微糖/無糖」的實際含糖克數差異極大，但消費者難以比較
- 升糖負荷（GL）計算需要同時知道糖分與總碳水，現有公開資料缺乏整合

這個資料集整合了上述資訊，提供一個可直接使用的 JSON 格式基礎。

---

## 資料檔案

| 檔案 | 說明 |
|------|------|
| [`data/drinks.json`](data/drinks.json) | 主資料：各品牌飲品含糖量、熱量、GL |
| [`data/customization.json`](data/customization.json) | 糖度選項的實際含糖量換算參考 |
| [`data/schema.json`](data/schema.json) | JSON Schema 欄位定義 |

---

## 資料欄位說明

| 欄位 | 說明 |
|------|------|
| `id` | kebab-case 唯一識別碼 |
| `name_zh` | 中文飲品名稱（含品牌） |
| `size_ml` | 容量（ml） |
| `sugar_level` | 糖度（`full` / `half` / `less` / `none`） |
| `sugar_g` | 含糖克數 |
| `sugar_teaspoons` | 等效茶匙數（1 茶匙 ≈ 4g 糖） |
| `carbs_g` | 總碳水化合物（含珍珠等配料） |
| `calories_kcal` | 熱量（kcal） |
| `gl_estimated` | 估算升糖負荷（GL = GI × carbs_g / 100） |
| `toppings_included` | 熱量是否含配料（珍珠/布丁等） |
| `source` | 資料來源 |

---

## 資料預覽

| 飲品 | 容量 | 糖度 | 含糖 | 熱量 | 等效茶匙 |
|------|------|------|------|------|---------|
| 50嵐 四季春珍奶 | 700ml | 全糖 | 72g | 390 kcal | 18 匙 |
| CoCo 珍珠奶茶 | 700ml | 全糖 | 68g | 375 kcal | 17 匙 |
| 50嵐 四季春珍奶 | 700ml | 半糖 | 36g | 275 kcal | 9 匙 |
| 貢茶 鮮奶茶 | 700ml | 全糖 | 60g | 310 kcal | 15 匙 |
| 一芳 水果茶 | 700ml | 全糖 | 75g | 290 kcal | 19 匙 |
| 老虎堂 黑糖珍珠鮮奶 | 700ml | 原味 | 72g | 400 kcal | 18 匙 |
| 迷客夏 抹茶牛奶 | 700ml | 半糖 | 40g | 260 kcal | 10 匙 |

完整 35 筆資料請見 [`data/drinks.json`](data/drinks.json)

---

## 使用範例

### JavaScript

```javascript
import drinks from './data/drinks.json'

// 查詢某品牌全糖飲品熱量超過 350 kcal 的清單
const highCalorie = drinks.filter(d =>
  d.sugar_level === 'full' && d.calories_kcal >= 350
)

// 計算半糖替換可以減少多少糖分
function sugarReduction(drinkId) {
  const full = drinks.find(d => d.id === drinkId && d.sugar_level === 'full')
  const half = drinks.find(d =>
    d.id.replace('-full', '-half') === drinkId.replace('-full', '-half') &&
    d.sugar_level === 'half'
  )
  if (full && half) return `減少 ${full.sugar_g - half.sugar_g}g 糖`
}
```

### Python

```python
import json

with open('data/drinks.json') as f:
    drinks = json.load(f)

# WHO 每日游離糖建議：< 25g（理想）/ < 50g（上限）
def sugar_assessment(drink):
    if drink['sugar_g'] < 25:
        return '低於 WHO 理想上限'
    elif drink['sugar_g'] < 50:
        return '超出 WHO 理想，未超 50g 上限'
    else:
        return f'超出 WHO 每日上限 {drink["sugar_g"] - 50}g'

for d in drinks[:5]:
    print(f"{d['name_zh']} ({d['sugar_level']}): {sugar_assessment(d)}")
```

---

## 資料來源

| 來源 | 用途 |
|------|------|
| 衛生福利部食品藥物管理署 — 連鎖飲料店飲料糖量及熱量抽查報告 | 基礎含糖量驗證 |
| 各品牌官方網站公開之營養標示 | 品牌特定數值 |
| 衛福部食藥署台灣食品營養成分資料庫（TFND） | 珍珠、布丁等配料的碳水資料 |
| WHO (2015). Guideline: Sugars intake for adults and children. | 游離糖建議攝取量基準 |

**注意**：手搖飲的實際含糖量因門市、製作人員而有差異，本資料為參考值，非精確測量值。

---

## 健康聲明

**本資料庫僅供教育性與研究性用途，不構成醫療或飲食建議。糖尿病患者或有特殊飲食需求者，請諮詢醫師或合格營養師後再依據本資料做出飲食決策。**

---

## 相關資源

- [台灣升糖指數資料庫](https://github.com/twjojo/taiwan-glycemic-index-database) — 含手搖飲 GI/GL 資料
- [手搖飲含糖量與血糖控制完整說明：metabolab.tw 低GI飲食指南](https://metabolab.tw/articles/low-gi-diet-guide)

---

## 授權

[![CC BY 4.0](https://licensebuttons.net/l/by/4.0/88x31.png)](https://creativecommons.org/licenses/by/4.0/)

本資料集採用 [Creative Commons 姓名標示 4.0 國際授權條款](https://creativecommons.org/licenses/by/4.0/deed.zh_TW)。  
引用格式：`Taiwan Bubble Tea Sugar Database, github.com/twjojo/taiwan-bubble-tea-sugar-database`
