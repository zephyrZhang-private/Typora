## **使用技巧**

*如下是使用宏，合并指定行的指定列，并将值传给合并后的单元格*

> `ALT`+`F11` 打开VBA编辑器
>
> `插入` - `模块` 粘贴代码
>
> ````
> Sub MergeFGHIRows144To183()
>     Dim ws As Worksheet
>     Dim rng As Range
>     Dim cell As Range
>     Dim startRow As Long
>     Dim endRow As Long
>     Dim startCol As Long
>     Dim endCol As Long
>     Dim i As Long
>     Dim combinedValue As String
>     
>     Set ws = ActiveSheet
>     startRow = 144
>     endRow = 183
>     startCol = 6 ' F 列是第 6 列
>     endCol = 9 ' I 列是第 9 列
>     
>     For i = startRow To endRow
>         combinedValue = ""
>         For j = startCol To endCol
>             combinedValue = combinedValue & ws.Cells(i, j).Value & " "
>         Next j
>         combinedValue = Trim(combinedValue) ' 去掉末尾空格
>         Set rng = ws.Range(ws.Cells(i, startCol), ws.Cells(i, endCol))
>         rng.Merge
>         rng.Value = combinedValue
>     Next i
> End Sub
> ````
>
> 关闭VBA编辑器
>
> `ALT` - `F8` ，选择 `MergeFGHIRows144To183`，`执行`
>
> ***代码说明***
>
> ````
> startRow = 144：起始行号为 144。
> endRow = 183：结束行号为 183。
> startCol = 6：F 列是第 6 列。
> endCol = 9：I 列是第 9 列。
> combinedValue：将每行的 F、G、H、I 列值拼接成一个字符串。
> Trim(combinedValue)：去掉字符串末尾的空格。
> rng.Merge：横向合并单元格。
> rng.Value = combinedValue：将拼接后的值赋给合并后的单元格。
> ````

