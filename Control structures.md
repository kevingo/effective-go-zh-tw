# Control structures


Go 的控制結構和 C 接近，但是在關鍵的地方還是有些區別。Go 中沒有 `while`， `switch` 用起來更加靈活，`if` 和 `switch` 可以使用初始化語句。Go 增加了新的控制結構，包含類型 `switch` 常用來判斷 interface 的動態類型。還有多路通信復用的select。語法上也有些細小的差別：控制結構並不需要左右括號，但是大括號還是需要的。

## If

在 Go 中，一個最基本的 `if` 長這樣：

```
if x > 0 {
    return y
}
```

強制使用大括弧鼓勵大家把簡單的 `if` 也寫成多行，這是比較好的程式風格，特別是你的 `if` 裡面含有 `return` 或 `break`。

由於 `if` 和 `switch` 接受變數初始化的寫法，你會常常看到某些人會用它在設定一個區域變數：

```
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

在 Go 的函式庫中，你會發現 `else` 經常是被省略的，特別是 `if` 描述中以 `break`、`continue`、`goto` 或 `return` 結尾。
```
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

下面這段程式中描述了 Go 語言中常見的錯誤判斷以及處理方式，由於在錯誤發生後就直接返回了，因此並不需要 `else` 描述。


```
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

## Redeclaration and reassignment

上面的範例中，其實還展示了如何使用 `:=` 來描述變數。這個程式片段使用 `os.Open` 來讀檔：

```
f, err := os.Open(name)
```

這一行宣告兩個變數：`f` 和 `err`。之後便呼叫 `f.Stat` 來讀取：

```
d, err := f.Stat()
```

而上面這一行看起來又宣告了 `d` 和 `err`。你會發現 `err` 在兩個宣告中都出現了。這樣的重複宣告是合法的，在第一個敘述中，`err` 被宣告了，而在第二個敘述中，我們重新指派內容到 `err` 變數。這代表 `f.Stat()` 只是重新指派一個值給 `err` 變數而已。

符合以下條件時，`:=`用來宣告一個已經定義過的變數是合法的，

- 第二次宣告和 v 在同樣的作用域下（如果 v 是在函數外定義的，那在函數內定義的 v 就會建立一個新的變數，並且覆蓋掉函數外的 v）
- 要重新指派給第二次宣告的 v 得值，是要可以被指派的，同時型態必須一致
- 第二次宣告的 v 必須要和另外一個新的變數一起被宣告

這些少見的特型代表了實用主義，你可以在一連串的 `if-else` 描述中重複使用 `err`。


## For

Go 的 For 迴圈類似於 C，但又有點不同。Go 的 For 結合ㄌ了 for 和 while，也因此在 Go 中並沒有 do while 的語法。在 Go 中，for 總共有三種形式，其中只有一種包含了分號。

```
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```


這種描述法讓 index 變數很容易被描述在敘述中：
```
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

如果你想要在 array、slice、string、map 或是從 channel 中讀取的內容、或是一個 range 中使用 for 的話，你可以使用 `range` ：

```
for key, value := range oldMap {
    newMap[key] = value
}
```

如果你只需要在 range 中的第一個 item（可能是 key 或 index），丟掉第二個變數宣告：

```
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

如果你只想要第二個 item（只想要值變量），使用 `_` 變數（一個底線）來捨棄第一個 item：

sum := 0
for _, value := range array {
    sum += value
}

這個 `_` 描述符號有許多的用途，我們會在[之後的章節]()中描述之。

對於 string 來說，range 函示有更多有用的用途：它會解析 UTF-8 的編碼，幫你做單一字元的切割。如果有錯誤編碼的字元則會花費 1 個 byte 的空間，並且被置換成 rune U+FFFD type（rune 是 Go 的保留字元，用來代表一個 unicode）。以下的迴圈：

```
for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
```

會印出：

```
character U+65E5 '日' starts at byte position 0
character U+672C '本' starts at byte position 3
character U+FFFD '�' starts at byte position 6
character U+8A9E '語' starts at byte position 7
```

最後，Go 沒有逗號運算子，而且 `++` 和 `--` 是陳述，而不是表達式。因此，如果你想要在 for 迴圈中執行多個變數，你應該使用多變量賦值表示法：


// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}

## Switch

Go 的 `switch` 結構比 C 更加通用。它的表達式不需要非得是常數或整數，`case`語句從頭開始執行，直到和某個 `case` 匹配。所以可以使用 `switch` 來改寫 `if-else-if-else`。

```
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

Go的 `switch` 並不像 C 一樣如果不加 `break` 會導致和某個 `case` 匹配後，繼續執行之後的 `case`。但是可以使用逗號分割的列表來達到此目的

```
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

儘管它們在 Go 中的用法和其它 C-like 語言差不多，但 `break` 可以使 `switch` 提前終止。不僅是 `switch`， 有時候也必須打破多層的循環。在 Go 中，我們只需將標籤放置到迴圈外，然後 「跳」 到那裡即可。下面的例子展示了二者的用法：


```
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
	}
```

當然，`continue` 也能接受一個選擇性的標籤，不過它只能在迴圈中使用。

作為這一節的結束，下面的程式碼使用兩個 `switch` 來比較兩個 byte slice。

```
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
```

## Type switch

`switch` 也可用於判斷介面變數的動態型別。例如：類型選擇通過括號中的關鍵字 `type` 使用類型斷言語法。若 `switch` 在表達式中聲明了一個變數，那麼該變數的每個子句中都將有該變數對應的類型。在這些 `case` 中重用一個名字也是符合語義的，實際上是在每個 `case` 裡聲明了一個不同類型但同名的新變數。


```
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```