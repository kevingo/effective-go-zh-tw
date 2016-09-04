# If

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