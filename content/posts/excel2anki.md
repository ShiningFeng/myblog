+++
title = "从 Excel 导入 Anki：批量制作卡片的完整指南"
date = 2025-03-15T08:00:00+08:00
draft = false
tags = [["Memos", "教程"]]
summary = "一文教你如何从 Excel 批量导入选择题至 Anki，包含前缀添加、字段清理、CSV 导出与模板导入等详细步骤。"
+++

使用 Anki 记忆工具时，很多人希望批量导入卡片，尤其是已有的 Excel 题库。本文将从 Excel 数据准备、字段清洗，到 Anki 模板配置、导入映射，手把手教你完成批量制卡。

---

## 1. Excel 数据准备

### 1. 1 字段规划（以选择题为例）

| 列号   | 内容说明       |
| ---- | ---------- |
| B    | 问题         |
| C\~F | 选项 A/B/C/D |
| G    | 正确答案       |
| H    | 解析（可选）     |

建议每列对应 Anki 中的一个字段，方便后续映射。

---

### 1.2 批量添加选项前缀

为指定列 `C/D/E/F` 的文本内容自动添加字母前缀  `A./B./C./D.`，提高排版一致性。使用如下 VBA 宏：

```vb
Attribute VB_Name = "AddAlphaHeader"
Sub AddCorrectPrefixes()
    Dim ws As Worksheet
    Dim lastRow As Long
    Dim i As Long
    
    ' 定义工作表
    Set ws = ThisWorkbook.Sheets("Sheet1") ' 请根据你的实际表名修改 "Sheet1"
    
    ' 找到最后一行
    lastRow = ws.Cells(ws.Rows.Count, "B").End(xlUp).Row
    
    ' 遍历每一行，为 C、D、E、F 列添加前缀
    For i = 5 To lastRow ' 假设从第5行开始，需根据实际情况修改
        If ws.Cells(i, "C").Value <> "" Then ws.Cells(i, "C").Value = "A. " & ws.Cells(i, "C").Value
        If ws.Cells(i, "D").Value <> "" Then ws.Cells(i, "D").Value = "B. " & ws.Cells(i, "D").Value
        If ws.Cells(i, "E").Value <> "" Then ws.Cells(i, "E").Value = "C. " & ws.Cells(i, "E").Value
        If ws.Cells(i, "F").Value <> "" Then ws.Cells(i, "F").Value = "D. " & ws.Cells(i, "F").Value
    Next i
End Sub
```

**提示**：根据实际的 Excel 文件，适当修改宏中“工作表名称”和循环起始行。

---

### 1.3 清理无效字符

有些题干或选项前会有中文标点或空格，可以使用如下宏清除开头杂字符：

```vb
Attribute VB_Name = "SanitizeLead"
Function CleanText(cell As Range) As String
    Dim str As String
    Dim i As Integer
    Dim symbols As String
    
    ' 需要删除的符号
    symbols = "、，。．."

    str = cell.Value

    ' 删除开头的空格和指定符号
    Do While Len(str) > 0 And (Left(str, 1) = " " Or Left(str, 1) = ChrW(12288) Or InStr(symbols, Left(str, 1)) > 0)
        str = Mid(str, 2)
    Loop
    
    CleanText = str
End Function
```

---

## 2. 导出为 CSV（UTF-8）

1. Excel 顶部菜单 → `文件 > 另存为`
2. 格式选择：`CSV UTF-8（逗号分隔）`
3. 文件名建议为英文，如 `anki_cards.csv`

---

## 3. Anki 设置与导入

### 3.1 创建新的 Note Type（笔记类型）

* 打开 Anki → `Tools > Manage Note Types`
* `Add` 创建或克隆一个新的类型（如“多选题”）
* `Fields`：添加与 Excel 对应的字段（如 问题、A、B、C、D、正确答案、解析）
* `Cards`：编辑前/背面模板，例如：  

**Front Template**

```html
<div>
    问题：{{问题}}<br><br>
    {{A}}<br>
    {{B}}<br>
    {{C}}<br>
    {{D}}
</div>
```

**Back Template**

```html
{{FrontSide}}
<hr>
正确答案：{{正确答案}}<br>
解析：{{解析}}
```

---

### 3.2 导入 CSV 文件

1. 打开 Anki → `File > Import` 选择 `.csv` 文件，编码确认为 UTF-8
2. `Type`：选“多选题”或对应的 Note Type
3. `Deck`：选择牌组
4. `Delimiter`：选择逗号（CSV）
5. 设置字段映射：
   * B → 问题
   * C → A
   * D → B
   * E → C
   * F → D
   * G → 正确答案
   * H（可选）→ 解析
6. 完成导入后点击 “Import”

---

## 4. 预览与检查

* 点击 Browse，选择你刚导入的牌组
* 确认每个字段对应正确
* 使用卡片预览检查格式是否清晰

---

## Tips & 常见问题

* 若乱码，确认保存时编码为 UTF-8
* 多余空行会导入空卡片，导入前请清理数据
* 选项顺序建议保持一致，否则模板显示混乱
* 若多个字段需要清洗，可在 Excel 中先使用 VBA 批处理