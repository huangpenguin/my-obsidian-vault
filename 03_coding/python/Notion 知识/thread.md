---
title: "thread"
publish: false
tags: ["Python"]
---
# thread

`daemon=True` 的关键词是 **“守护线程”**（Daemon Thread）。

### **解释：**

- **守护线程（Daemon Thread）：**
    - 当**主线程结束时，守护线程会自动终止**，不会阻止程序退出。
    - 适用于**后台任务**，比如日志记录、监控、定时任务等。
    - **一旦主线程退出，所有 daemon 线程都会被强制终止，而不会执行 `finally` 代码块。**

---

### **关键词/关键点：**

- **主线程退出时，守护线程自动终止**
- **适合后台运行**
- **不保证完成所有任务**
- **主线程是非守护线程，默认启动的线程也是非守护的**
- **必须在 `start()` 之前设置 `daemon=True`**

---

创建守护线程的方式通常是：

```python
import threading

def background_task():
    while True:
        print("Background task running")

# Create daemon thread
thread = threading.Thread(target=background_task,daemon = True)
thread.start()
```
