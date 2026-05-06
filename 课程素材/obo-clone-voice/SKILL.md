---
name: obo-clone-voice
description: >
  This skill should be used when the user wants to clone a voice in the 欧博AI系统
  (Oubo AI System). It automates the full voice clone workflow: login, navigate to
  AI语音生成, switch to the 声音克隆 tab, upload a reference audio file, fill in
  reading text, and trigger generation. Trigger phrases include: "克隆声音",
  "帮我克隆一个声音", "用欧博克隆语音", "AI语音生成", "voice clone",
  "声音克隆".
agent_created: true
---

# 欧博AI系统 - 声音克隆技能

自动化完成欧博AI系统的声音克隆流程。

## 执行前询问

必须先询问用户提供以下三项信息：

1. **账号** — 欧博系统登录用户名
2. **密码** — 对应账号的密码
3. **参考音频路径** — 本地音频文件的绝对路径（.wav / .mp3 / .m4a）
4. **朗读文本**（可选，未提供时使用默认值）— `WorkBuddy，您的个人智能助手，值得拥有。`

## 完整执行流程

### 第一步：关闭旧会话并打开浏览器

```bash
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser close --all
```

```bash
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser open "http://sc.wwwl.net:39090/login" --headed
```

### 第二步：登录

```bash
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser wait 2000
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser snapshot -i
```

从快照中找到用户名输入框、密码输入框、登录按钮的 ref（每次会话会变化），然后：

```bash
# 替换 <USERNAME> <PASSWORD> 和 <REF> 为实际值
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser fill @<用户名ref> "<USERNAME>"
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser fill @<密码ref> "<PASSWORD>"
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser click @<登录按钮ref>
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser wait 3000
```

### 第三步：导航到 AI 语音生成

```bash
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser snapshot -i
```

典型菜单路径：
1. 点击左侧 `ComfyUI工作流` 菜单（展开子菜单）
2. 点击展开后的子菜单 `AI工具箱`
3. 在 AI工具箱页面中点击 `AI 语音生成` 功能卡片

```bash
# 展开父菜单
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser click @<ComfyUI工作流ref>
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser wait 1500

# 点击 AI工具箱
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser click @<AI工具箱ref>
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser wait 1500

# 点击 AI 语音生成 卡片
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser snapshot -i
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser click @<AI语音生成卡片ref>
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser wait 2000
```

### 第四步：切换到"声音克隆"标签

```bash
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser snapshot -i
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser click @<声音克隆tab_ref>
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser wait 1500
```

### 第五步：上传参考音频（关键步骤）

> **注意**：Element UI 的 el-upload 组件不是原生 file input，直接用 `@ref` 上传会报错 `Node is not a file input element`。
> 必须使用 CSS 选择器。

```bash
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser upload "input.el-upload__input[type=file]" "<参考音频绝对路径>"
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser wait 3000
```

路径示例（Windows）：`E:/develop/my_obsidian_note/zp-workbuddy-note/课程素材/参考音频.MP3`

### 第六步：填写朗读文本 & 触发生成

```bash
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser snapshot -i
```

找到朗读文本框 ref 和开始生成按钮 ref（上传音频后按钮激活），然后：

```bash
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser fill @<朗读文本ref> "<朗读文本>"
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser network requests --clear
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser click @<开始生成ref>
```

### 第七步：等待生成并反馈结果

```bash
# 声音克隆通常需要 10~20 秒
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser wait 15000
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser screenshot
```

检查生成记录表格，确认克隆状态：

```bash
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && agent-browser snapshot
```

汇报结果给用户，包含：克隆音色 ID、生成状态（成功/失败）、截图路径。

## 关键踩坑

| 问题 | 原因 | 解决方案 |
|---|---|---|
| `agent-browser open` 报 `ERR_NO_SUPPORTED_PROXIES` | 系统代理 127.0.0.1:7897 不可用 | 所有命令前加 `unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy` |
| `agent-browser upload @ref` 报错 `Node is not a file input element` | el-upload 是自定义组件 | 改用 CSS 选择器 `input.el-upload__input[type=file]` |
| "开始生成"按钮 disabled | 声音克隆模式必须先上传参考音频 | 先执行上传文件步骤 |
| ref 编号每次会话变化 | agent-browser 每次快照重新分配 | 每次操作前重新 `snapshot -i` 获取最新 ref |
| 网络请求抓不到生成 API | 旧记录太多被淹没 | 点击生成前先 `network requests --clear` |
