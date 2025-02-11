---
title: Python 系列2：基本类型
date: 2020-02-28 14:52:01
tags:
    - python
    - types
categories: python
---

## 类型概览

Python中的数据类型可以分为两大类：

**不可变类型**
- 整型(int)
- 浮点型(float)
- 字符串(str)
- 字节(bytes)
- 元组(tuple)
- 不可变集合(frozenset)

**可变类型**
- 字节数组(bytearray)
- 字典(dict)
- 列表(list)
- 可变集合(set)

## 数值类型

### 整型(int)

Python3统一了int和long类型。整型只受限于内存大小，可以表示极大的数值：

```python
# 大数运算
>>> 2 << 100
2535301200456458802993406410752

# 除法运算
>>> 10 / 3   # 浮点除法
3.3333333333333335
>>> 10 // 3  # 整除
3
```

### 浮点型(float)

float采用双精度浮点数表示，默认精度为16-17位。由于基于二进制实现，可能产生精度误差：

```python
>>> 0.1 * 3
0.30000000000000004
>>> (0.1 + 0.1 + 0.1 - 0.3) == 0
False
```

对于需要精确十进制计算的场景，应使用`decimal.Decimal`：

```python
>>> from decimal import Decimal
# 使用字符串初始化Decimal以保证精度
>>> (Decimal("0.1") + Decimal("0.1") + Decimal("0.1") - Decimal("0.3")) == 0
True
```

### 布尔型(bool)

bool是int的子类型：
- True对应整数1
- False对应整数0
- 可以参与数值运算

```python
>>> True + 1
2
>>> list(range(10))[True]  # 可作为索引
1
```

## 序列类型

### 字符串(str)

字符串存储Unicode文本，是不可变序列。Python3将文本和二进制数据分离处理。

**Unicode编码说明**
- Unicode定义字符与码点(code point)的映射关系
- UTF-8是最常用的可变长度编码方案
- UTF-16/32提供定长编码，处理效率更高

```python
>>> s = "ɖ 中文"
>>> s.encode("utf-8")    # UTF-8编码
b'\xc9\x96\xe4\xb8\xad\xe6\x96\x87'
>>> hex(ord(s[1]))      # 获取字符码点
'0x4e2d'
```

### 字节序列(bytes/bytearray)

**bytes**: 不可变字节序列
```python
>>> bytes("中文", "utf-8")
b'\xe4\xb8\xad\xe6\x96\x87'
```

**bytearray**: 可变字节序列
```python
>>> b = bytearray(b"abc")
>>> b.append(0x4e)
>>> b
bytearray(b'abcN')
```

### 列表(list)

列表是可变序列，支持：
- 索引和切片访问
- 动态扩展
- 元素可以是任意类型

内部实现：
- 由头部(存储元素数量和内存分配计数)
- 和独立数组(存储元素指针)组成

注意：频繁插入删除操作时，建议使用`collections.deque`。
