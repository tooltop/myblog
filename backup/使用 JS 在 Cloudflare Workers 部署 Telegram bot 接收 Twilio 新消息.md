
在 Cloudflare Workers 上部署一个简单的 Telegram 机器人，该机器人可以接收从 Twilio 号码发送的短信，并将消息转发到指定的 Telegram 聊天中。

### 准备工作

1. 一个 Telegram Bot Token: 
   -  打开 Telegram 并搜索 "@BotFather"。
   -  给 BotFather 发送消息 /newbot  并按照指示操作，创建一个新的 Telegram 机器人。
   -  BotFather 会给你发送一串字符，这就是你的 TELEGRAM_BOT_TOKEN，请妥善保管。

2. 一个 Twilio 账号: 
   - 注册 Twilio  并获取你的  Account SID 和 Auth Token。
   - 找到并记录下你的 Twilio 电话号码。

3. 一个 Cloudflare Workers 账号: 
   -  注册 Cloudflare Workers  并登录。

4. 获取 Telegram Chat ID:
   - 将你的 Telegram 机器人添加到一个 Telegram 群组，或者直接与你的机器人私聊。
   - 在浏览器中访问以下网址，并将 YOUR_TELEGRAM_BOT_TOKEN 替换为你的机器人 Token:
      https://api.telegram.org/botYOUR_TELEGRAM_BOT_TOKEN/getUpdates
   -  返回的 JSON 数据中包含 "chat" -> "id" 字段，这就是你的 TELEGRAM_CHAT_ID，记录下来。 

### 步骤

1. 创建 Cloudflare Workers 函数:
   - 登录 Cloudflare Workers 并创建一个新的 Worker。
   -  将以下代码粘贴到 Worker 代码编辑器中：

``` JS
     (() => {
  // index.js
  CHAT_ID = globalThis.TG_CHAT_ID;
  TOKEN = globalThis.TG_TOKEN;
  var Handler = class {
    async send(req) {
      const formData = await req.formData();
      const body = {};
      for (const entry of formData.entries()) {
        body[entry[0]] = entry[1];
      }
      const msg = {
        chat_id: CHAT_ID,
        text: "From: " + body["From"] + "\r\nT   o  : " + body["To"] + "\r\n==== Message: ====\r\n" + body["Body"]
      };
      const init = {
        body: JSON.stringify(msg),
        method: "POST",
        headers: {
          "content-type": "application/json;charset=UTF-8"
        }
      };
      let url = "https://api.telegram.org/bot" + TOKEN + "/sendMessage";
      console.log(await fetch(url, init));
      return new Response("", { status: 200 });
    }
  };
  var handler = new Handler();
  addEventListener("fetch", (event) => {
    event.respondWith(handler.send(event.request));
  });
})();
//# sourceMappingURL=index.js.map
```

3. 设置 Cloudflare Workers 环境变量:
   - 在 Cloudflare Workers 代码编辑器中，点击左侧边栏的 "Settings"  标签页。
   - 选择 "Variables"  选项卡。
   - 点击 "Add variable"  按钮。
   - 在 "Variable name" 字段中输入 TELEGRAM_BOT_TOKEN，并在 "Variable value" 字段中粘贴你的 Telegram Bot Token。
   - 再次点击 "Add variable"  按钮，并以相同的方式添加  TELEGRAM_CHAT_ID 环境变量，并将你的 Chat ID填入值中。

4. 部署 Cloudflare Workers 函数:
   - 点击 Cloudflare Workers 代码编辑器顶部的  "Save and Deploy"  按钮。


5. 更新 Twilio Webhook URL:
   -  复制你刚刚部署的 Cloudflare Workers 函数的 URL。
   - 找到你的 Twilio 电话号码并找到 Webhook 配置的地方（通常在 "Messaging" 或类似的选项下）。
   - 创建一个新的 Webhook，将 Webhook URL  更新为你的 Cloudflare Workers  函数的 URL。
   -  保存 Webhook 配置。 

### 测试:
* 向你的 Twilio 号码发送短信。 
* 你的 Telegram 机器人应该会将消息转发到你指定的 Telegram 聊天中。 

🎉🎉🎉 恭喜！你已成功部署了一个可以接收 Twilio 短信并转发到 Telegram 的机器人！