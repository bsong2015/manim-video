# Manim 数学动画项目

使用 Manim Community Edition 制作数学教学动画视频，支持中文语音旁白和字幕。

## 项目简介

本项目是一个基于 Manim 的数学动画视频制作工具，支持：
- 数学概念可视化演示
- 中文语音旁白（Kokoro TTS）
- 字幕时间自动对齐
- 多种数学动画场景模板

## 环境设置

### 1. 系统要求

- **Python >= 3.11**（Manim 0.22+ 要求）
- **FFmpeg**（视频渲染）
- **Cairo**（图形渲染）

### 2. 安装依赖

```bash
# 克隆项目
git clone git@github.com:bsong2015/manim-video.git
cd manim-video

# 安装所有依赖
uv sync
```

### 3. Hugging Face 镜像配置（可选）

如果无法直接访问 Hugging Face，可在 `.env` 文件中配置镜像：

```
HF_ENDPOINT=https://hf-mirror.com
```

## Skills

本项目包含以下 Claude Code Skills：

| Skill | 说明 |
|-------|------|
| `manimce-best-practices` | Manim Community Edition 最佳实践指南 |
| `manim-skill` | 带语音旁白和字幕的动画生成工作流 |

### Skills 目录结构

```
.agents/skills/           # Skills 实际文件
├── manim-skill/
│   ├── SKILL.md
│   └── tools/
│       ├── Kokoro-82M/       # TTS 模型（本地）
│       ├── tts-generate.py   # TTS 生成脚本
│       ├── lint-subtitles.py # 字幕时间检查
│       └── video_viewer.py   # 视频预览
└── manimce-best-practices/
    ├── SKILL.md
    ├── rules/              # 最佳实践规则
    ├── examples/           # 示例代码
    └── templates/          # 场景模板

.claude/skills/            # Skills 符号链接
```

## 使用方法

### 使用 manim-skill 创建动画

完整工作流程：**Narration → TTS → Code → Render → Mux**

#### Step 1: 创建会话目录

```bash
# 在项目根目录下创建会话目录
mkdir -p .manim-skill/<project-name>
cd .manim-skill/<project-name>
```

#### Step 2: 编写旁白文件

创建 `narration.txt`：

```
subtitles:
- 第一句旁白内容。
- 第二句旁白内容。
- 每一行将生成一个 TTS 音频片段。
```

#### Step 3: 生成 TTS 音频

```bash
# 设置 tools 目录路径
TOOLS_DIR=".agents/skills/manim-skill/tools"

# 中文女声
uv run python "$TOOLS_DIR/tts-generate.py" --narration narration.txt --voice zf_xiaoxiao

# 中文男声
uv run python "$TOOLS_DIR/tts-generate.py" --narration narration.txt --voice zm_yunxi
```

生成文件：
- `voiceover.mp3` - 合并的音频文件
- `timestamps.json` - 每句字幕的时间信息

#### Step 4: 编写动画代码

创建 `script.py`，参考 `timestamps.json` 中的 `duration_s` 设置每个字幕的时间：

```python
from manim import *

# 渲染配置
config.pixel_width = 854
config.pixel_height = 480
config.frame_width = 16
config.frame_height = 9
config.frame_rate = 15

class Scene1_Introduction(Scene):
    def construct(self):
        # 从 timestamps.json 获取时间
        dur = 4.425  # duration_s

        self.add_subcaption("第一句旁白内容。", duration=dur)
        self.play(Write(Text("标题")), run_time=2)
        self.wait(max(0, dur - 2))
```

#### Step 5: 检查字幕时间

```bash
uv run python "$TOOLS_DIR/lint-subtitles.py" script.py
```

确保字幕时间与动画时间对齐。

#### Step 6: 渲染视频

```bash
uv run manim script.py Scene1 Scene2 Scene3 ...
```

#### Step 7: 合并视频和音频

```bash
# 创建场景列表 concat.txt（Windows 用户可用记事本创建）
# Linux/Mac:
cat > concat.txt << EOF
file 'media/videos/script/480p15/Scene1.mp4'
file 'media/videos/script/480p15/Scene2.mp4'
EOF

# Windows PowerShell:
# Set-Content -Path concat.txt -Value "file 'media/videos/script/480p15/Scene1.mp4'", "file 'media/videos/script/480p15/Scene2.mp4'"

# 合并场景
ffmpeg -y -f concat -safe 0 -i concat.txt -c copy video_silent.mp4

# 添加音频
ffmpeg -y -i video_silent.mp4 -i voiceover.mp3 -map 0:v:0 -map 1:a:0 -c:v copy -c:a aac -movflags +faststart video.mp4
```

### 中文语音选项

| 语音名称 | 类型 | 说明 |
|----------|------|------|
| `zf_xiaoxiao` | 中文女声 | 推荐，清晰自然 |
| `zf_xiaoni` | 中文女声 | - |
| `zf_xiaobei` | 中文女声 | - |
| `zf_xiaoyi` | 中文女声 | - |
| `zm_yunxi` | 中文男声 | 推荐 |
| `zm_yunjian` | 中文男声 | - |
| `zm_yunxia` | 中文男声 | - |
| `zm_yunyang` | 中文男声 | - |

### 参考最佳实践

编写动画代码时，参考 `manimce-best-practices` skill：

- `rules/text.md` - 文本渲染
- `rules/latex.md` - 数学公式
- `rules/animations.md` - 动画类型
- `rules/timing.md` - 时间控制
- `examples/math_visualization.py` - 数学可视化示例

## 项目结构

```
manim-video/
├── .agents/
│   └── skills/              # Skills 实际文件
│       ├── manim-skill/
│       │   └── tools/
│       │       ├── Kokoro-82M/   # TTS 模型
│       │       ├── tts-generate.py
│       │       └── lint-subtitles.py
│       └── manimce-best-practices/
├── .claude/
│   └── skills/              # Skills 符号链接
├── .env                     # HF 镜像配置（可选）
├── .gitignore
├── .python-version
├── .venv/                   # 虚拟环境
├── pyproject.toml           # 项目配置
├── uv.lock
└── README.md
```

## 常见问题

### 1. 项目名称

项目名称 `manim-video`，在 `pyproject.toml` 中定义为 `manim-math-videos`（避免与 `manim` 包名冲突）。

### 2. Python 版本要求

Manim 0.22+ 需要 Python >= 3.11，不是 3.10。

### 3. 中文 TTS 依赖

中文语音需要额外依赖，已在 `pyproject.toml` 中配置：
- `ordered-set`
- `pypinyin`
- `cn2an`
- `jieba`

### 4. 字幕时间对齐

使用 `lint-subtitles.py` 检查时间：
- 确保每个 `add_subcaption()` 的 `duration` 正确
- 每个场景的总动画时间不超过字幕时间
- 使用 `self.wait(max(0, dur - used))` 填充剩余时间

## 相关资源

- [Manim Community Edition](https://docs.manim.community/)
- [Kokoro TTS](https://github.com/hexgrad/kokoro)
- [3Blue1Brown](https://www.3blue1brown.com/)
- [manim-skill](https://github.com/Yusuke710/manim-skill)

## License

MIT