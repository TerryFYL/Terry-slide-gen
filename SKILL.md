---
name: slide-gen
version: 2.1.0
description: 幻灯片/PPT生成器。前置一次性素材收集，之后全程自主执行，不打断用户。
triggers:
  - "生成PPT / 做幻灯片 / slide gen / 制作幻灯片"
  - "按这个风格生成 / 分析PPT风格"
  - "create slides / make a presentation / slide deck"
neighbors:
  - "academic-figure-engine（提供图表素材）"
---

> **用户请求**: $ARGUMENTS

# slide-gen — 幻灯片生成器 v2.1

## 执行原则

**前置收集，自主执行。** 在 Intake 阶段一次性收齐所有素材，之后不再打断用户，直到交付 PPTX。只有两种情况才中断：① 文件路径不存在等硬性阻塞；② Intake 遗漏了无法合理推断的关键信息。

---

## Phase 0：Intake（一次性素材收集）

**判断触发消息是否已包含足够信息**：
- 内容 ✅ + 风格 ✅ + 图表说明 ✅（或明确无图表）→ 直接跳到 Phase 1
- 缺任何关键项 → 一次性问清，不分多轮

**Intake 要收集的信息**：

| 项目 | 说明 | 缺失时的处理 |
|------|------|-------------|
| **内容** | 主题/大纲/文档/粘贴文字 | 必须有，无法推断，唯一必问项 |
| **风格** | 参考图片/PPT 或风格描述（如"学术风""简洁商务"） | 缺失 → 根据内容自动推断预设 |
| **真实图表** | 需要嵌入的 PNG/PDF 路径，或"无" | 缺失 → 默认无，全部 AI 生成 |
| **受众** | 目标读者类型 | 缺失 → 根据内容推断 |
| **语言** | 中文/英文/其他 | 缺失 → 跟随内容语言 |
| **页数** | 预期总页数 | 缺失 → 按内容长度自动计算 |

**Intake 提问格式**（一次性，简洁）：

```
我需要以下信息来生成幻灯片：

1. **内容**：请提供主题大纲或粘贴源文档
2. **风格**（可选）：有参考模板/图片吗？或直接说风格偏好，留空我来推断
3. **真实图表**（可选）：有需要嵌入的图表文件吗？提供路径，没有则跳过
```

收到回复后 → 进入 Phase 1，不再询问。

---

## Phase 1：分析与规划（自动）

### 1.1 风格确定

**优先级**：
1. 用户提供了参考图片/PPT → 执行**风格提取**（见附录A），得到 `slide_style.json`
2. 用户说了风格偏好关键词 → 匹配预设（见风格映射表）
3. 都没有 → 按内容信号自动推断预设

**风格映射（关键词 → 预设）**：

| 用户说的 | 预设 |
|---------|------|
| 学术/研究/神经/医学/科学 | `scientific` |
| 技术/架构/系统/数据分析 | `blueprint` |
| 教学/课堂/教育 | `chalkboard` |
| 商务/商业/投资/汇报 | `corporate` |
| 简洁/极简/高管 | `minimal` |
| 科技/双语/技术文档 | `intuition-machine` |
| 默认 | `blueprint` |

### 1.2 页数计算

| 内容长度 | 推荐页数 |
|---------|---------|
| < 1000字 | 6–10页 |
| 1000–3000字 | 10–18页 |
| 3000–5000字 | 15–25页 |
| > 5000字 | 20–30页 |

### 1.3 检查图表文件

用户提供了图表路径时：
- 文件存在 → 标记为 `figure` 类型页面
- 文件不存在 → **中断**，告知用户哪个路径找不到，请确认

### 1.4 输出 topic-slug

从内容提取主题，生成 slug（2–4词，kebab-case）。
检查 `slide-deck/{slug}/` 是否已存在：
- 不存在 → 继续
- 已存在 → **自动备份**为 `{slug}-backup-{timestamp}`，继续

---

## Phase 2：生成大纲（自动）

生成 YAML 格式大纲，保存为 `slide-deck/{slug}/outline.yaml`。

**大纲格式**：

