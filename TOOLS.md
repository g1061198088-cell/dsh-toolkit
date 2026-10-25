# 已保留的工具与插件

最后更新：2026-10-25

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

## qq-media-mcp（自建：让大肥鱼开口说话、自己画图、唱歌）

| | |
|---|---|
| 来源 | 本机自建，`C:\Users\gj\dsh-qq\qq-media-mcp`（没有托管在 GitHub 上） |
| 用途 | 给 QQ 会话里的 agent 补上「自己造媒体」：`qq_say` 合成语音（再交桥接的 `qq_send_voice` 发）、`qq_draw` 画图并直接发到会话、`qq_sing` 唱歌（走本地 GPT-SoVITS） |
| 依赖 | Python 3.14 + `edge-tts` 7.3.1（装在 `qq-media-mcp\.venv`，走清华源）；Node + `@modelcontextprotocol/sdk` 1.32.2 |
| 语音库 | 桥接仓库的 `audio\`（从 `config.json` 的 `socialV2.voice.dir` 读） |

**为什么必须自建**：QQ 仿真会话的人格正文里写死了「你没有本地工具：不能执行命令、不能读写文件」，
所以 agent 自己跑不了 TTS；而 `qq_send_voice` 只会发**语音库里已有的文件**，不接受任意路径 / URL / base64。

**接线步骤**（`node install.mjs` 幂等做完前三条）：

1. `plugins\qq-agent-presets\qq-tool-restrict.mjs` 的 `SAFE_PREFIXES` 数组里加一行 `'mcp__qq-media__',`
   —— 桥接的执行期白名单只放行 `mcp__snowluma__` / `mcp__snowluma-host__` / `mcp__web-search-safe__`，
   不加这一行，新工具有注册但会被执行期拦掉。
2. `C:\Users\gj\.dsh\profiles\desktop\cordis.patch.yml` 末尾追加一个 `@deepseek-ai/dsh-mcp-client` 注册块
   （`serverName: qq-media`）。node 要写 `D:\Program Filesnodejs\node.exe` —— 那个目录名真的长这样，
   `D:\Program Files\nodejs\node.exe` 是不存在的。
3. `config.json` 里 `socialV2.voice.enabled` 与 `socialV2.tools.sendVoice` 都改 `true`，然后重启桥接。
4. **重启一次 DeepSeek Harness 桌面端**，`qq_say` 才会进入 agent 的工具表。

**画图（`qq_draw`）**：桥接**没有**任何「发图片」的工具，也没有对应路由（`/api/socialV2/` 下只有 send-voice /
send-sticker），所以 `qq_draw` 生成完自己借 SnowLuma 的 OneBot HTTP 直发：

```
POST http://127.0.0.1:3000/send_group_msg   （或 send_private_msg）
Authorization: Bearer <桥接 config.json 的 snowluma.accessToken>
{ "group_id": <群号>, "message": [{ "type": "image", "data": { "file": "base64://<jpeg>" } }] }
```

因为绕过了桥接，安全性由它自己兜：`authorize()` 拿 `state\social-v2.json` 里该会话的 `agentToken` 比对
（只能发到自己那个会话）+ 白名单校验，`checkDrawRate()` 限流（每会话 4 张/小时、全天 30 张），
用量记在 `state\qq-media-draw.json`，图片存在 `state\draw\`。
生成走 **image.pollinations.ai**（免费免 key，实测 768×768 约 8 秒、~47 KB）。**代价：提示词会发到境外服务**，
所以工具描述和人格卡里都写明「别把群里对话原文抄进 prompt」。

**音色**：`xiaoyi`（默认，活泼卡通）/ `xiaoxiao` / `yunxi` / `yunjian` / `yunxia` / `yunyang` /
`xiaobei`（东北话）/ `xiaoni`（陕西话）。

**人格卡**：`dsh\agent-presets\qq-chat-v2\agent.cordis.yml` 第 40 行那条「发语音」被改写过，
补了「本机管理员已打开语音并额外装了 qq_say」。改完必须重跑
`node scripts\build-agent-preset-patches.mjs` 重新生成 `plugins\qq-agent-presets\presets\qq-chat-v2.patch.yml`。
改写时**故意保留了契约测试断言的三段原文**（`语音**默认是关闭的**`、`verified=true 才代表这条语音确实落地`、
`不要假装发了语音`）——改测试会让以后升级桥接时冲突。

**验证语音链路**（不用等 agent，走操作者通道直接试发）：

```powershell
$b="C:\Users\gj\dsh-qq\qq-bridge-enhanced-v1.0.3"
$tok=(Get-Content "$b\state\console-token" -Raw).Trim()
Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:3100/api/voice/send" `
  -Headers @{'x-console-token'=$tok} -ContentType 'application/json' `
  -Body '{"target":"private:1061198088","file":"<audio 里的 mp3>","verify":true}'
```

