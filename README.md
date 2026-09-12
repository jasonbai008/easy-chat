# Easy Chat

基于 [Gemini 3.5 Live Translate](https://ai.google.dev/gemini-api/docs/live-api) 的浏览器同声传译。打开页面、点「开始传译」，麦克风里的话会实时识别、翻译，并同步播出译音。

无需构建、无需登录。静态页面通过 WebSocket 连到 Live 代理，即可工作。

## 功能

- **实时同传**：边说边译，原文 / 译文双栏同步滚动
- **译音回放**：模型返回 24 kHz PCM，浏览器排队播放
- **自动识别或指定语种**：源语言可选「自动识别」，也可锁定一种语言
- **双向对谈**：源语言不是自动时，连续两次识别到对方语言会自动对调译向并重连
- **语言交换**：一键对调识别语与译入语；自动识别模式下会用最近检测到的语种补位
- **80+ 语种**：含中英日韩法西德阿俄葡等；常用语种置顶，支持搜索
- **会话计时**：传译中显示已连接时长
- **字幕操作**：复制原文 + 译文，或一键清空
- **保持亮屏**：优先使用 Screen Wake Lock；不支持时用静音视频兜底
- **设置记住**：语言和自定义 API 地址写入 `localStorage`

## 快速开始

用任意静态服务器打开即可（需 HTTPS 或 `localhost`，否则浏览器会拦截麦克风）：

```bash
# Python
python -m http.server 8080

# 或 Node
npx serve .
```

浏览器访问 `http://localhost:8080`，允许麦克风后点「开始传译」。

也可把仓库部署到 GitHub Pages、Cloudflare Pages、Nginx 等任意静态托管。

## 使用方式

1. 左侧选**识别语言**（默认自动识别），右侧选**译入语**（默认简体中文）
2. 点中间 `⇄` 可交换译向
3. 点底部「开始传译」，对着麦克风说话
4. 左侧出原文，右侧出译文，同时听到译音
5. 再点一次按钮结束；也可按空格开始 / 结束（焦点不在输入框或按钮上时）

状态栏会显示：离线 / 连接中 / 传译中 / 重连中 / 切换中，以及会话计时。

## 两种模式

| 源语言 | 行为 |
| --- | --- |
| 自动识别 | 单向：任意语种 → 译入语。适合听讲、看片、听对方说话 |
| 指定语种 | 双向：锁定「我说 A、对方说 B」。连续两次识别到 B，会自动改成「识别 B、译成 A」 |

源语言与译入语不能相同。若选成一样，会自动改成另一侧，或把源语言退回自动识别。

## 自定义代理

默认连 `https://live.jasonbai.dpdns.org`，WebSocket 路径为 `/live`。换自己的代理：

```
https://你的域名/?api=https://your-live-proxy.example
```

`api` 参数会覆盖并写入本地设置。代理需实现 Gemini Live 的 WebSocket 协议：

1. 客户端连接 `ws(s)://{origin}/live`
2. 发送 `setup`（模型 `gemini-3.5-live-translate-preview`，`responseModalities: ["AUDIO"]`，带 `translationConfig`）
3. 持续发送 16 kHz PCM（`audio/pcm;rate=16000`）
4. 接收 `setupComplete`、输入 / 输出转写、以及 `inlineData` 译音

## 项目结构

```
easy-chat/
├── index.html      # 页面与全部交互逻辑
├── styles.css      # 深色、移动端优先样式
├── languages.js    # 语种列表
├── favicon.svg
└── README.md
```

没有框架、没有打包。逻辑都在 `index.html` 的脚本里。

## 浏览器要求

- 现代 Chromium / Safari / Firefox
- 允许麦克风
- 支持 WebSocket、Web Audio、`getUserMedia`
- 建议 HTTPS；移动端 Safari 需用户手势后才能播放译音

麦克风会开启回声消除、噪声抑制和自动增益。输入降采样到 16 kHz 再发给模型。

## 本地设置

键名：`easyTalkTranslate`

```json
{
  "sourceLang": "auto",
  "targetLang": "zh-Hans",
  "apiOrigin": "https://live.jasonbai.dpdns.org"
}
```

## 快捷语种

语言面板顶部固定：自动识别、简中、繁中、英、日、韩、法、西、德、阿、俄、巴西葡、意、泰、越、印地。完整列表见 `languages.js`。

## 许可

按仓库现状使用。Gemini API 与代理的配额、计费以对应服务为准。
