---
title: "gcp cli"
publish: false
tags: ["LLM","项目实践"]
---
# gcp cli

## Google Cloud CLI 初始化与配置

初始化 Google Cloud CLI，登录账号并设置默认项目、区域和区域。

```bash
gcloud auth login
```

再次进行身份验证（在浏览器中登录 Google 账号）。

```bash
gcloud config set project pjt-aks
```

设置默认的 GCP 项目为 `pjt-aks`。

```bash
gcloud config configurations list
```

查看当前的配置，包括活跃账户、项目、默认区域和区域设置。

---

## 🖥️ SSH 登录 GCP 虚拟机（Compute Engine）

```bash
gcloud compute ssh ubuntu@pjt-aks-huang --zone=us-central1-f -L 5555:localhost:5555

(增加安全性设置)gcloud compute ssh pjt-aks-huang --project=pjt-aks --zone=us-central1-f --tunnel-through-iap -- -L 5555:localhost:5555
```

通过 SSH 登录到名为 `pjt-aks-huang` 的虚拟机（需提前配置好实例和 SSH 密钥，若无则自动生成）。这里的-L是设置了本地和VM之间的转发。

> 这里其实还可以通过添加选项来实现,测试和调试连接到 GCP VM 上的 Ollama 服务如下
> 

---

## 🔑 SSH 密钥相关提示（首次连接）

当你第一次运行 SSH 命令时系统会提示：

- 是否生成 SSH 密钥对；
- 是否信任并缓存服务器的公钥；

按照提示输入 `y` 即可完成密钥生成和连接。

---

## GCP 与 SSH 的关系

- **SSH（Secure Shell）** 是一个通用的远程登录和端口转发协议，用来在不安全的网络上安全地访问远程机器。
- **GCP VM** 本质上是部署在 Google 数据中心里的 Linux 实例，你可以把它看成任何一台云端服务器。要登录或做隧道，就用 SSH。
- `gcloud compute ssh` 是 Google Cloud SDK 提供的一个包装器，底层也是调用 `ssh`：
    - 自动帮你管理 SSH 密钥
    - 集成了 **IAP（Identity-Aware Proxy）**，让你能在没有公网 IP 或者不想打开防火墙的情况下，也能安全地通过 Google 验证后通道到 VM。
- 同理，`gcloud compute start-iap-tunnel` 也是在背后启动了一条 SSH 隧道，只不过它专门做端口转发，不开 shell。

---
