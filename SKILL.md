---
name: slide-gen
version: 4.1.0
description: 幻灯片/PPT生成器。支持从学术PDF文献自动提取图表并生成汇报PPT。前置一次性素材收集，之后全程自主执行，不打断用户。学术文献模式使用故事线驱动的python-pptx原生渲染。v4.1新增图表矢量重绘：从栅格图中用AI提取数据点，matplotlib重绘为真矢量SVG。
triggers:
  - "生成PPT / 做幻灯片 / slide gen / 制作幻灯片"
  - "按这个风格生成 / 分析PPT风格"
  - "create slides / make a presentation / slide deck"
  - "从文献生成PPT / 文献总结PPT / 学术汇报"
neighbors:
  - "academic-figure-engine（提供图表素材）"
---

> **用户请求**: $ARGUMENTS

# slide-gen — 幻灯片生成器 v3.0

## 执行原则

**前置收集，自主执行。** 在 Intake 阶段一次性收齐所有素材，之后不再打断用户，直到交付 PPTX。只有两种情况才中断：① 文件路径不存在等硬性阻塞；② Intake 遗漏了无法合理推断的关键信息。

---

## Phase 0：Intake（一次性素材收集）

**判断触发消息是否已包含足够信息**：
- 内容 ✅ + 风格说明 ✅ → 直接跳到 Phase 0.5（有PDF）或 Phase 1（无PDF）
- 缺任何关键项 → 一次性问清，不分多轮

**Intake 要收集的信息**：

| 项目 | 说明 | 缺失时的处理 |
|------|------|-------------|
| **内容来源** | ① 粘贴文字/大纲，或 ② PDF 文献路径（单文件或文件夹） | 必须有，唯一必问项 |
| **风格** | 参考图片/PPT 或风格描述（如"学术风""临床指南"） | 缺失 → 有PDF默认 `academic-light`，否则按内容推断 |
| **受众** | 目标读者类型 | 缺失 → 根据内容推断 |
| **语言** | 中文/英文/其他 | 缺失 → 跟随内容语言 |
| **页数** | 预期总页数 | 缺失 → 按内容/文献数量自动计算 |

**Intake 提问格式**（一次性，简洁）：

```
我需要以下信息来生成幻灯片：

1. **内容**：请提供 ① 主题大纲/文字，或 ② PDF文献路径（单文件路径 或 文件夹路径均可）
2. **风格**（可选）：学术汇报/临床指南/深色科技？留空我来推断
3. **页数**（可选）：留空自动计算
```

收到回复后 → 进入 Phase 0.5（有PDF）或 Phase 1（无PDF），不再询问。

---

---

## 学术 PPT 核心哲学（v4.0 新增，学术文献模式必读）

### 第一性原理

**所有知识类内容的灵魂是故事线**——无论输出形式是 PPT、论文、博客还是公众号，底层无差别。

PPT 是演讲的视觉辅助，不是文档。人脑有语言通道（听/读文字）和视觉通道（看图像）两个并行通道，同一通道无法并行——所以 PPT 文字越少越好，不是因为美观，是认知机制决定的。

学术听众不是来被说服的，是来**评估证据是否支持结论**的。每张 slide 的核心问题不是"我要放什么内容"，而是：**这个逻辑节点上，最关键的关系是什么？**

### AI 在这个过程中的角色

人类读论文：线性阅读 → 内化理解 → 故事线自然浮现 → 表达

AI 没有内化能力，但有人类没有的优势：**同时读取全文所有叙事信号**。

论文作者已经把故事线编码进去了——摘要是压缩的故事线，Discussion 是作者在说"这意味着什么"，"Importantly…""We found…""This suggests…" 是作者自己标注的叙事高潮。

AI 的任务不是"理解"论文，而是**解码作者已编码的叙事锚点**，将其显式化为故事线文档，再从故事线生长出 slides。

### 执行顺序（学术文献模式）

```
PDF
  ↓ 解码叙事锚点
故事线文档（5个节点）
  ↓ 每个节点回答3个问题
Slide 规格计划
  ↓ python-pptx 原生渲染
PPTX
```

### 3个布局决策问题（每张 slide 必答）

1. **这张 slide 的"一个想法"是什么？**（标题句，直接来自故事线节点）
2. **什么视觉元素最能承载这个想法？**（论文原图 / 生成概念图 / 两者组合）
3. **这些元素之间是什么关系？**（决定布局形态）

### 布局形态库（内容关系 → 布局）

| 内容关系 | 布局形态 |
|---------|---------|
| 单一核心图（空间型，如脑图）| 全版大图 + 顶部一句结论 |
| 数据图 + 次级统计 | 主图居中 + 两侧数据注解卡片 |
| 两组对比 | 左右对称 |
| 流程/方法 | 横向箭头链 / 圆形闭环 |
| 多 panel 图 + 结论 | 图居中 + 侧边效应量卡片 |
| 三个并列发现 | 三栏卡片 |

> **布局由内容逻辑关系推导，不是套模板。**

### 配色系统（白底原则）

