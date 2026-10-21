# 已保留的工具与插件

最后更新：2026-10-22

---

## dsh-video-kit

| | |
|---|---|
| 来源 | [shisuiqingqian/dsh-video-kit](https://github.com/shisuiqingqian/dsh-video-kit) v1.2.2 |
| 用途 | 把视频读成逐字稿 + 关键帧，喂给模型。B 站免登录可用，抖音要 Cookie |
| 装在哪 | 插件 `C:\Users\gj\.dsh\local-plugins\dsh-video-kit`（profile 里以 `file:` 依赖挂载） |
| 模型 | `C:\Users\gj\.dsh\toolkits\video-kit\models\`（Paraformer + Silero VAD，约 233 MB） |
| Python 依赖 | 装进 PATH 上的 Python 3.14：`yt-dlp` / `imageio-ffmpeg` / `sherpa-onnx` / `websocket-client` |

**坑**：它下载时把 format selector 写死成 `best[height<=720]/best`，碰上 B 站只有 DASH 分离流的新视频会报
`ERROR: [BiliBili] xxx: Requested format is not available`。
绕过办法是自己下到本地再把本地文件交给 `video_read`：

```powershell
python -m yt_dlp --no-warnings --no-playlist `
  -f "bv*[height<=720]+ba/b[height<=720]/bv*+ba/b" --merge-output-format mp4 `
  -o "<目录>\name.%(ext)s" <url>
```

**卸载**：删插件目录，再从 profile 的 `package.json` 依赖和 `cordis.patch.yml` 里去掉对应条目；模型目录可一并删。

---

## qq-bridge-enhanced

| | |
|---|---|
| 来源 | [h-D-ACE/dsh-qq](https://github.com/h-D-ACE/dsh-qq)（Derpyu520/qq-bridge 的二次开发版）v1.0.3 |
| 用途 | QQ ↔ DSH 桥接：QQ 私聊/群消息变成 DSH 会话，agent 的回复、提问、工具审批发回 QQ |
| 装在哪 | `C:\Users\gj\dsh-qq\qq-bridge-enhanced-v1.0.3` |
| 界面 | 控制中心 `http://127.0.0.1:3099`，桥接控制台 `http://127.0.0.1:3100` |

**关键配置**（都在 `config.json`，改动后要重启桥接）：

- `snowluma.homeDir` —— **留空会导致 token 自愈失败**：桥接靠扫 `<SnowLuma>/config/onebot_<QQ>.json` 拿 token，
  目录空就一个候选都找不到，只剩空 token 去撞 OneBot HTTP，必然 401。日志里那句「token 可能还没写进 SnowLuma」
  会把人往 SnowLuma 那边引，其实是桥接自己的配置缺了目录。
- `allow.groups` / `allow.private` / `ownerQQ` —— 三者任一为空时，机器人要么不回应任何人，要么管理命令全不可用。
- `ownerQQ` **不能填机器人自己的号**：owner 通道是「你私聊机器人」，机器人没法给自己发私聊。

**坑**：SnowLuma 的 HTTP token 和 WS token 是**两份不同的值**，都在 `<SnowLuma>/config/onebot_<QQ>.json` 里；
首次 WS 连接几乎必然 1006 失败一次然后自愈，属正常。

---

## SnowLuma

| | |
|---|---|
| 来源 | [SnowLuma/SnowLuma](https://github.com/SnowLuma/SnowLuma) v2.1.0（专有组件） |
| 用途 | 注入 QQ 客户端，对外提供 OneBot v11（WS 服务端 + HTTP API） |
| 装在哪 | `D:\SnowLuma-v2.1.0-win-x64` |
| 端口 | WebUI `127.0.0.1:5099`，OneBot HTTP `3000` / WS `3001` |

**注意**：EULA 禁止自动化部署，所以桥接只探测、不启停它 —— 必须手动双击 `launcher.bat`（或项目里的
`control-center-hidden.vbs`）并在 WebUI 里扫码登录。注入失败的表现是「WS 连着但收不到消息」。

---

## 本机环境备注

- **DSH 桌面版的 dsh 命令行**：真正能用的是
  `C:\Users\gj\AppData\Local\Programs\DeepSeek Harness\resources\runtime\cli\bin\dsh.cmd`。
  PATH 上那个 `C:\Users\gj\AppData\Local\deepseek-harness\bin\dsh.ps1` 原本指向一个已消失的目录（另一款桌面端生成的 shim），
  已改写成转发到上面这个真身。
- **桌面版 Web API 端口每次启动都随机**（本次是 `127.0.0.1:53482`），不是固定的 3080；qq-bridge 的
  `dsh.baseUrl` 留空即可自动识别。
- 沙箱里子进程的 schannel TLS 是坏的（`curl` / `Invoke-WebRequest` 都会失败），下载要用 node 的 `fetch`。

---

## 已卸载 / 未采用

| 东西 | 原因 |
|---|---|
| Hermes Agent（B 站教程里那条「语音+画图+唱歌」路线） | 是独立产品不是 DSH 插件，本地要 8G 显存 + 35G 硬盘，未采用 |
| 各种探针脚本、下载的 tar.gz/zip、抽帧图片、测试日志 | 纯中间产物，用完即删 |