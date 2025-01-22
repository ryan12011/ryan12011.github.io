当你正在调用 ChatGPT API 时，有没有想过它是如何进行计费的？通过官网查询得知，API 的计费策略采用**按使用量付费**，用多少扣多少。

![ChatGPT API 计费逻辑](https://cdn.apifox.cn/blog/2025/09/image-5.png)

但是问题来了，API 服务里的这个 **“按量计费” 的 “量” 应该怎么算？**

也许你会说，那还不简单，直接看它回答了多少字数呗！这话只对了一半。因为 ChatGPT 不仅会讲中文，它还可以根据使用者所发送的语言，回答出英语、法语、西语等多国语言。相同意思下每个语言的长度都会不一样，所以它会将这些内容统一换算为另一个标准单位——**Tokens**，然后再进行计费。

ChatGPT 不仅会统计自己输出了多少内容，还会记录用户输入了多少内容，毕竟它也需要先分析输入的文本才能输出有意义的回答。统一换算为 Tokens 值后汇总费用。以下是 ChatGPT API 的官方收费表，里面的 4K、8K 等数据代表单次问答中的最大上下文 Tokens 值。

![ChatGPT API 价格表](https://cdn.apifox.cn/blog/2025/09/image-7.png)

如果你已经通过 ChatGPT API 成功地将 AI 能力嵌入至商业化产品中，比如说打造出了一个对客提供服务的智能客服，那么作为开发人员的你**需要立刻、马上、清楚地知道每个客户的问答所产生的消耗额**。因为公司需要知道最基础的成本才能制定出合理的运营策略。

总不能依靠实时监控 OpenAI 的账号后台来查看所消耗的成本吧？（更何况它还有五分钟的延迟）

![实时监控成本](https://cdn.apifox.cn/blog/2025/09/image-9.png)

那怎样才能在使用 Apifox 调用 API 的过程中就直接看到输入和输出的人民币成本呢？通过以下 2 步就能够清楚地看到人民币花费：

> **1. 计算输入/输出 Tokens 值**  
> **2. 基于实时汇率换算人民币成本**

---

## 计算输入 / 输出 Tokens

### 创建换算脚本

将内容准确换算为 Tokens 需要引用第三方 Token 换算库。下面将以 [OpenAI GPT Token Counter 库](https://bit.ly/bewildcard)为例，使得能够在调试接口的过程中将输入/输出数据换算为 Tokens 数。

安装第三方库的运行环境：

bash
npm install openai-gpt-token-counter


参考以下示例代码，新建名为 `gpt-tokens-counter.js` 的 Node.js 脚本：

javascript
const openaiTokenCounter = require('openai-gpt-token-counter');

const text = process.argv[2]; // 获取命令行参数中的测试内容
const model = "gpt-4"; // 替换为你想要使用的 OpenAI 模型

const tokenCount = openaiTokenCounter.text(text, model);
const characterCount = text.length; // 计算字符数

console.log(`${tokenCount}`);


将该脚本文件放置于 Apifox 的**外部程序目录**下以供调用：

![外部程序目录](https://cdn.apifox.cn/blog/2025/09/image-10.png)

---

### 换算 Tokens 输入值

输入值就是在询问 ChatGPT 时所填写的问题。你可以在 “聊天消息” API 文档的**前置操作**中添加一个自定义脚本，使得 Apifox 能够调用上文中写好的 Tokens 换算脚本，提取位于请求参数 Body 中的 `content` 参数后换算为 Tokens 值。

![前置操作](https://cdn.apifox.cn/blog/2025/09/image-11.png)

填写以下示例代码：

javascript
try {
  var jsonData = JSON.parse(pm.request.body.raw);
  var content = jsonData.messages[0].content; // 获取 content 中的信息
  var result_input_tokens_js = pm.execute('./gpt-tokens/gpt-tokens-counter.js', [content]);
  console.log(content);
  pm.environment.set("RESULT_INPUT_TOKENS", result_input_tokens_js);
  console.log("Input Tokens count: " + pm.environment.get("RESULT_INPUT_TOKENS"));
} catch (e) {
    console.log(e);
}


点击 “运行” 按钮后可以在控制台中看到已统计的输入值。

![统计输入值](https://cdn.apifox.cn/blog/2025/09/image-12.png)

---

### 换算 Tokens 输出值

接下来需要计算 ChatGPT 输出了多少个 Tokens。

ChatGPT 采用 SSE（Server-Sent Events，服务器推送事件）输出回答，好处在于可以实时显示问题的答案。当用户看到 GPT 正在说话也就更有耐心的等下去。这种方式不像普通的 API 是单次的请求和返回，而是建立连接后输出一个连续的时间线。

因此我们需要先拼接所有的输出内容，然后再进行 Tokens 换算。

![拼接输出内容](https://cdn.apifox.cn/blog/2025/09/image-13.png)

在**后置操作**中添加以下自定义脚本，拼接所有输出内容并换算 Tokens：

javascript
// 获取响应的文本内容
const text = pm.response.text();
// 将文本内容分割成行数
var lines = text.split('\n');
// 创建一个空数组存储 "content" 参数
var contents = [];
// 遍历每一行
for (var i = 0; i < lines.length; i++) {
    const line = lines[i];
    // 跳过不以 "data:" 开头的行
    if (!line.startsWith('data:')) {
        continue;
    }
    // 尝试解析 JSON 数据
    try {
        var data = JSON.parse(line.substring(5).trim());  // 移除前面的 "data: "
        // 从 "choices" 数组中获取 "content" 参数并将其添加到数组中
        contents.push(data.choices[0].delta.content);
    } catch (e) {
        // 如果当前行不是有效的 JSON 数据，则忽略该行
    }
}
// 使用 join() 方法将 "content" 参数合并起来
var result = contents.join('');
// 在 "Visualize" 选项卡中显示结果
pm.visualizer.set(result);
// 将结果打印到控制台
console.log(result);

// 计算输出令牌的数量
var RESULT_OUTPUT_TOKENS = pm.execute('./gpt-tokens/gpt-tokens-counter.js', [result]);
pm.environment.set("RESULT_OUTPUT_TOKENS", RESULT_OUTPUT_TOKENS);

console.log("Output Tokens count: " + pm.environment.get("RESULT_OUTPUT_TOKENS"));


点击 “运行” 按钮后可以在控制台中看到已统计的输出值。

![统计输出值](https://cdn.apifox.cn/blog/2025/09/image-14.png)

---

## 基于实时汇率换算人民币成本

得到输入和输出所耗费的 Tokens 值后，因为 ChatGPT 采用美元计价，因此还需要调用实时汇率 API 后才能预估所消耗的人民币成本。

### 计算输入成本

在**前置操作**中新增以下脚本：

javascript
// 发送请求以获取 USD 到 CNY 的汇率
pm.sendRequest("http://apilayer.net/api/live?access_key=YOUR-API-KEY&currencies=CNY&source=USD&format=1", (err, res) => {
  if (err) {
    console.log(err);
  } else {
    const quotes = res.json().quotes;
    const rate = parseFloat(quotes.USDCNY).toFixed(3);
    pm.environment.set("USDCNY_RATE", rate); 
    var USDCNY_RATE = pm.environment.get("USDCNY_RATE");
    var RESULT_INPUT_TOKENS = pm.environment.get("RESULT_INPUT_TOKENS");

    const tokensExchangeRate = 0.03; // 每 1000 个令牌的价格（美元）
    const CNYPrice = ((RESULT_INPUT_TOKENS / 1000) * tokensExchangeRate * USDCNY_RATE).toFixed(2);

    pm.environment.set("INPUT_PRICE", CNYPrice); 
    console.log("Estimated cost: " + "￥" + CNYPrice);
  }
});


点击 “运行” 按钮后可以在控制台中看到预估的人民币成本。

![输入成本](https://cdn.apifox.cn/blog/2025/09/image-15.png)

---

### 计算输出成本

在**后置操作**中添加以下脚本：

javascript
pm.sendRequest("http://apilayer.net/api/live?access_key=YOUR-API-KEY&currencies=CNY&source=USD&format=1", (err, res) => {
  if (err) {
    console.log(err);
  } else {
    const quotes = res.json().quotes;
    const rate = parseFloat(quotes.USDCNY).toFixed(3);
    pm.environment.set("USDCNY_RATE", rate); 
    var USDCNY_RATE = pm.environment.get("USDCNY_RATE");
    var RESULT_OUTPUT_TOKENS = pm.environment.get("RESULT_OUTPUT_TOKENS");

    const tokensExchangeRate = 0.06; // 每 1000 个令牌的价格（美元）
    const CNYPrice = ((RESULT_OUTPUT_TOKENS / 1000) * tokensExchangeRate * USDCNY_RATE).toFixed(2);

    pm.environment.set("OUTPUT_PRICE", CNYPrice); 
    console.log("Output cost (CNY): " + "￥" + CNYPrice);
  }
});


点击 “运行” 按钮后可以在控制台中看到输出值的预估人民币花费。

![输出成本](https://cdn.apifox.cn/blog/2025/09/image-16.png)

---

### 预估总成本

最后还可以在后置操作中添加一个可以自动计算输入+输出总成本的步骤：

javascript
const INPUTPrice = Number(pm.environment.get("INPUT_PRICE"));
const OUTPUTPrice = Number(pm.environment.get("OUTPUT_PRICE"));

console.log("总成本：" + (INPUTPrice + OUTPUTPrice) + "元");


这样就能够在调试 ChatGPT API 的过程中预估出大致的输入输出总成本。

![总成本](https://cdn.apifox.cn/blog/2025/09/image-17.png)

---

现在你不仅可以在 Apifox 中愉快地调用 ChatGPT API，还能够清楚掌握你的调用成本，做到心中有数。

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)