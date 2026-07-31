---
title: "Async concepts"
publish: false
tags: ["Python"]
---
# Async concepts

# Python 异步编程核心概念笔记

## 🧱 1. Blocking（阻塞）

- **定义**：当一个函数执行时，它会阻塞程序的下一步操作，直到该函数执行完毕。
- **特点**：
    - 常见于传统的 I/O 操作，例如 `time.sleep()`、文件读写、网络请求等。
    - 阻塞会让整个线程“停住”。

### 🧪 示例：

```python
python
复制编辑
import time

def blocking_function():
    print("Start")
    time.sleep(2)  # 阻塞2秒
    print("End")

blocking_function()

```

- 输出间隔为 2 秒，因为 `sleep()` 阻塞了主线程。

---

## 🧪 2. Non-blocking（非阻塞）

- **定义**：函数调用不会阻塞线程，它会立即返回并让程序继续运行。
- **应用场景**：用于并发或异步操作，让程序能同时处理多个任务。

### 🧪 示例（使用 `asyncio`）：

```python
python
复制编辑
import asyncio

async def non_blocking_function():
    print("Start")
    await asyncio.sleep(2)  # 非阻塞等待
    print("End")

asyncio.run(non_blocking_function())

```

- 即使 `sleep(2)`，程序仍然可以在这段时间去处理别的任务。

---

## 🔁 3. `yield`

- **定义**：`yield` 是用于生成器的关键字，用于**暂停函数执行**并返回一个值，下一次调用会从中断处继续执行。
- **适用于**：迭代器、数据流、协程的早期实现。

### 🧪 示例：

```python
python
复制编辑
def counter():
    yield 1
    yield 2
    yield 3

for num in counter():
    print(num)

```

- `yield` 让函数可以一个一个地“懒加载”值。

---

## ⏳ 4. `await`

- **定义**：`await` 用于**挂起**当前异步函数的执行，直到被 `await` 的任务完成。
- **只能在 `async def` 中使用**。
- 类似于 `yield`，但它适用于**异步协程**而非生成器。

### 🧪 示例：

```python
python
复制编辑
import asyncio

async def wait_and_print():
    await asyncio.sleep(1)  # 挂起1秒
    print("Done waiting")

asyncio.run(wait_and_print())

```

- `await` 会将控制权交还给事件循环，允许其他任务执行。

---

## 📌 小结对比表：

| 概念 | 是否阻塞线程 | 适用场景 | 是否暂停当前函数 | 控制权交还 |
| --- | --- | --- | --- | --- |
| blocking | ✅ 是 | 普通函数、同步代码 | ❌ 否 | ❌ 否 |
| non-blocking | ❌ 否 | 异步编程 | ✅ 可能 | ✅ 是 |
| `yield` | ✅ 是（但暂停函数） | 生成器、惰性迭代 | ✅ 是 | ✅ 是 |
| `await` | ❌ 否 | 异步协程函数 | ✅ 是 | ✅ 是 |
