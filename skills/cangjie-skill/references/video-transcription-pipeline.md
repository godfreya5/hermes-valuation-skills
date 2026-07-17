# B站视频音频转写流水线 (whisper.cpp)

> 当 B站视频无字幕/CC，且 `yt-dlp --write-auto-subs` 无法提取时使用。

## 工具链

| 环节 | 工具 | 安装 | 耗时 |
|------|------|------|------|
| 音频下载 | yt-dlp | `apt-get install -y yt-dlp` | ~30s (10MB音频) |
| 音频提取 | ffmpeg | 系统预装 | ~5s |
| 语音转文字 | whisper.cpp + tiny模型 | git clone + make + download | ~3min安装 + ~70s转写(21min音频) |

## 完整命令序列

### 1. 获取视频元信息
```bash
BV="BV1FPTT6uEAX"
curl -sL "https://api.bilibili.com/x/web-interface/view?bvid=${BV}" \
  -H "User-Agent: Mozilla/5.0" -H "Referer: https://www.bilibili.com" \
  | python3 -c "import json,sys;d=json.load(sys.stdin)['data'];print(f'Title:{d[\"title\"]}\nCID:{d[\"cid\"]}\nDuration:{d[\"duration\"]}s')"
```

### 2. 下载音频 (yt-dlp)
```bash
# 关键: 必须同时传 UA + Referer + Origin，否则 HTTP 412
yt-dlp -f "worstaudio" --extract-audio --audio-format mp3 \
  --user-agent "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  --add-header "Referer:https://www.bilibili.com" \
  --add-header "Origin:https://www.bilibili.com" \
  -o "/tmp/video_audio.%(ext)s" \
  "https://www.bilibili.com/video/${BV}"
```

### 3. 提取 WAV (16kHz mono)
```bash
ffmpeg -i /tmp/video_audio.mp3 -vn -acodec pcm_s16le -ar 16000 -ac 1 \
  /tmp/audio_16k.wav -y
```

### 4. 安装 whisper.cpp
```bash
cd /tmp
git clone --depth 1 https://github.com/ggerganov/whisper.cpp.git
cd whisper.cpp
make -j4
# 下载 tiny 模型 (75MB, 中文效果可接受, 速度最快)
./models/download-ggml-model.sh tiny
```

### 5. 转写
```bash
cd /tmp/whisper.cpp
./build/bin/whisper-cli \
  -m models/ggml-tiny.bin \
  -f /tmp/audio_16k.wav \
  -l zh \           # 中文
  -osrt \           # 输出 SRT 字幕格式
  -of /tmp/transcript
# 产出: /tmp/transcript.srt
```

## 模型选择

| 模型 | 大小 | 中文准确率 | 21分钟转写耗时 |
|------|------|-----------|--------------|
| tiny | 75MB | ~70% (口语化内容偏低) | ~70s |
| base | 142MB | ~80% | ~3min |
| small | 466MB | ~90% | ~10min |
| medium | 1.5GB | ~95% | ~30min |

**建议**: 首选用 tiny 快速出结果 → cangjie 分析 → 如质量不足以支撑蒸馏再换 base/small。

## 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| HTTP 412 Precondition Failed | B站防盗链，缺 Referer/Origin | 传 --add-header Referer + Origin |
| "no subtitles for requested languages" | 视频无字幕/CC | 走本流水线 (音频→STT) |
| whisper.cpp make 失败 | 缺依赖 | `apt-get install -y build-essential cmake` |
| 转写质量差 (大量错字) | tiny 模型 + 口语 + 专业术语 | 换 base/small 模型 |
| git clone 慢 | GitHub 网络问题 | 用镜像或用 `--depth 1` |

## 替代方案 (whisper.cpp 不可用时)

1. **Python whisper/faster-whisper**: 需要 PyTorch (~2GB), 容器安装经常超时
2. **云API**: 讯飞听见/阿里云STT/OpenAI Whisper API — 需 API Key
3. **用户提供**: 建议用户用剪映/必剪等工具导出字幕再粘贴