学术汇报幻灯片背景**必须用白色**（投影仪清晰度 + 专业感）。

| 区域 | 颜色 | 说明 |
|------|------|------|
| 标题栏 | `#1A2E4A` Navy | 唯一允许深色的区域 |
| 引用栏 | `#008C8C` Teal | 底部细条 |
| 卡片背景 | `#FFFFFF` 白 / `#F0F6F6` 极淡青 | 绝不用深色背景 |
| 卡片顶条/竖条 | Navy 或 Teal | 区分左右语义 |
| 大数字/标题 | Navy | 主视觉 |
| 标签/分类 | Teal | 次视觉 |
| 说明文字 | `#666666` 灰 | 辅助 |
| **禁止** | 橙色等第三色 | 只用 Navy + Teal 双色系 |

### 技术规范

- 所有文字标签：`tf.word_wrap = False`（防止 % 等短字符被挤到第二行）
- 数值标签 textbox 宽度：实际文字宽度 + 至少 0.15" 余量
- 图片处理：用 PyMuPDF 300dpi 提取 → PIL 裁掉英文 caption（保留 panels + 图例，约保留顶部 65-70%）→ 重新计算宽高比后布局
- 侧边留白处理：当主图宽高比无法填满 16:9 时，侧边用**次级证据卡片**填充（效应量、样本量、协议信息等），不留白

---

## Phase 0.5：文献解析（仅当用户提供 PDF 路径时执行）

### 0.5.1 扫描输入

```python
import os
from pathlib import Path

def collect_pdfs(input_path: str) -> list[Path]:
    """单文件 or 文件夹均可"""
    p = Path(input_path).expanduser()
    if p.is_file() and p.suffix.lower() == ".pdf":
        return [p]
    elif p.is_dir():
        return sorted(p.glob("*.pdf"))
    else:
        raise FileNotFoundError(f"路径不存在或非PDF: {input_path}")
```

### 0.5.2 提取文献元数据 + 故事线

这一步是学术 PPT 的核心。**不只提取元数据，要解码作者已编码的叙事锚点。**

作者在论文里留下了故事线信号：摘要（压缩故事线）、引言末段（研究问题）、Discussion 开头（核心主张）、"Importantly/We found/This suggests"（叙事高潮标记）。AI 的任务是找到这些锚点，构建显式故事线文档。

```python
import fitz  # PyMuPDF

def extract_story_line(pdf_path: Path) -> dict:
    """
    从全文提取故事线文档（5个节点）。
    策略：读摘要+引言+Discussion全文，找作者自己写的叙事锚点。
    """
    doc = fitz.open(pdf_path)
    # 提取全文（不只首页）
    full_text = ""
    for page in doc:
        full_text += page.get_text("text")

    prompt = f"""你是学术论文分析专家。请从以下论文全文中，找出作者自己写的叙事锚点（如"Importantly","We found","This suggests","Crucially"等关键句），构建故事线文档。

输出严格JSON格式：
{{
  "title": "论文完整标题",
  "authors": "第一作者 et al.",
  "journal": "期刊名",
  "year": "年份",
  "doi": "DOI（如有）",
  "story_line": {{
    "tension": {{
      "statement": "这个领域存在什么裂缝/未解决问题（≤60字，中文）",
      "source_quote": "论文原文对应句子（英文，≤100字）"
    }},
    "method": {{
      "statement": "用什么方法回答这个问题，为什么合适（≤60字，中文）",
      "source_quote": "论文原文对应句子（英文，≤80字）"
    }},
    "key_evidence": [
      {{
        "statement": "最关键发现，含具体数据（≤60字，中文）",
        "source_quote": "论文原文叙事高潮句（英文）",
        "figure": "对应Figure编号，如Figure 3",
        "effect_size": "效应量或统计值，如η²p=0.68"
      }}
    ],
    "conclusion": {{
      "statement": "作者最终主张（≤60字，中文）",
      "source_quote": "Discussion中的主张原句（英文，≤100字）"
    }},
    "significance": {{
      "statement": "改变了对什么的认知（≤60字，中文）",
      "source_quote": "论文原文对应句子（英文，≤80字）"
    }}
  }},
  "all_figures": [
    {{"number": "Figure 1", "caption": "原文图注（英文，≤80字）"}}
  ]
}}

只输出JSON，不要其他内容。

论文全文：
{full_text[:15000]}"""

    resp = gemini_text(prompt)
    story = json.loads(resp)
    story["file"] = str(pdf_path)
    return story
```

**输出示例（Jiang et al. 2026）**：
```json
{
  "story_line": {
    "tension": {
      "statement": "ADHD干预通常需要高频率课时，低频设计是否同样有效尚未验证",
      "source_quote": "Traditional NCT programs typically require 3-5 sessions weekly..."
    },
    "key_evidence": [{
      "statement": "笔迹构建问题显著改善，全文最大效应量",
      "figure": "Figure 5",
      "effect_size": "η²p = 0.68, p < 0.001"
    }]
  }
}
```

### 0.5.3 提取文献图表

