我现在想参加https://qiita.com/official-events/750d1f37b7217167b1ad这个比赛，我已经用ai&这个站点
注意括号中很多和正文无关的内容是供你参考帮我润色的方向，不用写进正文
标题：（编程初心者🔰ok）使用aI&和codex配置好MooMoo skill进行全自动的股票盯盘与策略选择

前言（为什么这么做）：
1.（夸ai&因为这是他们举办的比赛）如果没有自己gpu，可以在ai&通过简单的配置进行api调用模型使用，这次从测试通api到安装好openD和moomoo skill,全程交由codex管理，安装部分大概花费不到0.15刀；
2.技术上来说ai&不仅支持chat/message的格式，更是支持response的格式（请你查阅资料帮我扩展性的夸一下，你可以参考https://docs.aiand.com/integrations/codex/）他们的文档；
3.MooMoo证券虽然是网络证券，但是除了交易手续费低一点，同比传统券商有很多方便的交易算法，比如oco二择一订单组和twap时间加权算法单和tsl跟踪止损限价单（注：这个最近乐天证券也追加了），和股票的实时公众指标，此外结合codex还有一个好处是可以让agent边给我们解释边构建我们最先最直觉的策略，然后不停的优化迭代；
4.moomoo之前的openD网关一直是用来获取分时段数据，现在在这个基础上开发了moomoo skill便于非程序员也可以快速接入实时的股票数据进行量化分析；（这里可以小小的介绍和夸一下moomoo，你自己发挥）
5.本次我们采用codex.app，虽然貌似不会被cc switch给识别到安装了codex（因为是通过系统指令codex来检查的），但是方便新手直接以web端的体验进行使用
# 安装codex
*本次我们codex.app，可以直接拖动或者安装包安装，考虑到面向新手的体验，不实用cli，但是大致步骤是差不多的，不过codex.app是类似web chat的形式很多可以交互式控制而cli更像是在终端中进行，需要使用简单的指令进行配置和使用。*
### 1. 下载 ChatGPT Desktop

前往 OpenAI 官方下载页面：

> [https://chatgpt.com/download](https://chatgpt.com/download)

选择 **macOS**（或 Windows）版本下载安装。
macos的话是使用dmg安装，注意自己的系统内核是arm还是intel

### 2. 登录账号

如果您持有 **ChatGPT Plus / Team / Pro / Enterprise** 账号的话，请跳过下面的安装cc switch和

# 准备好ai&的api
*基本上和events说明界面一样,只是请注意api只会在创建的时候保存一次*


# 安装cc switch
*ccswitch是用来将我们的自定义提供商ai&的api接入codex*


