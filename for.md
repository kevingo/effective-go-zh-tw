# For

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