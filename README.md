# Fantasy Land Specification

![Fantasy Land Specification](https://raw.githubusercontent.com/fantasyland/fantasy-land/master/figures/dependencies.png)

## 概述

代數是一組值，一組操作，它是封閉的，必須遵守一些規則。

每個 Fantasy Land 代數是一個獨立的規範。一個代數可能依賴於其他必須實現的代數。

## 術語

1. 「value」是任何 JavaScript 的值，包含具有下面定義的的任何結構。
2. 「equivalent」對於給定的值有一個等價的定義。這個定義應該確保這兩個值可以在一個尊重抽象的程式中安全的被交換出來。例如：
   - 如果兩個 list 所有指數都是相同的，那它們是相同的。
   - 兩個被解釋為 dictionary 的 JavaScript 純 Object，當它們有相等的 key 時，它們是等價的。
   - 當兩個 Promise 產生相等的值時，它們是等價的。
   - 如果兩個函式輸入相同的值，且得到相同的輸出，它們是等價的。