```yaml
output_name: "topic-slug"
style: scientific
lang: zh

slides:
  - id: 1
    type: cover
    visual: "神经突触网络纹理背景 + 研究标题"
    labels:
      - "认知负荷与AI辅助决策"
      - "谭晓玲 · 神经认知实验室"
    speaker_notes: "各位好，今天汇报..."

  - id: 2
    type: concept
    visual: "大脑分区图，高亮前额叶皮层，箭头指向任务执行图标"
    labels:
      - "工作记忆容量"
      - "4±1 chunks"
    speaker_notes: "工作记忆是认知负荷理论的核心..."

  - id: 3
    type: figure
    visual: "白板风格画框，顶部标题，底部说明，中间完全留白"
    labels:
      - "核心发现：组间差异显著"
      - "F(2,87)=14.3, p<.001"
    figures:
      - path: "outputs/fig1_boxplot.png"
        position: left
      - path: "outputs/fig2_forest.png"
        position: right
    speaker_notes: "左图是箱线图..."
```

**幻灯片类型**：

| 类型 | 适用 | 核心策略 |
|------|------|---------|
| `cover` | 封面/致谢 | 装饰背景 + 大标题（≤15字） |
| `concept` | 概念可视化 | 视觉隐喻 + 关键词标签（≤20字） |
| `flow` | 流程/框架 | 图标节点 + 箭头连接（≤20字） |
| `figure` | 真实图表页 | AI画框 + PIL合成图表（labels≤15字） |
| `summary` | 总结/结论 | 要点列表 + 视觉收尾（≤20字） |

**文字预算（每张）**：≤20字为安全线；>35字强制拆分为两张。

---

## Phase 3：生成 Prompts（自动）

每张幻灯片生成一个 prompt 文件，保存到 `slide-deck/{slug}/prompts/NN-slide-{name}.md`。

**Prompt 构建**：

```python
ASPECT = (
    "CRITICAL: Generate this image in 16:9 widescreen aspect ratio "
    "(landscape, width ≈ 1.78× height). Do NOT generate square image.\n\n"
)

STYLE_PREFIXES = {
    "scientific": (
        "A 16:9 widescreen presentation slide with clean scientific aesthetic. "
        "White or very light gray background (#F8F9FA). Deep navy text (#1A2B4A). "
        "Accent: teal (#00A8A8) and amber (#FF8C00). "
        "Technical diagram style, precise geometric shapes, dense but clear hierarchy."
    ),
    "intuition-machine": (
        "A 16:9 slide in Intuition Machine style: "
        "dark navy background (#0D1B2A), electric blue accents (#00D4FF), "
        "clean geometric shapes, technical grid overlay, academic bilingual style."
    ),
    "blueprint": (
        "A 16:9 blueprint-style slide: subtle grid background (#E8EEF5), "
        "navy blue (#1A3A5C) text, technical line drawings, engineering aesthetic."
    ),
    "corporate": (
        "A 16:9 professional corporate slide: clean white background, "
        "navy/gold scheme, geometric shapes, professional sans-serif typography."
    ),
    "minimal": (
        "A 16:9 minimal slide: pure white background, single accent color, "
        "generous whitespace, one key visual, executive style."
    ),
    "chalkboard": (
        "A 16:9 chalkboard slide: dark green textured background, "
        "white chalk-style text and drawings, warm educational feel."
    ),
    "editorial-infographic": (
        "A 16:9 editorial infographic slide: light background, bold color blocks, "
        "magazine-style layout, data visualization focus."
    ),
    "bold-editorial": (
        "A 16:9 bold editorial slide: vibrant color blocks, "
        "magazine/keynote aesthetic, dramatic typography contrast."
    ),
    "notion": (
        "A 16:9 Notion-style slide: clean white background, "
        "subtle borders, product dashboard feel, sans-serif typography."
    ),
    "dark-atmospheric": (
        "A 16:9 dark atmospheric slide: deep dark background, "
        "glowing accents, cinematic mood, entertainment style."
    ),
    "sketch-notes": (
        "A 16:9 sketch-notes slide: warm paper-toned background, "
        "hand-drawn marker illustrations, organic layout, educational."
    ),
    "watercolor": (
        "A 16:9 watercolor slide: soft watercolor wash background, "
        "gentle colors, humanist feel, lifestyle/wellness."
    ),
}

def build_prompt(style: str, slide: dict, custom_prefix: str = None) -> str:
    prefix = custom_prefix or STYLE_PREFIXES.get(style, STYLE_PREFIXES["blueprint"])
    parts = [ASPECT, prefix, f"\nVisual scene: {slide['visual']}"]

    if slide.get('labels'):
        parts.append("\nText labels (MUST appear, exact characters):")
        for lbl in slide['labels']:
            parts.append(f'  - "{lbl}"')

    if slide.get('type') == 'figure':
        parts.append(
            "\nIMPORTANT: Center 60% of slide COMPLETELY EMPTY "
            "(plain background only). Title at top, captions at bottom only."
        )

    parts.append(
        "\nAll text: Simplified Chinese (except English terms). "
        "Write ONLY the exact texts above. Do NOT add any other text."
    )
    return "\n".join(parts)
```

