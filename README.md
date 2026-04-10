# Terry-slide-gen

> A Claude Code skill for generating professional slide decks with AI — one intake, fully autonomous execution.

[![GitHub stars](https://img.shields.io/github/stars/TerryFYL/Terry-slide-gen?style=social)](https://github.com/TerryFYL/Terry-slide-gen/stargazers)

## What it does

- **One-shot intake**: tells you what it needs upfront, then runs without interruption
- **16 visual style presets** (scientific, corporate, minimal, chalkboard, etc.)
- **Style extraction**: clone the look of an existing PPT or image
- **Figure compositing**: embed real research charts (PNG/PDF) into slides via PIL
- **Auto 16:9 correction** on all generated images
- **Outputs**: individual PNG slides + assembled `.pptx`

Powered by Gemini image generation (`google.genai` SDK).

## Installation

```bash
# 1. Clone into your Claude Code global skills directory
git clone https://github.com/TerryFYL/Terry-slide-gen.git ~/.claude/skills/slide-gen

# 2. Install Python dependencies
pip install google-genai python-pptx pillow pyyaml
```

Then add your Gemini API key. The skill reads from `~/.openclaw/credentials/.env`:

```bash
echo 'GOOGLE_GENAI_KEY="your-key-here"' >> ~/.openclaw/credentials/.env
```

Or set the environment variable directly:

```bash
export GOOGLE_GENAI_KEY="your-key-here"
```

Get a free Gemini API key at [Google AI Studio](https://aistudio.google.com/).

> ⭐ If this skill saves you time, consider giving it a star: https://github.com/TerryFYL/Terry-slide-gen

## Usage

Invoke the skill in Claude Code:

```
/slide-gen
```

You'll be asked once for:
1. **Content** — topic outline or paste source text (required)
2. **Style** — reference image/PPT or style preference (optional, auto-inferred)
3. **Real figures** — paths to chart files to embed (optional)

Then Claude runs the full pipeline autonomously and delivers a `.pptx`.

### Command options

```bash
/slide-gen content.md                        # full pipeline
/slide-gen content.md --style scientific     # skip style inference
/slide-gen content.md --slides 12            # specify slide count
/slide-gen content.md --outline-only         # outline only, no images
/slide-gen slide-deck/topic/ --images-only   # regenerate images from existing prompts
/slide-gen slide-deck/topic/ --regenerate 3  # regenerate slide #3 only
```

## Style presets

| Preset | Best for |
|--------|---------|
| `scientific` | Academic talks, medical, neuroscience |
| `intuition-machine` | Technical docs, bilingual |
| `blueprint` | Architecture, system design |
| `corporate` | Business proposals, investor decks |
| `minimal` | Executive briefings |
| `chalkboard` | Teaching, classroom |
| `editorial-infographic` | Science communication |
| `bold-editorial` | Product launches, keynotes |
| `notion` | Product demos, SaaS |
| `dark-atmospheric` | Entertainment, gaming |
| `sketch-notes` | Educational tutorials |
| `watercolor` | Lifestyle, wellness |

## Output structure

```
slide-deck/{topic-slug}/
├── source.md
├── outline.yaml
├── slide_style.json    (when style extraction used)
├── prompts/
│   ├── 01-slide-cover.md
│   └── ...
├── 01-slide-cover.png
├── ...
└── {topic-slug}.pptx
```

## Requirements

- Claude Code (CLI)
- Python 3.10+
- `google-genai`, `python-pptx`, `pillow`, `pyyaml`
- Gemini API key (free tier works)

## License

MIT
