要理解这个看似反人类的 `super(SarsaTable, self)`，我们需要明白 Python 内部是怎么找“爸爸”（父类）的。

### 1. 为什么里面写的是 `SarsaTable` 而不是 `RL`？

很多人直觉上认为 `super` 就是“呼叫父类”，所以应该写 `super(RL)`。但实际上，Python 里的 `super` 的真正含义是：**“顺着族谱往下找，给我下一个” (Next in line)。**

你可以把 `super(SarsaTable, self)` 翻译成一句给 Python 解释器的指令：

> **“请以 `self`（这个具体的对象）的族谱为基准，找到 `SarsaTable` 这个类的位置，然后把排在它后面的那个类（也就是它的父类）给我。”**

因为 `SarsaTable` 继承自 `RL`，所以在这份族谱（专业术语叫 MRO，方法解析顺序）里，排名是这样的：

1. `SarsaTable` （当前类）
    
2. `RL` （父类）
    
3. `object` （所有类的老祖宗）
    

当你写 `super(SarsaTable, self)` 时，Python 找到了 `SarsaTable`，然后向后顺延一位，精准地抓到了 `RL`。

### 2. 为什么不直接写 `RL.__init__(self)`？

在很久以前的 Python 里，大家确实是直接点名道姓地呼叫父类的：

Python

```
class SarsaTable(RL):
    def __init__(self, ...):
        RL.__init__(self, ...)  # 直接点名让 RL 干活
```

**既然能直接点名，为什么还要发明 `super` 这么别扭的东西？**

这是为了应对**代码重构**和**多重继承**的麻烦：

- **如果父类改名了：** 假设有一天你不想叫 `RL` 了，改名叫 `BaseAgent`。如果你用的是直接点名的方式，你就得把代码里所有的 `RL.__init__` 全改成 `BaseAgent.__init__`。但如果你用的是 `super(SarsaTable, self)`，它会自动去找新的父类，你什么都不用改。
    
- **多重继承的噩梦：** 如果一个类有两个父类（比如既继承 `RL` 又继承 `DeepLearning`），直接点名往往会导致某个父类被初始化两次，或者逻辑彻底乱套。`super` 会聪明地按照最优的族谱顺序，把所有父类按顺序排好，挨个执行一次。
    

### 3. 如果写错成 `super(RL, self)` 会发生什么？

如果你按照直觉，写成了 `super(RL, self).__init__()`，Python 会怎么做呢？ 它会翻开族谱：`[SarsaTable -> RL -> object]` 它定位到了 `RL`，然后**向后顺延一位**，找到了 `object`（老祖宗类）。结果就是，它跑去调用了老祖宗的初始化，完美地**跳过了**你真正想调用的 `RL` 的初始化，导致你的 Q表 根本没建出来，程序直接报错。

---

**总结一下：** Python 2 的 `super(当前类名, self)` 是一种相对严谨但反直觉的语法，意思是**“在这个对象的继承链中，寻找‘当前类’的上一级”**。

好消息是，Python 的开发者后来也觉得这太蠢了，所以在现在的 **Python 3 中，你永远只需要写 `super().__init__()` 即可**，Python 在后台会自动帮你填入 `(SarsaTable, self)` 这两个参数。这就是语言进化的魅力！