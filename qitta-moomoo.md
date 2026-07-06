我现在想参加https://qiita.com/official-events/750d1f37b7217167b1ad这个比赛，我已经用ai&这个站点
注意括号中很多和正文无关的内容是供你参考帮我润色的方向，不用写进正文
标题：（编程初心者🔰ok）使用aI&和codex配置好MooMoo skill进行全自动的股票盯盘与策略选择

前言（为什么这么做）：
1.（夸ai&因为这是他们举办的比赛）如果没有自己gpu，可以在ai&通过简单的配置进行api调用模型使用，这次从测试通api到安装好openD和moomoo skill,全程交由codex管理，安装部分大概花费不到0.15刀；
2.技术上来说ai&不仅支持chat/message的格式，更是支持response的格式（请你查阅资料帮我扩展性的夸一下，你可以参考https://docs.aiand.com/integrations/codex/）他们的文档；
3.MooMoo证券虽然是网络证券，但是除了交易手续费低一点，同比传统券商有很多方便的交易算法，比如oco二择一订单组和twap时间加权算法单和tsl跟踪止损限价单（注：这个最近乐天证券也追加了），和股票的实时公众指标，此外结合codex还有一个好处是可以让agent边给我们解释边构建我们最先最直觉的策略，然后不停的优化迭代；
4.moomoo之前的openD网关一直是用来获取分时段数据，现在在这个基础上开发了moomoo skill便于非程序员也可以快速接入实时的股票数据进行量化分析；（这里可以小小的介绍和夸一下moomoo，你自己发挥）
5.本次我们采用codex.app，虽然貌似不会被cc switch给识别到安装了codex（因为是通过系统指令codex来检查的），但是方便新手直接以web端的体验进行使用;
6.本次的操作不仅限于安装MooMoo skill，其他的skill可以以同样的操作交给codex完成。？
7.本教程的codex可以完美替换成claude code,opencode之类的软件，在安装的时候可能步骤有一定的区别。
# 安装codex app
*本次我们codex.app，可以直接拖动或者安装包安装，考虑到面向新手的体验，不实用cli，但是大致步骤是差不多的，不过codex.app是类似web chat的形式很多可以交互式控制而cli更像是在终端中进行，需要使用简单的指令进行配置和使用。*

前往 codex官方下载页面https://openai.com/ja-JP/codex/
选择 **macOS**（或 Windows）版本下载安装。
macos的话是使用dmg安装，注意自己的系统内核是arm还是intel
下载完后可以暂时不打开。第一次使用时，通常会提示：
- 授权访问本地项目（如果需要分析代码）
- 选择要打开的项目文件夹
简单选择home文件夹即可，或者新建一个moomoo文件夹。

### 2. 登录账号

如果您持有 **ChatGPT Plus / Team / Pro / Enterprise** 账号的话，请跳过下面的安装cc switch和

# 准备好ai&的api
*基本上和events说明界面一样,只是请注意api只会在创建的时候保存一次，注意复制*
*注意这里可以不用登陆信用卡，有个小按钮*
引用：
**ai& Inferenceの無料トライアル（クレジットカードの登録不要）**

