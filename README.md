# Voice Simultaneous Interpretation

A real-time voice simultaneous interpretation system (Chinese -> English) based on DeepSeek API + CosyVoice local TTS. Uses a three-thread pipeline architecture: recording, translation+synthesis, and playback run concurrently without blocking each other — enabling speak-while-translate-while-play.

## Features

- Real-time microphone capture, Google Speech Recognition for Chinese speech recognition
- DeepSeek API Chinese-to-English translation
- Local CosyVoice zero-shot voice cloning for English audio synthesis
- Three-thread pipeline architecture: Ear (recording) -> Brain (recognition + translation + synthesis) -> Mouth (playback), non-blocking
- Reference audio preprocessing tools: format cleaning (convert to 16kHz mono PCM_16), duration trimming (keep first N seconds)
- Reference audio playback test tool for verifying audio integrity

## Environment Requirements

- Windows 10+
- Python 3.x
- Locally deployed CosyVoice2-0.5B service (listening on 127.0.0.1:50000)
- DeepSeek API Key
- Microphone device

## Dependencies

```
pip install SpeechRecognition openai pygame requests soundfile librosa numpy
```

## Directory Structure

```
.
├── 主代码同声传译 接入deepseek api +CosyVoice本地运行.txt   # Main program
├── 音频转换成CosyVoice能识别出的音频.txt                    # Reference audio format cleaning
├── 切割音频代码.txt                                         # Reference audio duration trimming
├── 音频测试.txt                                              # Reference audio playback test
├── 命名为 启动50000端口.bat.txt                              # CosyVoice startup script
├── .env.example                                              # Environment variable template
├── my_voice.wav                                              # Reference audio (prepare your own)
└── README.md
```

## Installation

```bash
git clone https://github.com/doaneruby970-hub/voice-translator.git
cd voice-translator
pip install SpeechRecognition openai pygame requests soundfile librosa numpy
cp .env.example .env
```

## Configuration

### 1. Environment Variables

Copy `.env.example` to `.env` and fill in your DeepSeek API Key:

```
DEEPSEEK_API_KEY=sk-your-key-here
```

### 2. Start CosyVoice Local Service

Refer to `命名为 启动50000端口.bat.txt` to start the CosyVoice service on `http://127.0.0.1:50000`.

Core script command (modify paths according to your actual setup):

```bash
python runtime/python/fastapi/server.py --port 50000 --model_dir pretrained_models/CosyVoice2-0.5B
```

### 3. Prepare Reference Audio

Place a reference audio file named `my_voice.wav` in the project root directory.

Reference audio requirements:
- Format: WAV, 16kHz sample rate, mono, PCM_16
- Duration: within 10 seconds
- Purpose: Reference timbre for CosyVoice zero-shot voice cloning

If the audio does not meet the requirements, use the bundled tools to process it:

**Format Cleaning** (any format -> CosyVoice standard format):
```bash
python 音频转换成CosyVoice能识别出的音频.txt
```

**Duration Trimming** (keep first 10 seconds):
```bash
python 切割音频代码.txt
```

**Playback Test** (verify audio is working):
```bash
python 音频测试.txt
```

### 4. Modify Reference Audio Transcript

Edit the `REF_TEXT` variable in the main program file (line 21), changing the text to what is actually spoken in the reference audio:

```python
REF_TEXT = "your reference audio transcript"
```

## Usage

After confirming the CosyVoice service is running on port 50000, execute the main program:

```bash
python 主代码同声传译 接入deepseek api +CosyVoice本地运行.txt
```

Speak Chinese into the microphone, and the program will automatically:
1. Recognize what you said
2. Call DeepSeek to translate into English
3. Use CosyVoice to clone the reference audio timbre and synthesize English speech
4. Play the translation result through the speakers

Press `Ctrl+C` to stop.

## Notes

- The main program file is in `.txt` format instead of `.py`. It is recommended to rename it to `.py` before running.
- CosyVoice service must be started first, or audio synthesis will fail.
- The reference audio `my_voice.wav` must be in the project directory, and its duration must be kept within 10 seconds.
- The program uses Google Speech Recognition for Chinese recognition, which requires an internet connection.
- Generated translation audio files are automatically deleted after playback.
- Silence exceeding 2 seconds is treated as end-of-sentence, triggering translation.
- Only supports Chinese input -> English output. To modify the target language, edit the translation prompt.
