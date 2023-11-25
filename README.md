<p align="center">
  <img alt="Web client demo" src="./demos/client.png?v=3">
</p>

# PandoraAI

PandoraAI是一个web聊天客户端，由 [node-chatgpt-api](https://github.com/waylaidwanderer/node-chatgpt-api) 提供支持，允许用户轻松地与多个AI系统聊天，同时还提供自定义预置支持。凭借无缝和方便的设计，PandoraAI提供了引人入胜的对话式AI体验。

使用 Vue 3 框架 [Nuxt 3](https://v3.nuxtjs.org/) 构建。
只要API是兼容的，你也可以将PandoraAI与其他API服务器实现一起使用。

## 特性

- 与 `node-chatgpt-api` 支持的所有人工智能聊天，包括 `gpt-3.5-turbo`, `text-davinci-003`, ChatGPT, 和 Bing.
- 支持为每个客户端创建多个预设。
![Client Settings](demos/client-settings.png) 
- 选择不同的客户端或自定义预设。
![Client Dropdown](demos/client-dropdown.png)
- 所有内容都存储在本地，因此您可以在没有帐户的情况下使用该客户端，并且可以将其导入或导出到其他设备。

<details>
<summary><strong>Nuxt 3 安装</strong></summary>

查看 [Nuxt 3 文档](https://nuxt.com/docs/getting-started/introduction) 了解更多信息。

## 安装

确保安装了依赖项:

```bash
# yarn
yarn install

# npm
npm install

# pnpm
pnpm install
```

## 开发服务器

在 http://localhost:3000 上启动开发服务器

```bash
npm run dev
```

## 生产

为生产构建应用程序:

```bash
npm run build
```

本地预览产品构建:

```bash
npm run preview
```

查看 [部署文档](https://nuxt.com/docs/getting-started/deployment) 了解更多信息。
</details>

## 安装

1. 遵循上面的Nuxt 3安装说明。
2. 运行API服务器 [node-chatgpt-api](https://github.com/waylaidwanderer/node-chatgpt-api#api-server).
3. 复制 `.env.example` 为 `.env` 然后在 `API_BASE_URL` 变量中填写API服务器的URL。
4. 运行 `npm run dev` 启动开发服务器，或者 `npm run build` 构建用于生产的应用程序。
  1. 如果你在拉取最新的修改后看到一个空白的页面，先运行`nuxi upgrade --force`，然后再运行`npm run dev`。

## 贡献
如果您想为这个项目做出贡献，请创建一个拉请求，并详细描述您的更改。

## 协议
这个项目是根据MIT许可证授权的。
