# Manim 数学动画项目

使用 Manim Community Edition 制作数学教学动画视频，支持中文语音旁白和字幕。

## 项目简介

本项目用于制作面向小学四年级奥数的数学动画讲解视频，重点覆盖年龄差问题等经典题型。

## 环境设置

### 1. 安装依赖

```bash
# 创建虚拟环境并安装依赖
uv sync

# 安装 Manim 及相关依赖
uv pip install manim kokoro soundfile
```

### 2. 系统要求

- Python >= 3.10
- FFmpeg（视频渲染）
- Cairo（图形渲染）

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

## 使用方法

### 使用 manim-skill 创建动画

1. **创建旁白文件** `narration.txt`：
   ```
   subtitles:
   - 第一句旁白内容。
   - 第二句旁白内容。
   ```

2. **生成 TTS 音频**：
   ```bash
   cd .manim-skill/<project-name>
   uv run python "$TOOLS_DIR/tts-generate.py" --narration narration.txt --voice zf_xiaoxiao
   ```

3. **编写动画代码** `script.py`

4. **渲染视频**：
   ```bash
   manim script.py SceneName
   ```

### 中文语音选项

| 语音名称 | 类型 |
|----------|------|
| `zf_xiaoxiao` | 中文女声（推荐） |
| `zf_xiaoni` | 中文女声 |
| `zm_yunxi` | 中文男声 |

## 目录结构

```
D:\Python\manim\
├── .agents/
│   └── skills/              # Skills 实际文件
│       ├── manim-skill/
│       │   └── tools/
│       │       ├── Kokoro-82M/  # TTS 模型
│       │       └── tts-generate.py
│       └── manimce-best-practices/
├── .claude/
│   └── skills/              # Skills 符号链接
├── .env                     # 环境变量配置
├── .venv/                   # 虚拟环境
├── pyproject.toml
└── README.md
```

## 相关资源

- [Manim Community Edition](https://docs.manim.community/)
- [Kokoro TTS](https://github.com/hexgrad/kokoro)
- [3Blue1Brown](https://www.3blue1brown.com/)

## License

MIT