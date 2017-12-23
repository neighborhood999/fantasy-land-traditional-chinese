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

## Type Signature 符號

以下介紹文件中使用的 Type Signature 符號：

- _`::` 「is a member of」。_
   - `e :: t` 可以讀作：「表達式 `e` 是類型 `t` 的一位成員」。
   - `true :: Boolean` - 「`true` 是類型 `Boolean` 的一位成員」。
   - `42 :: Integer, Number` - 「`42` 是類型 `Integer` 和 `Number` 的一位成員」。

- _新 type 可以藉由 type constructor 被建立。_
   - Type constructor 可以接受零個或多個 type 參數。
   - `Array` 是一個 type constructor，它接受一個 type 參數。
   - `Array String` 是一個字串陣列的 type。以下每個 type 都是 `Array String`：`[]`、`['foo', 'bar', 'baz']`。
   - `Array (Array String)` 是一個陣列的字串陣列 type。以下每個 type 都是 `Array (Array String)`：`[]`、`[ [], [] ]`、`[ [], ['foo'], ['bar', 'baz'] ]`。

- _小寫字母代表 type 變數。_
   - Type 變數可以接受任何 type，除非已經透過 type constraint 被限制（參考以下 fat arrow）。

- `->`（arrow）_Function type constructor。_
   - `->` 是一個 infix type constructor，它接受兩個 type 參數，左邊參數是輸入的 type，右邊的則是輸出 type。
   - `->` 的輸入 type 可以是一組 type 來建立接受零個或多個的函式類型。語法是：`(<input-types>) -> <output-type>`，其中 `<input-types>` 包含零個或多個逗號（`,`）來分隔 type 表示，對於一元函式可以省略括號。_（參考此 [Issue](https://github.com/fantasyland/fantasy-land/issues/279) 解釋）_
   - `String -> Array String` 是一個透過函式滿足的 type，接受一個 `String` 並回傳一個 `Array String`。
   - `String -> Array String -> Array String` 是一個透過函式滿足的 type，它接受一個 `String` 並回傳一個函式接受一個 `Array String` 最後回傳一個 `Array String`。
   - `(String, Array String) -> Array String` 是一個透過函式滿足的 type，它接受一個 `String` 和一個 `Array String` 作為參數，並回傳一個 `Array String`。
   - `() -> Number` 是一個透過函式滿足的 type，它不接受任何的參數，並回傳一個 `Number`。

- `~>`（squiggly arrow）_Method type constructor。_
   - 當函式是一個 Object 的屬性時，它被稱為一個方法。所有的方法有一個隱式的參數 type - 它們是屬性的 type。
   - `a ~> a -> a` 是 type `a` 的 Object 上的方法所滿足的類型，它將 type `a` 作為參數並回傳 type `a` 的值。

- `=>`（fat arrow）_表示對 type 變數的 constraint。_
   - 在 `a ~> a -> a`（參考上方的 squiggly arrow），`a` 可以是任何 type。`Semigroup a => a ~> a -> a` 加入了一個 constraint，使得 type `a` 現在必須滿足 `Semigroup` typeclass。滿足 typeclass 意思是，合法地實現 typeclass 指定的所有函式和方法。

例如：

```
traverse :: Applicative f, Traversable t => t a ~> (TypeRep f, a -> f b) -> f (t b)
'------'    '--------------------------'    '-'    '-------------------'    '-----'
'           '                               '      '                        '
'           ' - type constraints            '      ' - argument types       ' - return type
'                                           '
'- method name                              ' - method target type
```

---

更多資訊，請參考 Sanctuary 文件的 [Type](https://sanctuary.js.org/#types) 部分。[↩️](https://github.com/fantasyland/fantasy-land#sanctuary-types-return)

## 前綴方法名稱

為了讓 data type 可以兼容 Fantasy Land，其值必須具有某些屬性。這些所有屬性都由 `fantasy-land/` 作為前綴。例如：

```js
//  MyType#fantasy-land/map :: MyType a ~> (a -> b) -> MyType b
MyType.prototype['fantasy-land/map'] = ...
```

此外，在本份文件沒有使用前綴名稱。

為了方便你可以使用 `fantasy-land` 的 package：

```js
var fl = require('fantasy-land')

// ...

MyType.prototype[fl.map] = ...

// ...

var foo = bar[fl.map](x => x + 1)
```

## Type 代表

某些行為是從一個 type 成員的角度定義的。其他行為不需要一個成員。因此，某些代數要求一個 type 提供一個 value 層級的代表（具有一定的屬性）。例如，可以提供 `Id` 作為其 type 代表：`Id :: TypeRep Identity`。

如果一個 type 提供了一個 type 代表，該 type 的每個成員必須有一個 `constructor` 屬性，該屬性是對 type 代表的引用。

## 代數

### Setoid

1. `a.equals(a) === true`（反身性）
2. `a.equals(b) === b.equals(a)`（對稱性）
3. 如果 `a.equals(b)` 而且 `b.equals(c)`，因此 `a.equals(c)`（傳遞性）

#### `equals` 方法

```haskell
equals :: Setoid a => a ~> a -> Boolean
```

具有 Setoid 的值必須提供一個 `equals` 方法。`equals` 方法接受一個參數：

```
a.equals(b)
```

1. `b` 必須是相同於 Setoid 的值。
    - i. 如果 `b` 不相同於 Setoid，`equals` 的行為是沒有被指定的（推薦回傳 `false`）。
2. `equals` 必須回傳一個布林值（`true` 或 `false`）。

### Ord

實作 Ord 規範的該值也必須實作 [Setoid](https://github.com/fantasyland/fantasy-land#setoid) 規範。

1. `a.lte(b)` 或 `b.lte(a)` (完整性)
2. 如果 `a.lte(b)` 而且 `b.lte(a)`，因此 `a.equals(b)` (反對稱性)
3. 如果 `a.lte(b)` 而且 `b.lte(c)`，因此 `a.lte(c)`（傳遞性）

#### `lte` 方法

```haskell
lte :: Ord a => a ~> a -> Boolean
```

具有 Ord 的值必須提供一個 `lte` 方法。`lte` 方法接受一個參數：

```
a.lte(b)
```

1. `b` 必須是相同於 Ord 的值。
    i. 如果 `b` 不相同於 Ord，`lte` 的行為是沒有被指定的（推薦回傳 `false`）。
2. `lte` 必須回傳一個布林值（`true` 或 `false`）。
