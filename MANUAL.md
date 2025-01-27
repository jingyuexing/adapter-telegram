# Telegram Adapter 使用指南

**此适配器还在施工中，遇到问题请尽快反馈！**

## 配置 Telegram Bot

### 申请一个 Telegram 机器人

首先你需要有一个 Telegram 帐号，添加 [BotFather](https://t.me/botfather) 为好友。

接着，向它发送`/newbot`指令，按要求回答问题即可。

如果你成功创建了一个机器人，BotFather 会发给你机器人的 token：

```plain
1234567890:ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHI
```

将这个 token 填入 NoneBot 的`env`文件：

```dotenv
telegram_bots =[{"token": "1234567890:ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHI"}]
```

如果你需要让你的 Bot 响应除了 `/` 开头之外的消息，你需要向BotFather 发送 `/setprivacy` 并选择 `Disable`。

## 配置 NoneBot

## 配置驱动器

NoneBot 默认的驱动器为 FastAPI，它是一个服务端类型驱动器（ReverseDriver），而 Telegram 适配器至少需要一个客户端类型驱动器（ForwardDriver），所以你需要额外安装其他驱动器。

HTTPX 是推荐的客户端类型驱动器，你可以使用 nb-cli 进行安装。

```shell
nb driver install httpx
```

别忘了在环境文件中写入配置：

```dotenv
driver=~fastapi+~httpx
```

## 使用代理

如果运行 NoneBot 的服务器位于中国大陆，那么你可能需要配置代理，否则将无法调用 Telegram 提供的任何 API。

```dotenv
telegram_proxy = "http://127.0.0.1:10809"
```

> NOTE 如果你的代理使用 socks 协议，你需要安装 httpx\[socks\]。

## 使用 Long polling 获取更新（推荐）

只要不在`env`文件中设置`url`，默认使用 long polling 模式。

## 使用 Webhook 获取更新

Telegram Bot 的 webhook 必须使用 https 协议，推荐使用 nginx 反向代理。

```conf
server {
        server_name tg.yourdomain.com;
        location / {
                proxy_pass http://127.0.0.1:4000;
                proxy_set_header Host $host;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        }
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/tg.yourdomain.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/tg.yourdomain.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
server {
    if ($host = tg.yourdomain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
        listen 80;
        server_name tg.yourdomain.com;
    return 404; # managed by Certbot
}
```

如果需要在本地调试，我推荐使用 VSCode 的 Remote - SSH 插件，或者使用 frp 再进行一次反向代理。

最后将域名填入`env`文件：

```dotenv
telegram_bots =[{"token": "1234567890:ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHI", webhook_url: "https://tg.yourdomain.com"}]
```

## 第一次对话


```python
import nonebot
from nonebot.adapters.telegram import Adapter as TelegramAdapter

nonebot.init()
app = nonebot.get_asgi()

driver = nonebot.get_driver()
driver.register_adapter(TelegramAdapter)

nonebot.load_builtin_plugins("echo")

if __name__ == "__main__":
    nonebot.logger.warning("Always use `nb run` to start the bot instead of manually running!")
    nonebot.run(app="__mp_main__:app")
```

现在，你可以私聊自己的 Telegram Bot `/echo hello world`，不出意外的话，它将回复你 `hello world`。

