---
title: Base64 创建方式不同的 bug
date: 2024-05-16 16:23:13
tags: [base64, python, shell, echo]
---

#　python base64 的方法

在Python中，`base64`模块提供了一系列用于编码和解码Base64的函数。下面是一些常用的方法：

1. `base64.b64encode(s)`: 对字符串`s`进行Base64编码。这里的`s`应该是字节类型，如果你有一个字符串类型的数据，需要先用`.encode()`方法将其转换为字节类型。

2. `base64.b64decode(s)`: 对Base64编码的字符串`s`进行解码。返回的也是字节类型的数据。

3. `base64.urlsafe_b64encode(s)`: 和`b64encode`类似，但是生成的Base64编码会将`+`和`/`分别替换为`-`和`_`，使得编码后的结果可以在URL中安全地使用。

4. `base64.urlsafe_b64decode(s)`: 对用`urlsafe_b64encode`方法编码的字符串进行解码。

下面是一个简单的例子，展示如何使用这些方法：

```python
import base64

# 原始数据
data = "Python Base64 编码测试！"

# 将字符串转换为字节
data_bytes = data.encode('utf-8')

# 编码
encoded_data = base64.b64encode(data_bytes)
print("编码后的数据:", encoded_data)

# 解码
decoded_data = base64.b64decode(encoded_data)
print("解码后的数据:", decoded_data.decode('utf-8'))

# URL安全的编码
urlsafe_encoded_data = base64.urlsafe_b64encode(data_bytes)
print("URL安全的编码数据:", urlsafe_encoded_data)

# URL安全的解码
urlsafe_decoded_data = base64.urlsafe_b64decode(urlsafe_encoded_data)
print("URL安全的解码数据:", urlsafe_decoded_data.decode('utf-8'))
```

请注意，Base64编码不是加密方法，它仅仅是一种编码方式，可以很容易地被解码。不应该用来保护敏感信息。


# 为什么 shell 中的 echo 'abc' | base64 和 python 中的不一样？如何一样

在Shell中使用`echo`命令时，默认情况下，`echo`会在输出的末尾添加一个换行符（`\n`）。这个换行符也会被`base64`编码。而在Python中，如果你直接对字符串`'abc'`进行编码，你不会在字符串末尾添加换行符。

为了在Shell中得到与Python中相同的结果，你需要使用`echo`命令的`-n`选项，这样它就不会在字符串末尾添加换行符了。

Shell命令（不添加换行符）：

```sh
echo -n 'abc' | base64
```

对应的Python代码（不添加换行符）：

```python
import base64

data = 'abc'
encoded_data = base64.b64encode(data.encode('utf-8'))
print(encoded_data.decode('utf-8'))
```

两者现在应该给出相同的Base64编码结果。

如果你想在Python中得到包含换行符的版本，你可以这样做：

```python
import base64

data = 'abc\n'  # 添加换行符
encoded_data = base64.b64encode(data.encode('utf-8'))
print(encoded_data.decode('utf-8'))
```

这将模拟Shell中不带`-n`选项的`echo`命令的行为。