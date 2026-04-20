### VS Code / Cursor 的终极解惑：`Cmd + P` 到底干嘛的？

你总是分不清，是因为 **Obsidian 把这两个键的逻辑搞反了**！ 在极其硬核的程序员世界（VS Code / Cursor）里，它的逻辑是极其严谨且统一的。请死死记住下面这个公式：

- 📄 **`Cmd + P` = 找“文件” (Path / Project Files)**
    
    - 敲下它，弹出的框是用来搜索**文件名字**的。比如你想打开 `main.py`，敲 `Cmd + P`，输入 `main`，回车。
        
- 🛠️ **`Cmd + Shift + P` = 找“命令” (Command Palette)**
    
    - 加上了 `Shift` 键，代表你“切换”了操作维度。这时候弹出的框，才是用来执行**动作**的（比如格式化代码、调出 Git、切换主题）。
        
    - **结论：VS Code 的 `Cmd + Shift + P`，才等价于 Obsidian 的 `Cmd + P`！**