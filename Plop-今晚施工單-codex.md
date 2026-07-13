# Plop｜第一批施工規格（Codex Implementation Spec）

## 開發原則

本次目標：

* 新增 Style
* 調整既有 Style 參數
* 修正疊色邏輯

不進行架構重構。

請遵守：

1. 以 **Minimal Change（最小修改）** 為原則。
2. 優先沿用現有程式架構。
3. 不重新命名既有變數、函式、檔案。
4. 不拆分檔案，除非完成需求時為必要條件。
5. 若發現架構問題，請完成本批需求後，在最後列出建議，不直接重構。

---

# 一、命名規則

本文件與程式碼統一使用：

* `Style`
* `StylePack`

避免新增以下平行概念：

* Skin
* Theme
* Material

除非未來進行架構重構。

本批所有新增視覺效果皆應作為新的 `StylePack` 定義。

---

# 二、修改項目

## 1. 疊色改為 Multiply

### 現況

目前疊色使用 Alpha 半透明。

問題：

* 重疊後只會變暗。
* 無法產生顏料混色效果。

### 修改

使用：

```css
mix-blend-mode: multiply;
```

預期：

* 黃 + 紅 → 橘
* 黃 + 藍 → 綠
* 紅 + 藍 → 紫

---

## 2. Multiply 模式行為調整

當：

```js
blend.mode === "multiply"
```

需要：

```js
shadow.enabled = false
allowStack = true
```

原因：

* 疊色物件不應產生實心陰影。
* 疊色需要允許積木重疊。

「是否可疊色」屬於 StylePack 屬性，不是全域功能開關。

---

# 三、既有 Style 調整

## 3. 現行黏土 Style 更名為 Paper

目前參數：

* 柔和陰影
* 圓角
* 無描邊

視覺較接近紙張／剪紙。

修改：

* 僅更名為 `Paper`
* 保留現有參數
* 不重新製作 Clay

---

## 4. Mondrian Style 調整

修改：

### 移除：

* Shadow

### Palette 權重：

調整為：

```
White : Black : Red : Blue : Yellow
  5   :   2   :  1  :  1  :  1
```

透過 `weight` 控制隨機生成比例。

---

# 四、新增 Style

## 5. Risograph Style

新增：

```js
id: "risograph"
```

設定：

### Blend

```js
mode: "multiply"
allowStack: true
```

### Shadow

```js
enabled: false
```

### Palette

包含：

* Fluorescent Pink
* Aqua Blue
* Yellow
* White

### Surface

* 米白紙背景
* 細微紙張顆粒

### Texture

可加入：

* 極輕微紙張紋理
* 網點效果

保持低強度，避免影響遊戲辨識度。

---

## Risograph 白色處理

Riso 白墨不是透明色。

新增屬性：

```js
opaqueInk: true
```

規則：

* 白色積木不參與 multiply。
* 白色積木需保持不透明。
* 白色需能覆蓋下層顏色。

`opaqueInk` 為 Risograph Style 專用需求，不需擴展為所有 Style 的必要屬性。

---

## 6. Glassine Style

新增：

```js
id: "glassine"
```

設定：

### Blend

```js
mode: "multiply"
allowStack: true
```

### Shadow

```js
enabled: false
```

### 特徵

* 使用 Mondrian Palette
* 保留黑色輪廓線
* 呈現透明色片重疊效果

---

## 7. Black & Gray Style

新增：

```js
id: "black_gray"
```

Palette：

```
White
Light Gray
Gray
Dark Gray
Black
```

設定：

```js
mode: "multiply"
allowStack: true
shadow.enabled = false
```

---

# 五、StylePack 結構

所有 Style 設定集中管理。

避免新增全域常數。

結構：

```js
{
  id,
  name,

  surfaces:[
    {
      id,
      name,
      bg,
      texture
    }
  ],

  palette:[
    {
      hex,
      weight,
      opaqueInk
    }
  ],

  shape:{
    cornerRadius,
    strokeWidth,
    strokeColor
  },

  shadow:{
    enabled,
    offsetX,
    offsetY,
    blur,
    color
  },

  blend:{
    mode,
    allowStack
  },

  motion:{
    dropBounce,
    dropDuration,
    liftScale
  }
}
```

---

# 六、本批不實作

* Reveal 顯影模式
* 背景與 Style 綁定
* Clay Style 重新製作
* 閃粉效果
* 碎光效果

---

# 七、驗收條件

完成後確認：

1. Multiply 疊色可正常產生混色色相。
2. Multiply 模式不產生陰影。
3. Multiply 模式允許積木重疊。
4. Mondrian Palette 比例符合：

```
White 5
Black 2
Red 1
Blue 1
Yellow 1
```

5. Risograph 白色積木不會因 Multiply 消失。
6. Glassine 可正常呈現透明色片疊色。
7. Black & Gray 可正常疊色。
8. 原有遊戲功能與操作方式不受影響。

---

完成後，若發現架構限制或可改善項目，請另外列出「後續重構建議」，不要直接修改架構。