1. [https://console.aiand.com](https://console.aiand.com/) にてユーザー登録
2. ホーム画面の右上にある「Credits:$0.00」をクリック
3. Enter Amountで$50を選択し、「Purchase」をクリック
4. プロモーションコードを追加で【unitabetai】を入力

# 安装cc switch
*ccswitch是用来将我们的自定义提供商ai&的api接入codex*
CC Switch 是一个 AI 编程工具管理器，可以统一管理 Codex、Claude Code、Gemini CLI 等工具的配置，无需手动修改配置文件。

## 第一步：下载 CC Switch

打开官网：

> [https://www.ccswitch.io/](https://www.ccswitch.io/)

或者 GitHub Releases 页面下载最新版本。

根据自己的系统选择安装包：

- **Windows**：`.msi`
- **macOS（Apple Silicon）**：`.dmg`
- **macOS（Intel）**：`.dmg`
- **Linux**：`.deb` 或 `.AppImage`

---

## 第二步：安装

### Windows

1. 双击下载好的 `.msi`
2. 一路点击 **Next**
3. 点击 **Install**
4. 安装完成后点击 **Finish**

---

### macOS

1. 双击下载好的 `.dmg`
2. 将 **CC Switch.app** 拖到 **Applications（应用程序）** 文件夹。
3. 第一次打开如果提示"无法验证开发者"，进入：

> 系统设置 → 隐私与安全性 → 仍要打开

再次打开即可。

---

## 第三步：启动

安装完成后：

- Windows：开始菜单搜索 **CC Switch**
- macOS：在 **应用程序** 中打开 **CC Switch**

首次启动时会自动创建本地配置目录。

---

## 第四步：确认安装成功

启动后，如果看到左侧出现类似下面的工具列表，就表示安装成功：

- Claude Code
- Codex
- Gemini CLI
- OpenCode（版本支持时）
- OpenClaw（版本支持时）

---
# 配置cc switch
插图，基本上找到codex的选项然后点击右上角的加号就能新生成一个配置了。基本上来说cc swtich只是帮助通过gui的方法修改config，所以其实手动去修改配置也行。此外、如果有自己的**ChatGPT Plus / Team / Pro / Enterprise**也可以在这里进行自由的切换

将如下的配置文件拷到最下方的详细配置指令中如图（插图），同时
这里我们使用的是glm5.2模型，
‘’‘ 
model = "zai-org/glm-5.2"
model_provider = "aiand"
web_search = "disabled"

[model_providers.aiand]
name = "ai&"
base_url = "https://api.aiand.com/v1"
wire_api = "responses"

[tools]
view_image = false

[features]
unified_exec = false
apps = false
browser_use = false
browser_use_external = false
computer_use = false
image_generation = false
multi_agent = false
in_app_browser = false
'''
然后将api拷到下图的api key的地方，注意这一步和ai&官方的步骤有出入，因为官方给的方法是针对
插图api
codex.cli的教程，我们这里使用app就是图简化，所以许多可以通过cc switch实现的gui的操作就没必要去改系统环境变量了。

然后配置成功，在cc switch点击启用该配置，接着重启codex即可使用。
注意：目前有bug，可以添加ai&的别的模型但是无法在codex中快速切换，所以最好需要别的模型的时候去cc switch里手动改前面提到的配置（model="zai-org/glm-5.2"）实现更换模型。
具体bug的复现可见https://youtu.be/v3fDWFRzS7E?si=V77FQVsNE-upcUVl
平时工作推荐一直使用glm5.2或者deepseek v4 pro即可。


# 安装MooMoo skill
*codex确认配置好之后即可让codex接管全部操作，此时只需要注意授权codex即可，同理此操作也适用于其他的skill*

来到官网
https://www.moomoo.com/ja/skillhub
然后复制给agent的prompt
"ガイドに従って Moomoo Skills をインストール：https://www.moomoo.com/skills/moomoo-install.md"
一直yes确认即可安装好需要的openD和skills，然后登陆MooMoo账号，在openD网关中显示已连接即可获取美股数据了，请注意自己是否有api权限，部分权限貌似需要一定的交易额或者直接购买。
codex还会自动删除安装过的dmg，这一点非常友好🧑‍🤝‍🧑。

注意：
codex默认会在你的启动位置进行部署，以及安装的sdk和各种需要做图的包都会默认安装在系统python环境中。
所以建议新建一个项目并使用uv创建一个新的环境进行包管理和回撤代码/策略保存，以便后续代码的进一步回撤测试和策略的迭代。（具体操作直接全部让codex执行）

注意2:
如果出现codex沙盒权限限制相关的问题，多半是因为你现在执行的文件夹codex没有权限操作，只需要在codex打开终端，macos自动就会自动跳出是否授权codex操作文件命令的权限，然后再让codex重试就可以跑通tcp连接了。只要有了权限，基本上出的所有问题codex都能帮忙解决。

最后，从安装到第一次确认苹果的股价和画出上一个交易日的均线大概用了0.4美元（glm5.2），所以我打算一直使用这个性能比较好的模型了。
