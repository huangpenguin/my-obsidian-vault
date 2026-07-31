---
title: "microsoft账号问题(使用tenant模式登陆)"
publish: false
tags: ["项目实践"]
---
# microsoft账号问题(使用tenant模式登陆)

---

az login --tenant cb2a3039-b4df-4f67-9bc5-0b4dd92e36e6
A web browser has been opened at [https://login.microsoftonline.com/cb2a3039-b4df-4f67-9bc5-0b4dd92e36e6/oauth2/v2.0/authorize](https://login.microsoftonline.com/cb2a3039-b4df-4f67-9bc5-0b4dd92e36e6/oauth2/v2.0/authorize). Please continue the login in the web browser. If no web browser is available or if the web browser fails to open, use device code flow with `az login --use-device-code`.

Retrieving subscriptions for the selection...

[Tenant and subscription selection]

No     Subscription name    Subscription ID                       Tenant

---

[1] *  pjt-interviewer-sbg  1d52fb41-7a09-4d7e-9f00-e9d32a106075  cb2a3039-b4df-4f67-9bc5-0b4dd92e36e6

The default is marked with an *; the default tenant is 'cb2a3039-b4df-4f67-9bc5-0b4dd92e36e6' and subscription is 'pjt-interviewer-sbg' (1d52fb41-7a09-4d7e-9f00-e9d32a106075).

本质上你的 Gmail 邮箱目前是以“外来访客”身份存在的。普通的 `az login` 就像是在敲一扇没有名字的大门，而 `--tenant` 则是告诉系统：“我要去敲 cb2a3039 这家公司的门”。

**试试看 `--tenant` 参数，通常这一步就能解决你的所有授权问题。**