```python
def extract_figures(pdf_path: Path, out_dir: Path, paper_id: str) -> list[dict]:
    """
    提取PDF中所有图表（图片+caption），保存为PNG。
    策略：扫描每页，找到独立图片块（宽>页宽30%，高>页高15%），裁剪保存。
    """
    doc = fitz.open(pdf_path)
    figures = []
    fig_idx = 0

    for page_num, page in enumerate(doc):
        page_dict = page.get_text("dict")
        page_w, page_h = page.rect.width, page.rect.height

        # 提取页面中的图片对象
        img_list = page.get_images(full=True)
        for img_info in img_list:
            xref = img_info[0]
            rects = page.get_image_rects(xref)
            if not rects:
                continue
            rect = rects[0]

            # 过滤太小的图（Logo、图标等）
            if rect.width < page_w * 0.25 or rect.height < page_h * 0.12:
                continue

            # 裁剪图片区域（向下多取80px以包含caption）
            clip = fitz.Rect(rect.x0, rect.y0, rect.x1, min(rect.y1 + 80, page_h))
            mat = fitz.Matrix(2, 2)  # 2x 分辨率
            pix = page.get_pixmap(matrix=mat, clip=clip)

            fig_idx += 1
            fname = f"{paper_id}_fig{fig_idx:02d}_p{page_num+1}.png"
            fpath = out_dir / fname
            pix.save(str(fpath))

            # 用 Gemini vision 生成图说
            caption = gemini_describe_figure(str(fpath))

            figures.append({
                "path": str(fpath),
                "paper_id": paper_id,
                "page": page_num + 1,
                "caption_ai": caption,
            })

    return figures
```

### 0.5.4 生成 figures_manifest.yaml

```yaml
# figures_manifest.yaml — 自动生成，勿手动编辑
papers:
  - paper_id: "p1"
    file: "/path/to/paper1.pdf"
    title: "Cognitive Load and AI Decision Support"
    first_author: "Zhang"
    journal: "Nat. Neurosci."
    year: "2024"
    key_finding: "AI辅助决策在高认知负荷条件下可将错误率降低34%"
    figures:
      - path: "slide-deck/{slug}/figs/p1_fig01_p3.png"
        caption_ai: "实验组与对照组在认知负荷测试中的箱线图对比"
      - path: "slide-deck/{slug}/figs/p1_fig02_p5.png"
        caption_ai: "fMRI激活图显示前额叶皮层差异"

  - paper_id: "p2"
    ...
```

**进度提示（实时输出）**：
```
解析文献中...
  ✅ paper1.pdf → 标题已提取，找到 3 张图
  ✅ paper2.pdf → 标题已提取，找到 2 张图
共 2 篇文献，5 张图表 → 生成大纲中...
```

---

## Phase 1：分析与规划（自动）

### 1.1 风格确定

**优先级**：
1. 用户提供了参考图片/PPT → 执行**风格提取**（见附录A），得到 `slide_style.json`
2. 用户说了风格偏好关键词 → 匹配预设（见风格映射表）
3. 有PDF文献输入 → 默认 `academic-light`
4. 都没有 → 按内容信号自动推断预设

**风格映射（关键词 → 预设）**：

| 用户说的 | 预设 |
|---------|------|
| 学术/研究/神经/医学/科学/汇报 | `academic-light` |
| 脑影像/神经科学/fMRI/深色 | `academic-dark` |
| 指南/临床/培训/教学 | `clinical-guide` |
| 技术/架构/系统/数据分析 | `blueprint` |
| 商务/商业/投资 | `corporate` |
| 简洁/极简/高管 | `minimal` |
| 科技/双语 | `intuition-machine` |
| 默认 | `blueprint` |

### 1.2 页数计算

**有PDF文献时**：
- 封面 1 页 + 每篇文献 2–3 页（核心发现页 + 图表页）+ 总结 1 页
- 例：3篇文献 → 1 + 3×2.5 + 1 ≈ 10页

**纯文字内容时**：
| 内容长度 | 推荐页数 |
|---------|---------|
| < 1000字 | 6–10页 |
| 1000–3000字 | 10–18页 |
| > 3000字 | 15–25页 |

### 1.3 输出 topic-slug

从内容提取主题，生成 slug（2–4词，kebab-case）。
检查 `slide-deck/{slug}/` 是否已存在：
- 不存在 → 继续
- 已存在 → **自动备份**为 `{slug}-backup-{timestamp}`，继续

---

## Phase 2：生成大纲（自动）

生成 YAML 格式大纲，保存为 `slide-deck/{slug}/outline.yaml`。

**有PDF文献时的标准页面结构**（每篇文献）：

