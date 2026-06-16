# AstrBot 悟声 TTS 插件 (wusound_tts)

## 项目描述

这是一个为 [AstrBot](https://github.com/Soulter/AstrBot) 框架开发的文本转语音（TTS）插件。
当 AstrBot 的 AI 生成短回复时，插件会自动拦截文本，调用当前会话的 LLM 将其翻译为自然口语的日语，并请求 **悟声 AI** 接口实时生成日语音频，最后以文件或语音气泡的形式发送到群聊或私聊中。

---

## 具体功能

- **🤖 自动日文翻译**：内置经过优化的严格英文翻译提示词，配合多层后处理逻辑，智能提取纯日语进行语音生成。
- **⏱️ 短回复拦截**：支持设置 Token 阈值，只有短回复才会触发语音，避免长篇大论拖慢响应或刷屏。
- **🛡️ 双层权限控制**：提供「群聊过滤」与「用户过滤」两层独立控制，均支持 `whitelist`（仅放行列表）、`blacklist`（屏蔽列表）与 `none`（无限制）模式。
- **🗣️ 多种发送模式**：支持以 `file`（音频文件）或 `record`（原生语音）两种模式发送。`record` 模式会自动将远程音频下载到本地缓存后再发送，提高兼容性。
- **🛠️ 零消耗调试模式**：内置 Mock 模式，不消耗悟声 API 额度和大模型 Token 即可测试发送链路。
- **🔍 丰富的诊断命令**：提供多条专属命令用于排查文本污染、查询会话 ID 等。

---

## 安装与操作

### 1. 安装步骤

1. 进入 AstrBot 的插件目录：
   ```bash
   cd data/plugins
   ```
2. 克隆本仓库：
   ```bash
   git clone https://github.com/linglinlinglin/AstrBot-TTS-.git astrbot_plugin_wusound_tts
   ```
3. 在 AstrBot 管理面板重启框架或重载插件。

### 2. 基础配置

在 AstrBot 的 WebUI 配置页面找到 `wusound_tts` 插件，填写以下核心配置：

- **api_key**: 悟声开发者中心申请的 API Key（也可在运行环境设置 `WUSOUND_API_KEY` 变量）。
- **voice_id**: 悟声语音角色 ID。
- **send_as**: 选择发送格式（`file` 或 `record`）。推荐先用 `file` 测试链路，跑通后再尝试改为 `record`。

### 3. 权限配置（白名单/黑名单）

插件支持灵活的**群聊过滤**和**用户过滤**，两者是**叠加关系**（必须同时通过两层校验，插件才会工作）。

- **群聊过滤 (`group_filter_mode`)**: 
  - 可选 `whitelist` / `blacklist` / `none`。
  - 在 `group_filter_list` 中填写群号或完整会话 ID（如 `123456` 或 `onebot:GroupMessage:123456`）。
- **用户过滤 (`user_filter_mode`)**: 
  - 可选 `whitelist` / `blacklist` / `none`。
  - 在 `user_filter_list` 中填写允许或屏蔽的用户 ID。

> **注意**：如果任一层配置为 `whitelist` 但其列表为空，该层过滤将不会放行任何请求！

### 4. 调试命令（仅管理员可用）

在聊天框中发送以下命令，帮助你快速调试和配置：

- `/wusound_where`：获取当前聊天的 `group_id`、`user_id` 和完整会话 `origin`，并显示权限是否放行，用于填写过滤名单。
- `/wusound_preview`：预览 LLM 翻译结果，**不调用悟声 API**。用于排查输出文本是否混入了中文杂质或提示词是否对当前大模型失效。
- `/wusound_test`：用当前 `send_as` 配置发送一条测试音频。
- `/wusound_file_test`：强制以**文件**形式发送测试音频，用于排查文件链路。
- `/wusound_record_test`：强制以**语音**形式发送测试音频，用于排查底层协议端是否支持原生语音。

---

## 常见问题 (FAQ)

- **Q: 为什么发送 `file`（文件）正常，但改成 `record`（语音）就不发了？**
  - **A**: 不同平台或消息适配器（如某些 QQ 客户端、特定协议端）对 `record` 的支持度不同。如果发送失败，说明底层协议端不支持该语音格式，建议退回使用 `file` 模式。

- **Q: 发现生成的音频质量很差、乱读，如何排查？**
  - **A**: 请在群里发一条触发回复的消息后，查看控制台输出。
    1. **看翻译结果**：如果混入了中文解释、Markdown 符号，说明是当前大模型未遵循指令，污染了发给 TTS 的文本。
    2. **看发音效果**：如果 `/wusound_preview` 打印的纯净日语没有问题，但生成的音频依旧差，则是悟声 API 或当前选择的 `voice_id` 本身表现不佳。

- **Q: 悟声已经扣除积分，但群里没收到音频？**
  - **A**: 可能是网络超时或发送失败。请在 WebUI 中开启 `mock_mode`（该模式下只会生成 1 秒的本地蜂鸣声并发送）。如果开启 Mock 后依然收不到，说明是 AstrBot 到聊天平台的发送链路存在问题，与悟声 API 无关。

- **Q: 为什么有些回复不会触发语音？**
  - **A**: 插件具备自动拦截机制。以下情况不会触发：
    1. 含有代码块、HTTP 链接或以 `/` 开头的指令回复。
    2. 回复文本长度超过了 `max_output_tokens` 阈值（默认 80），以防长文刷屏。

---

## 致谢

- [AstrBot](https://github.com/Soulter/AstrBot) - 优秀的 LLM 机器人框架
- 悟声 AI - 提供高质量的 TTS 接口支持

---

## 许可证

本项目采用 [GNU Affero General Public License v3.0 (AGPL-3.0)](LICENSE) 许可协议。