返回 `verified: true` 才算真落地（桥接会回读那条 record 段）。

**卸载**：删 `C:\Users\gj\dsh-qq\qq-media-mcp`，再去掉 `cordis.patch.yml` 的注册块、`SAFE_PREFIXES` 里那行，
并把 `config.json` 的两个开关关回去 —— `install.mjs` 是单向的，不能反做。

---

## GPT-SoVITS（自建：让大肥鱼唱歌）

| | |
|---|---|
| 来源 | 上游 [RVC-Boss/GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS)（MIT），代码在 `C:\Users\gj\dsh-qq\sing\src\GPT-SoVITS-main` |
| 入口 | MCP 工具 `qq_sing` / `qq_sing_refs`，实现在 `qq-media-mcp\server.mjs` |
| 用途 | 零样本音色克隆：给一段参考音频 + 歌词，本地 GPU 现唱，产物 ogg 进桥接语音库，再 `qq_send_voice` 发 |
| 运行环境 | `C:\Users\gj\dsh-qq\sing\venv`（Python **3.12.14**，用 DSH 桌面端自带的 3.12 建的，没碰应用运行时） |
| torch | **2.7.1+cu128** / torchaudio 2.7.1+cu128（RTX 5060 Laptop 是 Blackwell sm_120，必须 cu128） |
| 模型 | ModelScope `XXXXRT/GPT-SoVITS-Pretrained`，共约 4.9 GB，解压到仓库的 `GPT_SoVITS\` |
| API | 本地 `http://127.0.0.1:9880`（`api_v2.py`），由 MCP 工具按需 detached 拉起 |

**踩过的坑（都记下来，重装时别再来一遍）**：

- **镜像**：aliyun 的 pytorch-wheels 对 torch 限速 0.47 MB/s；用 `https://mirrors.nju.edu.cn/pytorch/whl/cu128/`
  （7.5 MB/s）。torch 装好后，其余依赖走
  `-i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements-py312-win-flash_attention\requirements_py312_win_cu128_china.txt`
  （注意**目录名是连字符**，不是下划线）。
- **`api_v2` 只支持 `wav` / `raw` / `ogg` / `aac`，不支持 mp3**（`api_v2.py:404` 直接 400
  `media_type: mp3 is not supported`）。所以唱歌产物是 ogg。
- **必须用 `custom` 档、且在仓库根目录启动**：`tts_infer.yaml` 里的模型路径是相对仓库根的。
- **判断下载进度别用 `dir` / `Get-ChildItem`**：正在写的文件目录项长度不刷新，会一直显示 0 字节；
  用 node 的 `fs.fstatSync(fd).size` 看真值。
- **参考音频决定一切**：音色和「唱腔」都来自参考。默认 `C:\Users\gj\dsh-qq\sing\ref\ref_xiaoyi.wav`
  （9.96 秒 / 32 kHz 单声道，用 edge-tts 的晓伊声线合成，和 `qq_say` 默认音色一致）。
  要换音色、或想唱得更像唱，把带旋律的参考音频路径 + 对应文本写进 `C:\Users\gj\dsh-qq\sing\refs.json`。
- **冷启动约 20 秒**（加载权重），之后每次 2~3 秒。API 进程由工具用 `detached: true` 拉起，不随 DSH 会话死。
- 参考音频注册表：`refs.json`，字段 `audio` / `text`（参考音频的原话）/ `lang` / `desc`。

**卸载**：删 `C:\Users\gj\dsh-qq\sing`（约 12 GB：venv 5.7 GB + 模型 4.9 GB + wheel 缓存），
再把 `server.mjs` 里的 `qq_sing` / `qq_sing_refs` 工具与 `ensureSingServer` 摘掉即可。

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