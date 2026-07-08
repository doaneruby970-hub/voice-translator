# 语音同声传译

基于 DeepSeek API + CosyVoice 本地 TTS 的实时语音同声传译系统（中文 -> 英文）。采用三线程流水线架构，录音、翻译合成、播放互不阻塞，实现边说边译边播。

## 功能特性

- 麦克风实时拾音，Google Speech Recognition 识别中文语音
- DeepSeek API 中译英翻译
- 本地 CosyVoice 零样本语音克隆合成英文音频
- 三线程流水线架构：耳朵（录音）-> 大脑（识别+翻译+合成）-> 嘴巴（播放），互不阻塞
- 参考音频预处理工具：格式清洗（转 16kHz 单声道 PCM_16）、时长裁剪（保留前 N 秒）
- 参考音频回放测试工具，用于验证音频是否完整

## 环境要求

- Windows 10+
- Python 3.x
- 本地部署的 CosyVoice2-0.5B 服务（监听 127.0.0.1:50000）
- DeepSeek API Key
- 麦克风设备

## 依赖

```
pip install SpeechRecognition openai pygame requests soundfile librosa numpy
```

## 目录结构

```
.
├── 主代码同声传译 接入deepseek api +CosyVoice本地运行.txt   # 主程序
├── 音频转换成CosyVoice能识别出的音频.txt                    # 参考音频格式清洗
├── 切割音频代码.txt                                         # 参考音频时长裁剪
├── 音频测试.txt                                              # 参考音频回放测试
├── 命名为 启动50000端口.bat.txt                              # CosyVoice 启动脚本
├── .env.example                                              # 环境变量模板
├── my_voice.wav                                              # 参考音频（需自行准备）
└── README.md
```

## 安装

```bash
git clone <repo-url>
cd voice-translator
pip install SpeechRecognition openai pygame requests soundfile librosa numpy
cp .env.example .env
```

## 配置

### 1. 环境变量

复制 `.env.example` 为 `.env`，填入 DeepSeek API Key：

```
DEEPSEEK_API_KEY=sk-your-key-here
```

### 2. 启动 CosyVoice 本地服务

参考 `命名为 启动50000端口.bat.txt`，将 CosyVoice 服务启动在 `http://127.0.0.1:50000`。

脚本核心命令（请根据实际路径修改）：

```bash
python runtime/python/fastapi/server.py --port 50000 --model_dir pretrained_models/CosyVoice2-0.5B
```

### 3. 准备参考音频

放置一个名为 `my_voice.wav` 的参考音频文件到项目根目录。

参考音频要求：
- 格式：WAV, 16kHz 采样率, 单声道, PCM_16
- 时长：10 秒以内
- 用途：CosyVoice 零样本语音克隆的参考音色

如果音频不符合要求，使用配套工具处理：

**格式清洗**（任意格式 -> CosyVoice 标准格式）：
```bash
python 音频转换成CosyVoice能识别出的音频.txt
```

**时长裁剪**（保留前 10 秒）：
```bash
python 切割音频代码.txt
```

**回放测试**（验证音频是否正常）：
```bash
python 音频测试.txt
```

### 4. 修改参考音频台词

编辑主程序文件中的 `REF_TEXT` 变量（第 21 行），将该行文本改为参考音频中实际朗读的内容：

```python
REF_TEXT = "你的参考音频台词"
```

## 使用方式

确认 CosyVoice 服务已在 50000 端口运行后，执行主程序：

```bash
python 主代码同声传译 接入deepseek api +CosyVoice本地运行.txt
```

对着麦克风说中文，程序会自动：
1. 识别你说的话
2. 调用 DeepSeek 翻译成英文
3. 用 CosyVoice 克隆参考音频的音色合成英文语音
4. 通过扬声器播放翻译结果

按 `Ctrl+C` 停止。

## 注意事项

- 主程序文件是 `.txt` 格式而非 `.py`，运行前建议重命名为 `.py`
- CosyVoice 服务必须先启动，否则音频合成会失败
- 参考音频 `my_voice.wav` 必须在项目目录下，且时长控制在 10 秒以内
- 程序使用 Google Speech Recognition 进行中文识别，需要网络连接
- 生成的翻译音频文件会在播放后自动删除
- 静默超过 2 秒视为一句话结束，触发翻译
- 仅支持中文输入 -> 英文输出，如要修改目标语言请修改翻译 prompt