```yaml
# 文献发现页：文字精简，结论突出
- id: 3
  type: lit-finding
  paper_id: "p1"
  visual: "白底简洁布局，左侧竖条accent色，右侧核心数字大字显示"
  labels:
    - "Zhang et al., Nat. Neurosci. 2024"
    - "AI辅助决策降低错误率34%"
    - "n=120, p<0.001"
  citation: "Zhang et al. (2024) Nature Neuroscience"
  speaker_notes: "这篇来自Nature Neuroscience的研究..."

# 文献图表页：图大、字少、来源清晰
- id: 4
  type: lit-figure
  paper_id: "p1"
  visual: "白底，顶部细标题栏，图占70%版面，底部文献来源"
  labels:
    - "认知负荷条件下的错误率对比"
  figures:
    - path: "slide-deck/{slug}/figs/p1_fig01_p3.png"
      caption: "实验组与对照组箱线图（p<0.001）"
  citation: "Zhang et al. (2024) Nat. Neurosci."
  speaker_notes: "左图显示..."
```

**幻灯片类型（v3.0新增）**：

| 类型 | 适用 | 核心策略 |
|------|------|---------|
| `cover` | 封面/致谢 | 装饰背景 + 大标题（≤15字） |
| `concept` | 概念可视化 | 视觉隐喻 + 关键词标签（≤20字） |
| `flow` | 流程/框架 | 图标节点 + 箭头连接（≤20字） |
| `figure` | 用户提供的图表 | AI画框 + PIL合成（labels≤15字） |
| `lit-finding` | 文献核心发现 | 精简结论 + 统计数字突出 + 底部来源 |
| `lit-figure` | 文献图表展示 | 图占70%版面 + 标题 + 底部来源 |
| `summary` | 总结/结论 | 要点列表 + 视觉收尾（≤20字） |

**文字预算（每张）**：
- 标题：≤12字
- 结论/正文：≤30字
- 图表区：占60–70%版面
- 文献来源（角落）：≤25字，小字
- 超出自动拆分为两张

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
    "academic-light": (
        "A 16:9 widescreen academic presentation slide. "
        "Clean white background (#FFFFFF). Deep navy text (#1A2B4A). "
        "Left accent bar in teal (#007A8A, 8px wide, full height). "
        "Title area at top (height ~12%), main content in center. "
        "Bottom 10% of slide: plain white, completely empty — no text, no lines, no decoration. "
        "Minimal text, precise and professional."
    ),
    "academic-dark": (
        "A 16:9 widescreen neuroscience presentation slide. "
        "Dark navy background (#0D1B2A). Electric blue accents (#00D4FF). "
        "White text (#F0F4F8). Figure-dominant layout. "
        "Citation text in light gray at bottom-right corner. "
        "High contrast, suitable for brain imaging and fMRI data."
    ),
    "clinical-guide": (
        "A 16:9 widescreen clinical guideline slide. "
        "White background (#FAFAFA). Forest green header bar (#2E7D4F). "
        "Clean sans-serif typography. Multi-column layout for key points. "
        "Structured, hierarchical, minimal decoration. "
        "Citation at bottom in small gray text."
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
}