---

## Phase 4：生成图片（自动）

### 4.1 API 配置

```python
from pathlib import Path
from google import genai
from google.genai import types

def load_api_key() -> str:
    env = Path("~/.openclaw/credentials/.env").expanduser()
    for line in env.read_text().splitlines():
        if line.startswith("GOOGLE_GENAI_KEY="):
            return line.split("=", 1)[1].strip().strip('"')
    raise RuntimeError("GOOGLE_GENAI_KEY not found")

# 备用通道（官方 Key 失效时）
FALLBACK = {
    "env_key": "AICODECAT_GEMINI_API_KEY",
    "base_url": "https://aicode.cat",
    "model": "gemini-3.1-flash-image-preview",
}
```

### 4.2 生成单张

```python
import os

def generate_slide(prompt: str, output_path: str) -> bool:
    """生成图片，失败自动重试一次，两次都失败才报告"""
    client = genai.Client(api_key=load_api_key())
    for attempt in range(2):
        try:
            resp = client.models.generate_content(
                model="gemini-3.1-flash-image-preview",
                contents=prompt,
                config=types.GenerateContentConfig(
                    response_modalities=["IMAGE", "TEXT"],
                    temperature=0.7,
                ),
            )
            for cand in resp.candidates or []:
                for part in cand.content.parts or []:
                    inline = getattr(part, "inline_data", None)
                    if inline and getattr(inline, "data", None):
                        with open(output_path, "wb") as f:
                            f.write(inline.data)
                        ensure_16_9(output_path)
                        return True
        except Exception:
            if attempt == 1:
                return False
    return False
```

### 4.3 16:9 校正

```python
from PIL import Image

def ensure_16_9(path: str, bg=(248, 249, 250)):
    img = Image.open(path).convert("RGB")
    w, h = img.size
    if abs(w/h - 16/9) < 0.05:
        return
    if abs(w/h - 1.0) < 0.1:
        target_h = int(w * 9 / 16)
        img = img.crop((0, (h - target_h)//2, w, (h + target_h)//2))
    else:
        new_w = int(h * 16 / 9)
        canvas = Image.new("RGB", (max(new_w, w), h), bg)
        canvas.paste(img, ((max(new_w, w) - w)//2, 0))
        img = canvas
    img.save(path, quality=95)
```

### 4.4 Figure 合成（figure 类型页面）

```python
from PIL import Image, ImageDraw

def composite_figures(frame_path: str, figures: list, output_path: str):
    """将真实图表合成到 AI 生成的画框中（中间留白区域）"""
    frame = Image.open(frame_path).convert("RGB")
    W, H = frame.size
    TITLE_BOTTOM, TEXT_TOP = int(H * 0.10), int(H * 0.76)
    FIG_Y, FIG_H = TITLE_BOTTOM + 5, TEXT_TOP - TITLE_BOTTOM - 10

    draw = ImageDraw.Draw(frame)
    draw.rectangle([0, TITLE_BOTTOM, W, TEXT_TOP], fill=frame.getpixel((10, 10)))

    def fit(img, mw, mh):
        r = min(mw/img.width, mh/img.height)
        return img.resize((int(img.width*r), int(img.height*r)), Image.LANCZOS)

    if len(figures) == 1:
        fig = fit(Image.open(figures[0]["path"]).convert("RGB"), W-80, FIG_H)
        frame.paste(fig, ((W-fig.width)//2, FIG_Y+(FIG_H-fig.height)//2))
    elif len(figures) == 2:
        hw = (W - 90) // 2
        for i, f in enumerate(figures):
            fig = fit(Image.open(f["path"]).convert("RGB"), hw, FIG_H)
            x = 30 + i*(hw+30) + (hw-fig.width)//2
            frame.paste(fig, (x, FIG_Y+(FIG_H-fig.height)//2))

    frame.save(output_path, quality=95)
```

