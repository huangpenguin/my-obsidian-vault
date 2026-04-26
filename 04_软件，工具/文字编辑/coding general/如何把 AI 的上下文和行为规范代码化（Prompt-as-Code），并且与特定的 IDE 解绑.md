> [!NOTE]
> 我现在是练习rl，想拷贝一个rl的项目跑一跑，但是最近ai技术规范发展的很快，我也想最好是规范一下自己的开发， 目前我在另一个项目中有.cursor文件夹放一系列mdc来控制cursor的行为，请问我如果想放到全局curso设置中去的话怎么做比较好：
> 
> 1.使用最新的best practice；
> 
> 2.考虑不同agent之间可能迁移（比如以后可能不使用cursor，转为使用claude code的时候），要有一定的兼容性；
> 
> 3.除了全局配置之外我还想创建一个pj模版，之后比如我新建也好clone也好能够直接把这套ai框架套进去，你觉得怎么实现比较好
> 


---

# 🚀 AI 极客全自动工程化操作手册

### 极客的“双保险”策略

如果你经常在两个 Shell 之间切换，或者不确定以后会用哪个，这里有三种处理方案，按**推荐程度**排序：

#### 方案 A：最省事 —— 哪里需要点哪里（推荐）

如果你查出来当前是 `zsh`，就写进 `~/.zshrc`；如果是 `bash`，就写进 `~/.bashrc`。

#### 方案 B：最极客 —— “套娃”同步法（高级推荐）

很多极客会把所有的 `alias` 全都写在 `~/.bashrc` 里，然后在 `~/.zshrc` 的最后一行加上一句：

```
# 在 .zshrc 里加这一句，意思是：顺便把 bash 的字典也背下来
[ -f ~/.bashrc ] && source ~/.bashrc
```

这样你只需要维护一份 `~/.bashrc` 里的别名，`zsh` 也会自动同步。这是最专业的做法，因为 `bash` 是所有服务器的“公约数”。


```
当前 remotes：

- `origin`: `git@github.com:huangpenguin/ai-coding-rules.git`
- `gitlab`: `git@gitlab.com:huang.pengbin/ai-coding-rules.git`
## 下载仓库，并设置别名

如果服务器默认是 bash，就写进 bashrc
echo 'alias init-ai="bash ~/.ai-coding-rules/inject-ai.sh"' >> ~/.bashrc
## 场景一：从零新建自己的项目 (The Creator)

_适用情况：你今天突然有了个好点子，准备写一个新的强化学习（RL）算法。_
1. **初始化基础设施**
    mkdir my-new-rl && cd my-new-rl
    uv init          # 秒级生成现代 pyproject.toml 骨架
    git init         # 初始化 Git（为了让后面的 hook 有地方挂）
    ```
    
2. **一键注入 AI 灵魂**：
    
    Bash
    
    ```
    init-ai
    ```
    
3. **背后发生了什么**：
    
    - 注入了全中文、强类型的 AI 提示词（Cursor / Claude Code 瞬间懂规矩了）。
        
    - `uv` 自动为你安装了 `ruff`, `pyright`, `pre-commit` 等开发依赖。
        
    - Git 自动挂载了代码拦截器。
        
4. **下一步**：直接打开 Cursor 唤出 Composer：“根据项目里的规范，帮我写一个 PPO 算法的脚手架。”
    

---

## 场景二：克隆接手“现代”开源项目 (The Contributor)

_适用情况：你在 GitHub 上 clone 了一个 2024 年以后的项目，它根目录下自带 `pyproject.toml`。_

1. **拉取与进入**：
    
    Bash
    
    ```
    git clone https://github.com/someone/cool-rl.git
    cd cool-rl
    ```
    
2. **一键注入与环境同步**：
    
    Bash
    
    ```
    init-ai          # 注入你的 AI 私人规范和格式化工具
    uv sync          # 让 uv 把项目的核心依赖装好
    ```
    
3. **背后发生了什么**：
    
    - 你成功把**你自己的**高质量开发流，强加到了别人的项目上。
        
    - 别人的代码也许很乱，但在你本地，AI 帮你改代码时会严格按照你 `ruff` 设定的 120 行宽和严谨的中文注释来写。
        
4. **下一步**：随便改点代码，敲下 `git commit`，让 pre-commit 帮你把烂格式自动洗刷一遍。
    

---

## 场景三：接手“祖传/上古”老旧项目 (The Archaeologist)

_适用情况：导师或者网上的老项目，只有 `requirements.txt`，甚至连环境隔离都没做。_

1. **拉取与创建独立环境**：
    
    Bash
    
    ```
    git clone https://github.com/old/legacy-rl.git
    cd legacy-rl
    uv venv          # 强制创建一个干净的虚拟环境
    uv pip install -r requirements.txt
    ```
    
2. **一键优雅降级注入**：
    
    Bash
    
    ```
    init-ai
    ```
    
3. **背后发生了什么**：
    
    - 脚本探测到没有 `pyproject.toml`，**聪明地跳过了**安装 Ruff/Pyright 的步骤（防止搞脏老项目的依赖树）。
        
    - 但是，它把 `.cursor/rules` 和 `CLAUDE.md` 塞进去了！
        
4. **下一步**：就算没有严格的代码检查，AI 依然会遵循 MDC 规则，用中文为你解释这些上古代码，并在修改时使用现代的 Type Hints。
    

---

## 场景四：日常 AI 结对编程防翻车工作流 (The Daily Grind)

_适用情况：写代码、改 Bug、提交 PR。_

1. **让 AI 写代码**： 在 Cursor 中按下 `Cmd + K` 或 `Cmd + I`，直接用简单的中文描述需求（比如：“增加一个价值网络的算子”）。你不需要再啰嗦“请用中文”、“请注意缩进”，AI 会自动读取隐藏规则。
    
2. **本地保存并触发安检**：
    
    Bash
    
    ```
    git add .
    git commit -m "feat: add value network"
    ```
    
3. **遭遇 Hook 拦截（最常见情况）**： 如果终端报出红字（比如 Ruff 发现未使用的 import，或者 Pyright 发现类型对不上），**不要自己去修**！
    
4. **让 AI 自己擦屁股**： 把终端里的红字报错完整复制，扔回给 Cursor：“过不了 pre-commit 安检，按报错修好它。”
    
5. **顺利通关**： 再次 commit 成功绿灯后，如果你有 `.github` 模板，AI 还能自动帮你按照 Markdown 模板生成完美的 PR 描述。