def build_prompt(style: str, slide: dict, custom_prefix: str = None) -> str:
    prefix = custom_prefix or STYLE_PREFIXES.get(style, STYLE_PREFIXES["blueprint"])
    parts = [ASPECT, prefix, f"\nVisual scene: {slide['visual']}"]

    if slide.get('labels'):
        parts.append("\nText labels (MUST appear, exact characters):")
        for lbl in slide['labels']:
            parts.append(f'  - "{lbl}"')

    # lit-figure / figure 类型：中间留空给真实图表
    if slide.get('type') in ('figure', 'lit-figure'):
        parts.append(
            "\nIMPORTANT: Center 65% of slide height COMPLETELY EMPTY "
            "(pure white, absolutely NO drawings, illustrations, or patterns). "
            "Only the title label at top is allowed."
        )

    # ✅ 全局 citation 禁令：Gemini 不负责任何 citation 渲染
    # citation 统一由 burn_citation() 用 PIL 烧录，此处禁止 Gemini 添加
    parts.append(
        "\nAll text: Simplified Chinese (except author names, journal names, statistics). "
        "Write ONLY the exact text labels listed above."
        "\nSTRICT RULE: Do NOT add any citation, reference, footnote, source line, "
        "acknowledgement, or author credit anywhere on the slide. "
        "Bottom 10% must be plain white with zero text."
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
import json

def load_api_key() -> str:
    env = Path("~/.openclaw/credentials/.env").expanduser()
    for line in env.read_text().splitlines():
        if line.startswith("GOOGLE_GENAI_KEY="):
            return line.split("=", 1)[1].strip().strip('"')
    raise RuntimeError("GOOGLE_GENAI_KEY not found")

def gemini_text(prompt: str) -> str:
    """文字任务用 flash，节省成本"""
    client = genai.Client(api_key=load_api_key())
    resp = client.models.generate_content(
        model="gemini-2.0-flash",
        contents=prompt,
    )
    return resp.text

def gemini_describe_figure(image_path: str) -> str:
    """用 Gemini vision 描述图表内容（中文一句话）"""
    client = genai.Client(api_key=load_api_key())
    with open(image_path, "rb") as f:
        img_bytes = f.read()
    resp = client.models.generate_content(
        model="gemini-2.0-flash",
        contents=[
            types.Part.from_bytes(data=img_bytes, mime_type="image/png"),
            "用一句话描述这张学术图表的核心内容（中文，≤40字）"
        ],
    )
    return resp.text.strip()

# 备用通道（官方 Key 失效时）
FALLBACK = {
    "env_key": "AICODECAT_GEMINI_API_KEY",
    "base_url": "https://aicode.cat",
    "model": "gemini-2.0-flash-preview-image-generation",
}
```

### 4.2 生成单张幻灯片背景图

```python
def generate_slide(prompt: str, output_path: str) -> bool:
    """生成图片，失败自动重试一次"""
    client = genai.Client(api_key=load_api_key())
    for attempt in range(2):
        try:
            resp = client.models.generate_content(
                model="gemini-2.0-flash-preview-image-generation",
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

### 4.4 Citation 烧录（所有含 citation 的幻灯片）

```python
from PIL import Image, ImageDraw, ImageFont

# ✅ 设计原则：citation 完全由 PIL 控制，Gemini prompt 里不出现任何 citation 指令
# 这样杜绝 Gemini 幻觉添加虚假引用，且来源文字 100% 准确

def _citation_font(size: int = 20):
    candidates = [
        "/System/Library/Fonts/STHeiti Light.ttc",
        "/System/Library/Fonts/PingFang.ttc",
        "/System/Library/Fonts/Supplemental/Arial.ttf",
    ]
    for path in candidates:
        try:
            return ImageFont.truetype(path, size)
        except Exception:
            continue
    return ImageFont.load_default()

def burn_citation(img_path: str, citation: str, out_path: str):
    """
    用 PIL 把真实 citation 文字烧录到幻灯片右下角。
    先用白色矩形覆盖底部区域（清除 Gemini 可能留下的任何文字），
    再写入正确来源。适用于所有幻灯片类型。
    """
    img = Image.open(img_path).convert("RGB")
    W, H = img.size
    draw = ImageDraw.Draw(img)
    font = _citation_font(int(H * 0.023))

    bbox = draw.textbbox((0, 0), citation, font=font)
    text_w = bbox[2] - bbox[0]
    text_h = bbox[3] - bbox[1]
    pad = 8
    # 白色矩形覆盖底部右侧（防止 Gemini 幻觉残留）
    draw.rectangle([W - text_w - 30 - pad, H - text_h - 14 - pad, W - 2, H - 2],
                   fill=(255, 255, 255))
    draw.text((W - 22, H - 14), citation, fill=(120, 120, 120), font=font, anchor="rb")
    img.save(out_path, quality=95)
```

### 4.5 Figure 合成（figure / lit-figure 类型）

```python
def composite_figures(frame_path: str, figures: list, output_path: str):
    """
    将真实图表合成到 AI 生成的画框中。
    注意：不在此函数处理 citation，由 burn_citation 单独调用。
    """
    frame = Image.open(frame_path).convert("RGB")
    W, H = frame.size
    TITLE_BOTTOM = int(H * 0.13)
    CITATION_TOP = int(H * 0.88)
    FIG_Y = TITLE_BOTTOM + 4
    FIG_H = CITATION_TOP - FIG_Y - 8

    draw = ImageDraw.Draw(frame)
    # ✅ 硬编码白色，不用 frame.getpixel()，避免 Gemini 深色帧污染图表区
    draw.rectangle([0, TITLE_BOTTOM, W, CITATION_TOP], fill=(255, 255, 255))

    def fit(img, mw, mh):
        r = min(mw/img.width, mh/img.height)
        return img.resize((int(img.width*r), int(img.height*r)), Image.LANCZOS)

    paths = [f["path"] for f in figures if Path(f["path"]).exists()]
    if len(paths) == 1:
        fig = fit(Image.open(paths[0]).convert("RGB"), W-60, FIG_H)
        frame.paste(fig, ((W-fig.width)//2, FIG_Y+(FIG_H-fig.height)//2))
    elif len(paths) == 2:
        hw = (W - 80) // 2
        for i, p in enumerate(paths):
            fig = fit(Image.open(p).convert("RGB"), hw, FIG_H)
            x = 20 + i*(hw+40) + (hw-fig.width)//2
            frame.paste(fig, (x, FIG_Y+(FIG_H-fig.height)//2))

    frame.save(output_path, quality=95)
```

**Phase 4 执行顺序（每张幻灯片）**：

```python
# Step A：Gemini 生成背景图（prompt 里绝对不含任何 citation 指令）
ok = gemini_image(prompt, raw_path)

# Step B：figure/lit-figure 类型 → 合成真实图表（白色填充区）
if slide_type in ("lit-figure", "figure") and slide.figures:
    composite_figures(raw_path, slide.figures, final_path)

# Step C：有 citation 的所有幻灯片 → PIL 烧录文献来源（覆盖 Gemini 可能的任何文字）
if slide.citation:
    burn_citation(final_path, slide.citation, final_path)
```

### 4.5 进度报告

逐张生成，实时输出（不等用户）：
```
生成中... [3/12] 认知负荷对比图（文献图合成）✅
生成中... [4/12] Zhang et al. 核心发现页 ✅
```

---

## Phase 5：组装 PPTX（自动）

```python
from pptx import Presentation
from pptx.util import Inches
import os

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
文献：[N篇，共M张图表已提取]
位置：slide-deck/{slug}/
共 N 张：01-cover / 02-... / ... / NN-back-cover

PPTX：slide-deck/{slug}/{slug}.pptx
大纲：slide-deck/{slug}/outline.yaml
文献图表：slide-deck/{slug}/figs/

如需调整某张，告诉我页码，我重新生成。
```

---

---

## Phase 4B：python-pptx 原生渲染（学术文献模式推荐）

**触发条件**：输入是学术 PDF 文献（单篇或多篇）。

此模式**不调用 Gemini 生图**，直接用 python-pptx 构建矢量化幻灯片。优势：精确控制布局、文字不溢出、颜色一致、无 AI 幻觉。

### 核心工具函数

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN
from PIL import Image

# 色彩常量（双色系，不扩展）
C_NAVY  = RGBColor(0x1A, 0x2E, 0x4A)
C_TEAL  = RGBColor(0x00, 0x8C, 0x8C)
C_WHITE = RGBColor(0xFF, 0xFF, 0xFF)
C_GRAY  = RGBColor(0x66, 0x66, 0x66)
C_LIGHT = RGBColor(0xF0, 0xF6, 0xF6)   # 卡片背景
C_LGRAY = RGBColor(0xCC, 0xCC, 0xCC)   # 次要元素

def R(slide, x, y, w, h, color):
    """添加填充矩形（无边框）"""
    s = slide.shapes.add_shape(1, x, y, w, h)
    s.fill.solid(); s.fill.fore_color.rgb = color
    s.line.fill.background()

def T(slide, text, x, y, w, h, sz=12, bold=False,
      color=C_NAVY, align=PP_ALIGN.CENTER,
      italic=False, wrap=True):
    """添加文字框，wrap=False 防止短标签换行"""
    tb = slide.shapes.add_textbox(x, y, w, h)
    tf = tb.text_frame
    tf.word_wrap = wrap        # 数值标签必须设 wrap=False
    p = tf.paragraphs[0]; p.alignment = align
    r = p.add_run(); r.text = text
    r.font.size = Pt(sz); r.font.bold = bold
    r.font.italic = italic; r.font.color.rgb = color
```

### 图片预处理（裁掉英文 caption）

```python
def crop_figure_caption(img_path: str, out_path: str, keep_ratio: float = 0.67) -> tuple:
    """
    裁掉图片底部的英文 caption，只保留 panels + 图例。
    keep_ratio：保留顶部百分比（默认67%，含图例行）。
    返回裁切后的 (width_px, height_px)。
    """
    img = Image.open(img_path)
    w, h = img.size
    crop_h = int(h * keep_ratio)
    cropped = img.crop((0, 0, w, crop_h))
    cropped.save(out_path)
    return w, crop_h
```

### 图片布局计算

```python
def fit_image(img_w_px, img_h_px, max_w_in, max_h_in, slide_w_in=13.333):
    """
    计算图片在 slide 中的最优尺寸和居中位置。
    返回 (fw, fh, fx, fy) 单位 EMU。
    """
    max_w = Inches(max_w_in)
    max_h = Inches(max_h_in)
    ratio = min(max_w / img_w_px, max_h / img_h_px)
    fw = int(img_w_px * ratio)
    fh = int(img_h_px * ratio)
    fx = int((Inches(slide_w_in) - fw) / 2)
    return fw, fh, fx
```

### 侧边数据卡片（填充 16:9 留白）

当图片宽高比 < 2.0（如四 panel 2×2 图）时，两侧会有留白。用次级证据卡片填充：

```python
def render_side_cards(slide, story_line, fig_top, fig_bottom, fig_left_x, fig_right_x):
    """
    左侧：专注/过程类指标（对应图的左半 panels）
    右侧：主要效应量 + 样本信息（对应图的右半 panels 或核心发现）

    每侧分三区：上（次级数据）/ 中（主卡片）/ 下（协议或可信度锚点）
    """
    CW = Inches(1.95)
    LX = Inches(0.1)
    RX = Inches(13.333) - Inches(2.13)

    fig_h   = fig_bottom - fig_top
    TOP_H   = Inches(1.25)
    MID_H   = Inches(2.5)
    BOT_H   = Inches(1.2)
    GAP     = Inches(0.07)

    top_y = fig_top
    mid_y = fig_top + TOP_H + GAP
    bot_y = fig_bottom - BOT_H

    # 所有卡片：白底 + 顶条（左侧用Teal，右侧用Navy）
    # 参见附录C参考模板的完整实现
```

### 图表矢量重绘（v4.1 新增，解决 PDF 栅格模糊问题）

学术 PDF 中嵌入的图通常是压缩 JPEG（92KB，约 1357×810px）——放大到 16:9 slide 时会明显模糊。**矢量重绘**从根本上解决这个问题：用 AI 读取图中的数据点，再用 matplotlib 完整重画为真矢量图（SVG）。

#### 步骤 1：验证 PDF 图是否为栅格

```python
import fitz

def check_figure_type(pdf_path: str, page_num: int) -> str:
    """返回 'vector' 或 'raster'"""
    doc = fitz.open(pdf_path)
    page = doc[page_num]
    drawings = page.get_drawings()
    images = page.get_images(full=True)
    # 只有 1 个 drawing（边框）且有嵌入 JPEG → 栅格
    if len(drawings) <= 2 and images:
        xref = images[0][0]
        base = doc.extract_image(xref)
        return f"raster ({base['ext']}, {base['width']}x{base['height']}, {len(base['image'])//1024}KB)"
    return "vector"
```

#### 步骤 2：用 LLM Vision 提取数据点

```python
from llm_router import call
import base64, json

def extract_figure_data(img_path: str) -> dict:
    """
    用视觉模型读取折线图/柱状图的数据点。
    返回严格 JSON，含各子图的均值和误差棒。
    """
    with open(img_path, "rb") as f:
        img_b64 = base64.b64encode(f.read()).decode()

    prompt = """图是学术论文的多子图折线图（a/b/c/d）。
请精确读取每个子图的：标题、Y轴标签、Y轴范围、每条线每个时间点的均值和误差棒上下界。
只返回JSON，格式：
{"a":{"title":"...","y_label":"...","y_min":...,"y_max":...,
  "ctrl_mean":[p,m,po],"ctrl_upper":[p,m,po],"ctrl_lower":[p,m,po],
  "int_mean":[p,m,po],"int_upper":[p,m,po],"int_lower":[p,m,po]},
 "b":{...},"c":{...},"d":{...}}"""

    messages = [{"role": "user", "content": [
        {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{img_b64}"}},
        {"type": "text", "text": prompt}
    ]}]

    text, _ = call.chat(messages, model="google_pro/gemini-2.5-flash", max_tokens=4000)
    if "```" in text:
        text = [p for p in text.split("```") if p.strip().startswith("{")][0]
    return json.loads(text.strip())
```

> **精度验证**：Pre-intervention 值必须用论文 Table 2 的精确基线数据校正；Mid/Post 值来自图像读取（误差约 ±3-5%，趋势方向 100% 正确）。

#### 步骤 3：matplotlib 矢量重绘

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
import numpy as np

def redraw_figure_vector(data: dict, out_png: str, out_svg: str,
                          x_labels=None, c_ctrl="#4472C4", c_int="#C00000"):
    """
    从提取的数据字典重绘多 panel 折线图为真矢量。
    data: extract_figure_data() 的返回值
    """
    if x_labels is None:
        x_labels = ["Pre-\nintervention", "Mid-\nintervention", "Post-\nintervention"]

    plt.rcParams.update({
        'font.family': 'Arial', 'font.size': 10,
        'axes.spines.top': False, 'axes.spines.right': False,
        'axes.linewidth': 1.0,
    })

    panels = list(data.keys())  # ['a', 'b', 'c', 'd']
    n_rows = (len(panels) + 1) // 2
    fig, axes = plt.subplots(n_rows, 2, figsize=(9, 3*n_rows), constrained_layout=True)
    axes_flat = axes.flatten() if n_rows > 1 else list(axes)

    for ax, key in zip(axes_flat, panels):
        d = data[key]
        xpts = list(range(len(d["ctrl_mean"])))
        cm = np.array(d["ctrl_mean"])
        cu = np.array(d["ctrl_upper"]) - cm
        cd = cm - np.array(d["ctrl_lower"])
        im = np.array(d["int_mean"])
        iu = np.array(d["int_upper"]) - im
        id_ = im - np.array(d["int_lower"])

        ax.errorbar(xpts, cm, yerr=[cd, cu], color=c_ctrl, marker='o',
                    markersize=5, linewidth=1.5, capsize=4, label='Control group')
        ax.errorbar(xpts, im, yerr=[id_, iu], color=c_int, marker='s',
                    markersize=5, linewidth=1.5, capsize=4, label='Intervention group')

        ax.set_xlim(-0.4, len(xpts) - 0.6)
        ax.set_ylim(d["y_min"], d["y_max"])
        ax.set_xticks(xpts)
        ax.set_xticklabels(x_labels[:len(xpts)], fontsize=9)
        ax.set_ylabel(d["y_label"], fontsize=10)
        ax.set_title(d["title"], fontsize=10, loc='left', pad=4)
        # 子图标签 a) b) c) d)
        ax.text(-0.12, 1.04, f"{key})", transform=ax.transAxes,
                fontsize=11, fontweight='bold', va='top')
        ax.yaxis.grid(True, linestyle='--', alpha=0.4, linewidth=0.6, color='gray')
        ax.set_axisbelow(True)

    handles, lbls = axes_flat[-1].get_legend_handles_labels()
    fig.legend(handles, lbls, loc='lower center', ncol=2, fontsize=9,
               frameon=False, bbox_to_anchor=(0.5, -0.04))

    fig.savefig(out_svg, format='svg', bbox_inches='tight', dpi=300)
    fig.savefig(out_png, format='png', bbox_inches='tight', dpi=300)
    plt.close()
```

#### 完整调用流程

```python
# 1. 提取原始嵌入图
img_path = "/tmp/fig5_original.jpeg"
doc.extract_image(xref)  # 保存原始 JPEG

# 2. AI 读取数据点（可选：用 Table 2 校正 Pre 基线值）
fig_data = extract_figure_data(img_path)

# 3. 矢量重绘（SVG 无限缩放，PNG 300dpi 备用）
redraw_figure_vector(fig_data, "/tmp/fig5_vector.png", "/tmp/fig5_vector.svg")

# 4. 嵌入 slide（用 PNG 版本，对应 fit_image 函数）
fit_image(slide, "/tmp/fig5_vector.png", FIG_X, FIG_Y, FIG_W, FIG_H)
```

> **注意**：适用于折线图、带误差棒的点图。柱状图需调整 `ax.bar()` 渲染逻辑。

---

### slide 类型对应渲染函数

| 类型 | 渲染策略 |
|------|---------|
| `cover` | 深蓝顶栏 + 完整标题（自动换行）+ teal 底栏 + DOI |
| `tension` | 白底 + 大字陈述问题 + teal 方法信息卡 |
| `method` | 全版方法流程图（论文 Figure 1）+ 标题 |
| `results-card` | 标题栏 + 主图（裁切版）+ 侧边6卡片 + 概要句 + 引用栏 |
| `conclusion` | 三栏卡片（每栏一条结论）+ 大字主张 |
| `summary` | teal标题栏 + 序号要点列表（均匀分布） |

---

## 附录 C：参考模板（v4.0 基准 slide）

**基于 Jiang et al. (2026) 学术汇报的第一张参考 slide（results-card 类型）**

生成脚本：`/tmp/nat_adhd_slide_v10_final.pptx`（2026-04-15 矢量重绘版验证通过）

**布局规格**：
- 幻灯片尺寸：13.333" × 7.5"（16:9）
- 标题栏：0"~1.1"，Navy 背景，左侧 Teal 竖条 0.08"，白字 24pt bold
- 主图：1.15" 起，居中，裁切版（67% crop，去掉英文 caption），实际宽 8.51"
- 两侧卡片列宽：1.95"，左列 x=0.1"，右列 x=11.2"
- 六卡片布局：
  - 左上（1.25"高）：次级数据，前/后对比柱，C_LIGHT 背景，Teal 顶条
  - 左中（2.5"高）：主卡片，↑2× 大字，白底，Teal 顶条+左竖条
  - 左下（1.2"高）：协议信息，白底，Teal 顶条+左竖条
  - 右上（1.25"高）：次效应量，C_LIGHT 背景，Navy 顶条
  - 右中（2.5"高）：主效应量，0.68 大字，白底，Navy 顶条+左竖条
  - 右下（1.2"高）：RCT 信息，白底，Navy 顶条+左竖条
- 概要句：图下方 0.07" gap，11pt 灰色斜体，居中，宽 8.7"（夹在两侧卡片间）
- 引用栏：7.0"~7.5"，Teal 背景，白字 10pt 右对齐

**关键技术规范**：
- 所有数值标签（百分比、数字）：`wrap=False`，textbox 宽度 ≥ 实际文字宽 + 0.15"
- 柱图布局：标签列(0.25") + 柱区(1.05") + 数值列(0.52")，总计 1.82" < 卡片宽 1.95"
- 图片裁切：`int(ih * 0.67)` 保留图例行，`int(ih * 0.585)` 只保留 panels

---

## 局部工作流

```bash
/slide-gen paper.pdf                         # 单篇文献 → 总结PPT
/slide-gen ~/papers/                         # 文件夹批量文献 → 总结PPT
/slide-gen content.md                        # 纯文字内容 → 完整流程
/slide-gen content.md --style academic-dark  # 指定风格
/slide-gen content.md --slides 12            # 指定页数
/slide-gen content.md --outline-only         # 只生成大纲
/slide-gen slide-deck/topic/ --images-only   # 从现有 prompts/ 重新生图
/slide-gen slide-deck/topic/ --regenerate 3  # 只重新生成第3张
```

---

## 附录 A：风格提取（用户提供参考PPT时）

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
├── source.md                  （纯文字模式）
├── figures_manifest.yaml      （PDF文献模式，自动生成）
├── outline.yaml
├── slide_style.json           （风格提取时存在）
├── figs/                      （从PDF提取的原始图表）
│   ├── p1_fig01_p3.png
│   └── p2_fig01_p4.png
├── prompts/
│   ├── 01-slide-cover.md
│   └── 02-slide-{name}.md
├── 01-slide-cover.png
├── 02-slide-{name}.png
└── {topic-slug}.pptx
```
