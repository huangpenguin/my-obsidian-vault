---
title: "byte string"
publish: false
tags: ["Python"]
---
# byte string

```python

a = b'h\x65llo'
```

- `b'h\x65llo'` 表示一个字节字符串（byte string）。
- 前缀 `b` 表示这是一个字节字符串，而不是一个普通的字符串。
- 在字节字符串中，每个字符实际上是一个字节（8位）。其中 `\x65` 是一个十六进制的转义序列，表示 ASCII 值为 101 的字符，也就是 `e`。

因此，`b'h\x65llo'` 等价于 `b'hello'`。

```python
print(list(a))
```

- 这会将字节字符串 `a` 转换为一个包含每个字节对应的整数值的列表。
- `b'hello'` 中每个字符的 ASCII 值依次为：`h (104)`, `e (101)`, `l (108)`, `l (108)`, `o (111)`。

因此，这一行代码的输出是：

```python
[104, 101, 108, 108, 111]
```

```python
print(a)
```

- 这会直接打印字节字符串 `a`，显示它的字节字符串形式。

因此，这一行代码的输出是：

```python
b'hello'
```
