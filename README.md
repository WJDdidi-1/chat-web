# chat-web

一个用 **Node.js + Express + Socket.IO** 写的实时聊天室：浏览器打开页面、输入昵称、
就能看到在线名单和所有人的消息。没有账号系统，也没有数据库——消息只在内存里转发。

## 目录

```
chat-app/
  server.js         Express + Socket.IO 服务（/health 健康检查）
  public/           前端页面
  package.json      start / dev 脚本
  .env.example      PORT / HOST / ALLOWED_ORIGINS 示例
```

## 怎么跑

```console
$ cd chat-app
$ cp .env.example .env      # 按需修改，三项都是可选的
$ npm install
$ npm start                 # http://localhost:3000
```

`npm run dev` 用 nodemon 热重启。

## 事件协议

| 方向 | 事件 | 负载 |
| --- | --- | --- |
| 客户端 → 服务端 | `user join` | 昵称字符串 |
| 客户端 → 服务端 | `chat message` | 消息字符串 |
| 服务端 → 客户端 | `user joined` | `{username, users[]}` |
| 服务端 → 客户端 | `chat message` | `{username, message, timestamp}` |
| 服务端 → 客户端 | `user left` | `{username, users[]}` |

没有登录态：`socket.id` 到昵称的映射放在一个 `Map` 里，进程重启即清空。

## 这一版修掉的两件事

1. **CORS 配置**：原来是 `origin: "*"` 配 `credentials: true`。浏览器会拒绝这个
   组合（通配符不能与凭证同时出现），同时它也意味着任意站点都能连上来。现在改成
   由 `ALLOWED_ORIGINS` 决定来源，留空即只允许同源。
2. **仓库卫生**：`node_modules/`（989 个文件）和 `.env` 之前被提交进了仓库。现在都
   移出版本控制，并加了 `.gitignore` 与 `.env.example` —— `.env` 里只有
   `PORT`/`HOST`/`ALLOWED_ORIGINS`，没有密钥，但配置本来就不该进仓库。

## 部署注意

`vercel.json` 与 `module.exports = app` 说明这套代码曾按 Vercel Serverless 部署。
Serverless 函数不会长驻，Socket.IO 的长连接因此不能依赖它——真要部署，需要一台常驻
进程（或换成托管 WebSocket 服务）。本地与容器里运行没有问题。

## 许可

`package.json` 声明 MIT，但仓库没有 LICENSE 文件；补一份 LICENSE 才算完整。
