
# 文章目录
1. 摘要
2. 如何将SNS通知转发到Telegram
2.1. 创建一个新的Telegram机器人。
2.2. 新建SNS主题
2.3. 新建Lambda函数
2.4. 发布函数
2.5. 让SNS主题和Lambada关联
2.6. 测试消息
3. 发送至不同Telegram

##  摘要
　　Amazon Simple Notification Service（SNS）是一项Web服务，允许从应用程序发布消息，然后立即将其传递给订阅者或其他应用程序。

发布者将消息发送到SNS主题，Amazon SNS服务将该消息（通知）传递给该SNS主题的订阅者。支持的通知协议是HTTP/S，SQS，Lambda，移动推送，电子邮件或SMS。

如果希望将通知传递到Telegram聊天中，则不能简单地使用HTTP/S端点将SNS主题与Telegram Bot API集成在一起。您需要创建一个简单的Lambda函数，该函数调用Bot API将通知转发到Telegram聊天。以下过程描述了如何进行。

## 如何将SNS通知转发到Telegram
　　在此过程中，您将创建一个Telegram bot。机器人是由软件而非人员操作的电报帐户。在我们的案例中，该机器人由Lambda函数操作，该函数代表该机器人将通知发送到Telegram聊天，通信是单向的。也就是说，该程序会向您发送消息，但不会处理您收到的任何消息。

要将SNS通知转发到Telegram聊天，请执行以下步骤：

## 创建一个新的Telegram机器人。
在您的Telegram应用中，搜索@BotFather并按下Start按钮（或发送/start命令）。然后，发送/newbot命令并遵循一些简单的步骤来创建一个新的Telegram机器人。BotFather为您的新机器人生成一个授权令牌。令牌是看起来像字符串123456789:ABCD1234efgh5678-IJKLM。需要将请求发送到Telegram Bot API。

在您的Telegram应用中，按名称搜索刚创建的机器人，然后按Start按钮（或发送/start命令）。然后，将任何测试消息写入与您的机器人聊天。例如，写Hello。

执行Bot API调用以获取与机器人聊天的ID。
在以下命令中，替换为从BotFather收到的授权令牌。


curl 'https://api.telegram.org/bot<token>/getUpdates' | python -m json.tool
在输出中，找到您的测试消息和相应的聊天ID。例如，在以下输出中，聊天ID为-478223748。


```css
{
  "ok": true,
  "result": [
    {
      "message": {
        "chat": {
          ...
          "id": -478223748,
          ...
        },
        ...
        "message_id": 2,
        "text": "Hello"
      },
      ...
    }
  ]
}
```
## 新建SNS主题
通过以下网址打开Amazon SNS控制台：https://console.aws.amazon.com/sns/home，然后在您选择的AWS区域中创建一个新的SNS主题。

## 新建Lambda函数
通过以下网址打开Lambda管理控制台：https://console.aws.amazon.com/lambda/home，然后切换到创建SNS主题的同一AWS区域。然后，使用以下配置创建一个新的Lambda函数。
运行时：Python 3.7
执行角色：创建一个新的Lambda IAM角色，该角色具有以下内联策略

```css
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*",
      "Effect": "Allow"
    }
  ]
}
```
功能代码：使用以下代码段
此函数执行Telegram Bot API的sendMessage方法以将SNS消息（通知）转发到Telegram聊天。


```python

import os
import json
from datetime import datetime, timezone, timedelta
from urllib import request, parse, error

MAX_CHUNK_SIZE = 4096

def getObject(msg, f):
    return msg.get(f)


def lambda_handler(event, context):
    url = 'https://api.telegram.org/bot%s/sendMessage' % os.environ['TOKEN']
    message = event['Records'][0]['Sns']['Message']
    null = 'null'
    msg = eval(message)

    now = datetime.now().astimezone(timezone(timedelta(hours=8)))
    Name = getObject(msg,'AlarmName')
    Time = getObject(msg,'StateChangeTime')
    Reason = getObject(msg,'NewStateReason')
    Region = getObject(msg,'Region')
    instance = getObject(getObject(getObject(msg, 'Trigger'), 'Dimensions')[0], 'value')

    chunk = '预警项目：%s\n实例名称：%s\n实例区域：%s\n预警时间：%s\n北京时间：%s\n预警内容：%s' % (Name, instance, Region, Time, now, Reason)

    data = parse.urlencode({"chat_id": os.environ['CHAT_ID'], "text": chunk})
    try:
        request.urlopen(url, data.encode('utf-8'))
    except error.HTTPError as e:
        print('Failed to send the SNS message below:\n%s' % chunk)
        response = json.load(e)
        if 'description' in response:
            print(response['description'])
        raise e
```

内存（MB）：128 MB
超时：15 sec

如果计划发布大型SNS消息，则可能需要将超时设置为30 sec甚至更高的值，以避免Lambda函数在将消息块转发到Telegram时超时。

环境变量：设置Lambda函数的CHAT_ID和TOKEN环境变量（使用步骤1中的值）
例如：


## 发布函数(代码上面的发布)
发布您的Lambda函数的新版本。然后，从页面顶部复制功能ARN（包括版本后缀）。

## 让SNS主题和Lambada关联
在Amazon SNS控制台中打开您的SNS主题。然后，点击您创建的主题，并且创建订阅，协议选择 AWS Lambda，选择刚刚发布的终端节点，使用上一步中的ARN为协议创建新的订阅。

测试消息
在Amazon SNS控制台中打开您的SNS主题。然后，发布测试消息。
该消息将传递给您与机器人的Telegram聊天。

## 发送至不同Telegram
Lambda函数的不同发布版本可以具有TOKEN和CHAT_ID环境变量的不同值。因此，要将SNS通知从一个SNS主题转发到不同的Telegram聊天，您可以使用同一Lambda函数的不同版本创建两个订阅。

