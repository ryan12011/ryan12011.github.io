## 一、ChatGPT API 简介

OpenAI 最近推出了收费计划，开放了 ChatGPT 和 Whisper API。ChatGPT API 允许企业或个人在其应用程序、网站、产品和服务中集成 ChatGPT 功能，而 Whisper API 则支持语音转文本功能。

ChatGPT API 的价格为 $0.002/1K tokens，性价比极高。它的开放类似于当年 Apple 应用商店的建立，为开发者提供了强大的人工智能接口。

## 二、ChatGPT 账号申请与充值

如果你还没有 OpenAI 账号，可以通过接码平台和虚拟信用卡（如 Depay）完成注册，成本不到 1 美元。完成注册后，登录 OpenAI 网站，进入 **Billing** 页面，绑定虚拟信用卡即可完成充值。

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)

## 三、如何使用 ChatGPT API

完成账号注册后，进入 OpenAI 平台，点击 **View API keys**，生成属于你的 API Key。请注意，API Key 只会显示一次，务必妥善保存。

### 使用方法

1. **Token 使用量监控**：在 **Usage** 页面可以查看 token 的使用情况，数据每 5 分钟更新一次。新注册用户可获得 $18 的免费额度，但需注意有效期。
2. **简单测试**：可以通过 curl 命令测试 API，替换 `$OPENAI_API_KEY` 为你的 API Key。
3. **编程语言支持**：官方提供了多种语言的示例代码，例如 Python，只需导入 `openai` 包并使用最新的 `gpt-3.5-turbo` 模型即可。

更多示例代码请参考 [官方文档](https://platform.openai.com/docs/api-reference/introduction)。

## 四、ChatGPT API 常见问题

### 1. GPT-3.5-turbo 模型

GPT-3.5-turbo 是目前 OpenAI 提供的最强大且性价比最高的文本生成模型，其成本仅为 text-davinci-003 的十分之一。

### 2. API 响应速度

ChatGPT API 的响应速度比官方 ChatGPT 更快，适合需要高效处理的场景。

### 3. Token 收费标准

API 的收费标准为 $0.002/1K tokens。需要注意的是，连续会话需要回传历史消息，这会增加 token 消耗。

### 4. 连续会话能力

API 支持上下文连续对话功能，但需要在接口参数中传递历史对话内容，单次传递的 token 上限为 4096。

### 5. 地区限制

目前国内访问 API 可能存在不稳定的情况，建议使用国际 IP。

## 五、Whisper API 简介

Whisper API 是 OpenAI 推出的语音转文本服务，基于开源的 whisper-large-v2 模型，价格为每分钟 $0.006。它支持多种音频格式的转录和翻译功能。

## 六、ChatGPT Plus 是否还有必要续费？

ChatGPT API 更适合开发者，而 ChatGPT Plus 对普通用户更友好。API 的 token 消耗和上下文限制使其成本并不一定低于 Plus 的 $20 定价。此外，Plus 的回答质量和模型参数优化仍然占优。

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)