### 4.5 进度报告

逐张生成，实时输出（不等用户）：
```
生成中... [3/12] 核心发现：组间差异（figure合成）✅
生成中... [4/12] 实验设计框架 ✅
```

---

## Phase 5：组装 PPTX（自动）

```python
from pptx import Presentation
from pptx.util import Inches

def assemble_pptx(image_paths: list, output_path: str):
    prs = Presentation()
    prs.slide_width = Inches(13.333)
    prs.slide_height = Inches(7.5)
    blank = prs.slide_layouts[6]
    for p in image_paths:
        if p and os.path.exists(p):
            s = prs.slides.add_slide(blank)
            s.shapes.add_picture(p, 0, 0, prs.slide_width, prs.slide_height)
    prs.save(output_path)
```

---

## Phase 6：交付

```
✅ 完成！

主题：[topic]        风格：[preset]
位置：slide-deck/{slug}/
共 N 张：01-cover / 02-... / ... / NN-back-cover

PPTX：slide-deck/{slug}/{slug}.pptx
大纲：slide-deck/{slug}/outline.yaml

如需调整某张，告诉我页码，我重新生成。
```

---

## 局部工作流

```bash
/slide-gen content.md                        # 完整流程
/slide-gen content.md --style scientific     # 跳过风格推断，直接用指定预设
/slide-gen content.md --slides 12            # 指定页数，跳过自动计算
/slide-gen content.md --outline-only         # 只生成大纲，不生图
/slide-gen slide-deck/topic/ --images-only   # 从现有 prompts/ 重新生图
/slide-gen slide-deck/topic/ --regenerate 3  # 只重新生成第3张
```

---

## 附录 A：风格提取（用户提供参考时）

### 从 .pptx 提取样例帧

```python
import subprocess, fitz, os

def pptx_to_frames(pptx_path: str, out_dir="/tmp/pptx_frames", n=5) -> list:
    os.makedirs(out_dir, exist_ok=True)
    subprocess.run(
        ["libreoffice", "--headless", "--convert-to", "pdf", "--outdir", out_dir, pptx_path],
        check=True
    )
    pdf = os.path.join(out_dir, os.path.basename(pptx_path).replace(".pptx", ".pdf"))
    doc = fitz.open(pdf)
    paths = []
    for i, page in enumerate(doc[:n]):
        p = os.path.join(out_dir, f"frame_{i+1:03d}.png")
        page.get_pixmap(matrix=fitz.Matrix(2, 2)).save(p)
        paths.append(p)
    return paths
```

### 风格分析 Prompt

```
分析这张演示幻灯片的视觉风格，输出 JSON：
{
  "background": {"type": "solid|gradient|dark|paper|whiteboard", "description": "英文"},
  "typography": {"style": "clean_sans|serif|handwritten|technical", "description": "英文"},
  "color_palette": {"primary_text": "#hex", "accent_1": "#hex", "background_base": "#hex"},
  "atmosphere": "academic|corporate|educational|dark_tech|playful",
  "prompt_prefix": "A 16:9 widescreen presentation slide. [100词内英文，描述视觉风格和氛围，不描述文字排版]"
}
只输出 JSON。
```

分析结果保存为 `slide-deck/{slug}/slide_style.json`，后续所有页面使用其 `prompt_prefix` 替代预设前缀。

---

## 附录 B：输出目录结构

```
slide-deck/{topic-slug}/
├── source.md
├── outline.yaml
├── slide_style.json       （风格提取时存在）
├── prompts/
│   ├── 01-slide-cover.md
│   └── 02-slide-{name}.md
├── 01-slide-cover.png
├── 02-slide-{name}.png
└── {topic-slug}.pptx
```
