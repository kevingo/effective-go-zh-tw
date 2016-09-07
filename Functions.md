
# Functions

## Multiple return values

Go的一個特有性質就是函數和方法具有多個返回值。這種特性使C程序中各種笨拙習慣用法得以改善：帶內返回錯誤（例如-1代表EOF）和通過傳遞地址修改一個參量。

在 C 中，一個寫錯誤是使用一個負數來標誌，該錯誤代碼隱藏在另外的不確定的位置。在 Go 中，Write 可以返回一個數值和一個錯誤：「是的，您寫入了一些位元組，但並沒有全部寫入，因為儲存裝置已滿」。在 os 函式庫中 Write 方法的簽名是：

```
func (file *File) Write(b []byte) (n int, err error)
```

正如文件所述，當 `n != len(b)` 時，它返回被寫入的位元組的數目以及一個非 `nil` 的 error；這是一個常用的方式；參見[錯誤處理章節]()獲得更多範例。

## Named result parameters

Go 函數的返回值或結果「參數」可以給定名稱並像一般的變數來使用，就像接收的參數那樣。命名後，一旦函數開始執行，他們就被初始化為其類型對應的零值；如果函數中的`return` 不帶參數，結果參數的當前值將作為返回值返回。


此名稱並不是強制要求的，但它能使代碼變得更加簡短和清晰：他們就是一種文件。如果我們命名了 `nextInt` 的返回值，就能很容易地知道各個返回的 `int` 所代表的意思。

```
func nextInt(b []byte, pos int) (value, nextPos int) {
```

由於被命名的返回結果被初始化，並且可以和一個不帶參數的 `return` 綁定，他們不僅可使程式碼變得清晰，也可使程式碼簡化。這裡的 `io.ReadFull` 是一個很好的範例：

```
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

## Defer

Go 的 `defer` 語句預設一個函數調用（延期的函數），該調用在函數執行 `defer` 返回時立刻運行。該方法顯得不同常規，但卻是處理一些情況的有效方式，如無論函數怎樣返回，都必須進行資源釋放。典型的例子是解開一個互斥鎖並關閉文件。

```
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

對像Close這樣的函數的延期調用有兩個優點。第一，它確保你不會忘記關閉文件，在一段時間之後編輯函數以便向其中添加新的返回路徑時，往往會發生此種錯誤。第二，它意味著關閉與打開靠得很近，這要比將關閉放在函數結尾處更為清楚明了。

被延期函數的參量（如果函數是一個方法，將還包括接收者）是在進行延期時被估值，而不是在調用時被估值。這樣不僅可不必擔心變量值被改變，同時也意味著單個延期調用可以延期多個函數執行。以下是一個不太聰明的例子：

```
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```

被延期的函數以後進先出（LIFO）的順行執行，因此以上代碼在返回時將打印4 3 2 1 0。一個更合理的例子是用一種簡單的方法通過程序追蹤函數調用。我們能以如下方式寫一些簡單的追蹤例程：

```
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```

我們可以通過利用被延期函數的參量在defer執行時被估值的特點更好地完成工作。追蹤例程可以針對非追蹤例程建立參量。如下例所示：

```
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

此程序將打印

```
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

對於習慣於其他語言的塊級資源管理的程序員，defer看起來有點怪異。但它最有趣和強大的應用恰恰來自於它是基於函數而不是基於塊的特點。在panic和recover節中我們將看到它的另一種應用的例子。





