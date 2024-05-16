---
title: Shell 小技巧 —— Sed
date: 2024-05-16 16:16:07
tags: [shell, sed, linux, ChatGPT]
---

# sed 语法可以用其他字符代替 / 吗 's/abc/xyz/g'

是的，在`sed`命令中，通常使用的分隔符是正斜杠`/`，但是你可以使用几乎任何其他字符作为分隔符。如果你需要处理包含正斜杠的路径或数据，使用其他字符作为分隔符会很方便。

例如，如果你想把字符串`abc`替换成`xyz`，你可以写成：

```bash
sed 's/abc/xyz/g'
```

如果你的模式或替换字符串中包含正斜杠，如`/path/to/abc`替换为`/path/to/xyz`，你可以使用其他字符，例如`#`，来避免混淆：

```bash
sed 's#/path/to/abc#/path/to/xyz#g'
```

或者使用下划线`_`：

```bash
sed 's_/path/to/abc_/path/to/xyz_g'
```

使用不同的分隔符可以让`sed`命令更加可读，尤其是当处理包含正斜杠的文件路径时。只需确保所选的分隔符不会在模式或替换文本中出现，否则你需要对该字符进行转义。