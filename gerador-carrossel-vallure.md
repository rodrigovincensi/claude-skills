---
name: instagram-carousel-vallure
description: >
  Creates high-quality Instagram carousels as swipeable HTML previews with
  export-ready slides (1080×1350px PNG). Handles the full workflow: brand
  setup, slide copy, visual design system (colors, fonts, components), HTML
  generation, and Playwright-based export. Use this skill whenever the user
  asks to create, design, or generate an Instagram carousel, carrossel,
  slides para Instagram, or any Instagram multi-image post — even if they
  don't explicitly say "carousel" or "skill". Also trigger for requests to
  "create a post with multiple slides", "fazer carrossel", or "exportar slides
  para o Instagram".
---

# Instagram Carousel Generator

Generates fully self-contained, swipeable HTML carousels where every slide is
designed to be exported as an individual 1080×1350px PNG for Instagram.

---

## Step 1: Collect Brand Details

If brand assets exist, skip manual brand collection.

## Brand Assets

Always prioritize assets from:

assets/

Available assets:
- brand-guide.md
- logo_vallure_odontologia_gold.png
- logo_vallure_odontologia_white.png
- logo_vallure_only_black.png
- logo_vallure_only_gold.png
- logo_vallure_only_white.png
- vallure-branding.pdf
- fotos-clinica/
- fotos-dentista/
- before-after/

Photo priority order:
1. Web photos (Unsplash, Pexels) — default choice for hero and background slides; brings variety and avoids repetition
2. Before/after images from `assets/after-before/` — only when the carousel is specifically about transformation
3. Dentist photos from `assets/fotos-dentista/` — use when the carousel calls for authority, trust, expertise, or human connection (e.g. "meet the dentist", professional credibility slides, emotional CTAs)
4. Clinic photos from `assets/fotos-clinica/` — only when the carousel is about the clinic space itself
4. No photo — valid option; use gradient, texture, or color-block background instead when a photo doesn't add value

**Clinic photos are NOT required in every carousel.** Do not force them into slides where they don't fit naturally.

When downloading web photos:
- Use the secondary agent (Explore) with a short, focused prompt — specifies search breadth "quick" and asks for one direct URL
- Download with Python `urllib.request` directly to `assets/web_*.jpeg`
- Move downloaded files to `source/` on export alongside other files
- Best for: lifestyle, emotions, abstract textures, CTA backgrounds, smiles, dental aesthetics

Always use:
- brand-guide.md as the strategic branding source
- vallure-branding.pdf as the visual identity reference

Use vallure-branding.pdf for:
- composition
- luxury direction
- logo spacing
- visual consistency
- premium aesthetics
- photography style
- layout inspiration

If assets exist, do not ask the user for:
- branding
- colors
- logo
- typography
- visual direction

---

## Auto Editorial Mode

If the user requests a carousel without providing a topic, DO NOT ask for a topic.

Automatically choose:
- a strategic topic
- an emotional angle
- a strong hook
- a carousel structure

Focus on:
- implants
- protocolo
- fixed prosthesis
- self-esteem
- fear reduction
- transformation
- patient doubts
- authority

Avoid:
- exaggerated promises
- aggressive sales language
- unrealistic claims

Always:
1. Generate preview first
2. Wait for approval
3. Export only after confirmation

---


## Handling User-Provided Images

**This section applies from the very first HTML generation — not only during export.**

When the user provides an image file path (e.g., `/home/user/gestante.png`, `/mnt/user-data/uploads/foto.jpg`):

### ⚠️ Critical Rules

1. **NEVER use relative paths** (`gestante.png`) — they break in every browser context except the exact folder the HTML lives in.
2. **NEVER use `background: url(filepath)`** — leads to 1.5MB+ base64 inline strings that crash the browser parser.
3. **ALWAYS embed as base64 `data:` URI** — works in preview, export, and any environment.
4. **ALWAYS generate the HTML via Python** (`Path.write_text()`) — shell heredocs interpolate `$` and backticks, corrupting base64 strings.

### Step-by-step: embed an image

```bash
# 1. Check the actual file format (extension may lie)
file /path/to/image.png
```

```python
import base64
from pathlib import Path

# 2. Read and encode
img_path = Path("/path/to/image.png")
# Use "image/jpeg" if `file` command says JPEG, else "image/png"
mime = "image/jpeg"  # or "image/png"
b64 = base64.b64encode(img_path.read_bytes()).decode()
data_uri = f"data:{mime};base64,{b64}"

# 3. Inject into HTML template as a Python variable — never via shell
html = f"""
<div style="position:relative;width:100%;height:100%;">
  <img src="{data_uri}"
       style="position:absolute;inset:0;width:100%;height:100%;object-fit:cover;z-index:0;">
  <div style="position:absolute;inset:0;background:rgba(255,255,255,0.35);z-index:1;"></div>
  <!-- slide content goes here, z-index:2 -->
</div>
"""

Path("/home/claude/carousel.html").write_text(html, encoding="utf-8")
```

### Image as slide background (most common use)

```html
<!-- Inside the slide div, before any content -->
<img src="{data_uri}"
     style="position:absolute;inset:0;width:100%;height:100%;object-fit:cover;z-index:0;">
<!-- Semi-transparent overlay so text stays readable -->
<div style="position:absolute;inset:0;background:rgba(255,255,255,0.35);z-index:1;"></div>
<!-- All slide content must have z-index:2 or higher -->
```

For dark slides, use `rgba(0,0,0,0.45)` as the overlay instead.

### Common image mistakes to avoid

| Mistake | What goes wrong | Fix |
|---------|----------------|-----|
| `<img src="gestante.png">` | Broken image — relative path only works if HTML and image share the same folder | Always use base64 `data:` URI |
| `background: url('data:...')` inline with 1.5MB base64 | Browser parser crash, 1.3M token context | Use `<img>` tag with `object-fit:cover` |
| Generating HTML via shell `echo` or heredoc | `$` and backtick characters in base64 get interpolated and corrupt the string | Always use Python `Path.write_text()` |
| Assuming `.png` extension = PNG format | File may actually be JPEG; wrong MIME type breaks rendering | Run `file` command to detect actual format |

---

## Step 2: Derive the Full Color System

From the user's **single primary brand color**, generate the full 6-token palette:

```
BRAND_PRIMARY   = {user's color}                    // Main accent — progress bar, icons, tags
BRAND_LIGHT     = {primary lightened ~20%}           // Secondary accent — tags on dark, pills
BRAND_DARK      = {primary darkened ~30%}            // CTA text, gradient anchor
LIGHT_BG        = {warm or cool off-white}           // Light slide background (never pure #fff)
LIGHT_BORDER    = {slightly darker than LIGHT_BG}    // Dividers on light slides
DARK_BG         = {near-black with brand tint}       // Dark slide background
```

**Rules for deriving colors:**
- LIGHT_BG: tinted off-white complementing the primary (warm → warm cream, cool → cool gray-white)
- DARK_BG: near-black with subtle brand tint (warm → #1A1918, cool → #0F172A)
- LIGHT_BORDER: always ~1 shade darker than LIGHT_BG
- Brand gradient: `linear-gradient(165deg, BRAND_DARK 0%, BRAND_PRIMARY 50%, BRAND_LIGHT 100%)`

---

## Step 3: Set Up Typography

Based on the user's font preference, pick a **heading font** and **body font** from Google Fonts.

| Style | Heading Font | Body Font |
|-------|-------------|-----------|
| Editorial / premium | Playfair Display | DM Sans |
| Modern / clean | Plus Jakarta Sans (700) | Plus Jakarta Sans (400) |
| Warm / approachable | Lora | Nunito Sans |
| Technical / sharp | Space Grotesk | Space Grotesk |
| Bold / expressive | Fraunces | Outfit |
| Classic / trustworthy | Libre Baskerville | Work Sans |
| Rounded / friendly | Bricolage Grotesque | Bricolage Grotesque |

**Font size scale (fixed across all brands):**
- Headings: 28–34px, weight 600, letter-spacing -0.3 to -0.5px, line-height 1.1–1.15
- Body: 14px, weight 400, line-height 1.5–1.55
- Tags/labels: 10px, weight 600, letter-spacing 2px, uppercase
- Step numbers: heading font, 26px, weight 300
- Small text: 11–12px

Apply via CSS classes `.serif` (heading font) and `.sans` (body font) throughout all slides.

---

## Content Modes

There are two carousel modes:

### Premium Carousel

Purpose:
- emotional impact
- authority
- storytelling
- luxury positioning
- strong visual experience

Characteristics:
- emotional copywriting
- cinematic feel
- transformation-focused
- sophisticated pacing
- premium layouts
- deeper storytelling

Best for:
- self-esteem
- smile transformation
- emotional journeys
- before/after
- confidence
- premium positioning
- patient stories

---

### Light Carousel

Purpose:
- fast engagement
- educational content
- daily consistency
- quick consumption
- FAQ content

Characteristics:
- shorter copy
- direct explanations
- faster pacing
- objective communication
- highly scannable structure

Best for:
- FAQs
- myths
- quick tips
- implant doubts
- educational explanations
- mini guides

---

## Mode Selection Rules

If the user explicitly says:

- "premium carousel"
→ always use Premium Carousel mode

- "light carousel"
→ always use Light Carousel mode

Examples:

"Create a premium carousel about implants"
→ emotional / transformational / authority angle about implants

"Create a light carousel about implants"
→ educational / FAQ / quick-consumption content about implants

---

If the user provides ONLY a topic:

Example:
"Create a carousel about implants"

Claude must automatically decide:
- whether the topic works better as Premium or Light
- based on emotional potential, educational value and engagement potential

---

If the user provides NO topic:

Example:
"Create a carousel"

Claude must automatically:
1. choose the topic
2. choose the best mode
3. choose the emotional angle or educational angle
4. generate the full carousel preview

---

**Strictly alternate modes across generations.** Before choosing the mode, check the most recently created folder in `exports/` and pick the OPPOSITE type:
- Last export was `light` → generate `premium`
- Last export was `premium` → generate `light`
- No exports yet → start with `light`

Never generate two consecutive carousels of the same type.

---

## Slide 1 — Hook Rules

The first slide must stop the scroll in under 1 second. Prioritize these formats:

| Hook format | Example |
|---|---|
| Afirmação polêmica | "Você está usando IA errado" |
| Número + benefício | "7 ferramentas que substituem seu designer" |
| Pergunta que dói | "Por que seus carrosséis têm 0 salvamentos?" |
| Resultado concreto | "Esse post gerou 4.200 seguidores em 3 dias" |
| Inversão de expectativa | "Mais esforço no design = menos alcance" |

**Rules:**
- Never start with the brand name as headline
- Visual proof on Slide 1 whenever possible (screenshot, result, real number)
- Hook must promise value that the following slides deliver

---

## Slide Sequences

### Standard (7 slides — default)

| # | Type | Background | Purpose |
|---|------|------------|---------|
| 1 | Hero | LIGHT_BG | Hook — bold statement, logo lockup, optional watermark |
| 2 | Problem | DARK_BG | Pain point — what's broken, frustrating, or outdated |
| 3 | Solution | Brand gradient | The answer — what solves it, optional quote/prompt box |
| 4 | Features | LIGHT_BG | What you get — feature list with icons |
| 5 | Details | DARK_BG | Depth — customization, specs, differentiators |
| 6 | How-to | LIGHT_BG | Steps — numbered workflow or process |
| 7 | CTA | Brand gradient | Call to action — logo, tagline, CTA button. **No arrow. Full progress bar.** |

### Listicle (5–10 slides)

| # | Type | Background |
|---|------|------------|
| 1 | Hero | LIGHT_BG |
| 2–N | Item N | Alternating LIGHT/DARK |
| Last | CTA | Brand gradient |

Use for: "X ferramentas", "X erros", "X dicas"

### Tutorial (7 slides)

| # | Type | Background |
|---|------|------------|
| 1 | Hero | LIGHT_BG |
| 2 | Contexto / Por quê | DARK_BG |
| 3–5 | Passo 1, 2, 3 | Alternating |
| 6 | Resultado esperado | DARK_BG |
| 7 | CTA | Brand gradient |

### Comparação (5 slides)

| # | Type | Background |
|---|------|------------|
| 1 | Hero (o que será comparado) | LIGHT_BG |
| 2 | Opção A | LIGHT_BG |
| 3 | Opção B | DARK_BG |
| 4 | Veredicto | Brand gradient |
| 5 | CTA | DARK_BG |

**General rules for all sequences:**
- Start with a hook — first slide must stop the scroll
- End CTA on brand gradient — no swipe arrow, progress bar at 100%
- Alternate light and dark backgrounds for visual rhythm
- Adapt sequence to topic — not every carousel needs all slides

---

## Slide Architecture

### Format
- Aspect ratio: **4:5** (Instagram carousel standard)
- Each slide is self-contained — all UI elements baked into the image
- Alternate LIGHT_BG and DARK_BG backgrounds for visual rhythm

### Required Elements on Every Slide

#### 1. Progress Bar (bottom of every slide)

Shows position in the carousel. Fills as user swipes.

- Position: absolute bottom, full width, 28px horizontal padding, 20px bottom padding
- Track: 3px height, rounded corners
- Fill width: `((slideIndex + 1) / totalSlides) * 100%`
- Light slides: `rgba(0,0,0,0.08)` track, `BRAND_PRIMARY` fill, `rgba(0,0,0,0.3)` counter
- Dark slides: `rgba(255,255,255,0.12)` track, `#fff` fill, `rgba(255,255,255,0.4)` counter
- Counter label beside the bar: "1/7" format, 11px, weight 500

```javascript
function progressBar(index, total, isLightSlide) {
  const pct = ((index + 1) / total) * 100;
  const trackColor = isLightSlide ? 'rgba(0,0,0,0.08)' : 'rgba(255,255,255,0.12)';
  const fillColor = isLightSlide ? BRAND_PRIMARY : '#fff'; // use actual BRAND_PRIMARY value
  const labelColor = isLightSlide ? 'rgba(0,0,0,0.3)' : 'rgba(255,255,255,0.4)';
  return `<div style="position:absolute;bottom:0;left:0;right:0;padding:16px 28px 20px;z-index:10;display:flex;align-items:center;gap:10px;">
    <div style="flex:1;height:3px;background:${trackColor};border-radius:2px;overflow:hidden;">
      <div style="height:100%;width:${pct}%;background:${fillColor};border-radius:2px;"></div>
    </div>
    <span style="font-size:11px;color:${labelColor};font-weight:500;">${index + 1}/${total}</span>
  </div>`;
}
```

⚠️ **Important:** Always replace `BRAND_PRIMARY` with the actual hex value before rendering. Never leave it as a variable name in the HTML output.

#### 2. Swipe Arrow (right edge — every slide EXCEPT the last)

Subtle chevron guiding the user to keep swiping. Removed on the last slide.

- Position: absolute right, full height, 48px wide
- Background: gradient fade transparent → subtle tint
- Chevron: 24×24 SVG, rounded strokes
- Light slides: `rgba(0,0,0,0.06)` bg, `rgba(0,0,0,0.25)` stroke
- Dark slides: `rgba(255,255,255,0.08)` bg, `rgba(255,255,255,0.35)` stroke

```javascript
function swipeArrow(isLightSlide) {
  const bg = isLightSlide ? 'rgba(0,0,0,0.06)' : 'rgba(255,255,255,0.08)';
  const stroke = isLightSlide ? 'rgba(0,0,0,0.25)' : 'rgba(255,255,255,0.35)';
  return `<div style="position:absolute;right:0;top:0;bottom:0;width:48px;z-index:9;display:flex;align-items:center;justify-content:center;background:linear-gradient(to right,transparent,${bg});">
    <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
      <path d="M9 6l6 6-6 6" stroke="${stroke}" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
    </svg>
  </div>`;
}
```

---

## Reusable Components

### Strikethrough pills
```html
<span style="font-size:11px;padding:5px 12px;border:1px solid rgba(255,255,255,0.1);border-radius:20px;color:#6B6560;text-decoration:line-through;">{Old tool}</span>
```

### Tag pills
```html
<span style="font-size:11px;padding:5px 12px;background:rgba(255,255,255,0.06);border-radius:20px;color:{BRAND_LIGHT};">{Label}</span>
```

### Prompt / quote box
```html
<div style="padding:16px;background:rgba(0,0,0,0.15);border-radius:12px;border:1px solid rgba(255,255,255,0.08);">
  <p class="sans" style="font-size:13px;color:rgba(255,255,255,0.5);margin-bottom:6px;">{Label}</p>
  <p class="serif" style="font-size:15px;color:#fff;font-style:italic;line-height:1.4;">"{Quote text}"</p>
</div>
```

### Feature list
```html
<div style="display:flex;align-items:flex-start;gap:14px;padding:10px 0;border-bottom:1px solid {LIGHT_BORDER};">
  <span style="color:{BRAND_PRIMARY};font-size:15px;width:18px;text-align:center;">{icon}</span>
  <div>
    <span class="sans" style="font-size:14px;font-weight:600;color:{DARK_BG};">{Label}</span>
    <span class="sans" style="font-size:12px;color:#8A8580;">{Description}</span>
  </div>
</div>
```

### Numbered steps
```html
<div style="display:flex;align-items:flex-start;gap:16px;padding:14px 0;border-bottom:1px solid {LIGHT_BORDER};">
  <span class="serif" style="font-size:26px;font-weight:300;color:{BRAND_PRIMARY};min-width:34px;line-height:1;">01</span>
  <div>
    <span class="sans" style="font-size:14px;font-weight:600;color:{DARK_BG};">{Step title}</span>
    <span class="sans" style="font-size:12px;color:#8A8580;">{Step description}</span>
  </div>
</div>
```

### Color swatches
```html
<div style="width:32px;height:32px;border-radius:8px;background:{color};border:1px solid rgba(255,255,255,0.08);"></div>
```

### CTA button (final slide only)
```html
<div style="display:inline-flex;align-items:center;gap:8px;padding:12px 28px;background:{LIGHT_BG};color:{BRAND_DARK};font-family:'{BODY_FONT}',sans-serif;font-weight:600;font-size:14px;border-radius:28px;">
  {CTA text}
</div>
```

### Tag / Category Label
```html
<span class="sans" style="display:inline-block;font-size:10px;font-weight:600;letter-spacing:2px;color:{color};margin-bottom:16px;">{TAG TEXT}</span>
```
- Light slides: `BRAND_PRIMARY`
- Dark slides: `BRAND_LIGHT`
- Brand gradient slides: `rgba(255,255,255,0.6)`

### Logo Lockup (first and last slides)
- If logo icon: 40px circle (BRAND_PRIMARY bg) + icon centered + brand name beside
- If initials: 40px circle with first letter in white
- Brand name: 13px, weight 600, letter-spacing 0.5px

#### Logo no Slide 1 (hero) — Regra Obrigatória
- **`height` mínimo: 20px** — nunca abaixo disso
- **`opacity` mínimo: 0.88** — nunca semi-transparente
- A logo deve ser claramente legível no slide 1 em todos os carrosséis
- Em slides intermediários ou CTA, o tamanho pode variar conforme o layout

---

## Image Composition Rules

Never place:
- text
- logos
- cards
- overlays
- decorative shapes
- gradients
- UI elements

on top of important visual areas.

Important visual areas include:
- patient smiles
- teeth
- before/after comparison regions
- eyes
- facial expressions
- implant results
- transformation details

When using before/after images:
- keep the smile area completely visible
- position labels away from teeth
- use clean side labels or top/bottom labels
- avoid covering transformation results

When using patient or dentist photos:
- preserve facial visibility
- avoid placing text directly over faces
- use negative space when available

Composition should always prioritize:
1. visual clarity
2. transformation visibility
3. emotional impact
4. readability
5. premium aesthetics

---

## Layout Rules

- Content padding: `0 36px` standard
- Bottom-aligned slides with progress bar: `0 36px 52px` to clear the bar
- **Hero/CTA slides:** `justify-content: center`
- **Content-heavy slides:** `justify-content: flex-end`
- **Content must never overlap the progress bar** — use `padding-bottom: 52px`

---

## Instagram Frame (Preview Wrapper)

When displaying in chat, wrap in an Instagram-style frame:

- **Header:** Avatar (BRAND_PRIMARY circle + logo) + handle + subtitle
- **Viewport:** 4:5 aspect ratio, swipeable/draggable track with all slides
- **Dots:** Small dot indicators below the viewport
- **Actions:** Heart, comment, share, bookmark SVG icons
- **Caption:** Handle + short description + "2 HOURS AGO" timestamp

Include pointer-based swipe/drag interaction for preview. Slides are still standalone export-ready images.

**Important:** `.ig-frame` must be exactly **420px wide**. The carousel viewport is 420×525px. Do NOT change this width — export depends on it.

---

## Instagram Caption Generation

After generating the carousel preview, always generate an Instagram caption aligned with the carousel tone and mode.

Rules:
- Premium carousels:
  - more emotional
  - more sophisticated
  - storytelling-oriented
  - premium tone
  - subtle CTA

- Light carousels:
  - more direct
  - educational
  - easier to scan
  - simpler CTA

Caption structure:

Premium carousels:
1. Emotional or intriguing opening
2. Short storytelling or emotional development
3. Authority and reassurance
4. Soft premium CTA
5. Optional minimal hashtags

Light carousels:
1. Clear educational hook
2. Direct explanation
3. Simple reassurance
4. Soft CTA
5. Optional relevant hashtags

Caption length:
- short to medium
- easy to scan
- elegant spacing
- concise paragraphs

Avoid:
- exaggerated emojis
- aggressive selling
- clickbait
- generic marketing phrases

Prefer:
- elegant writing
- premium tone
- humanized communication
- concise paragraphs

Always provide:
- caption directly in chat
- and optionally save as `.txt` file if requested

---

## Review Flow

**Always follow this flow. Never skip to export without approval.**

1. Generate the HTML preview first — never jump directly to export
2. Show the preview and ask: **"Quais slides precisam de ajuste antes de exportar?"**
3. Fix only the mentioned slides — never regenerate the entire carousel unless the direction fundamentally changes
4. Only proceed to export when the user explicitly confirms approval (e.g., "pode exportar", "aprovado", "ok")

---

## Export Package Rules

When the user approves and says:
- "export"
- "exportar"
- "pode exportar"
- "approved"
- "aprovado"

create a complete post package.

Create one folder using this naming pattern:

exports/YYYY-MM-DD-mode-topic/

Where:
- YYYY-MM-DD = current date
- mode = premium or light
- topic = short lowercase slug of the carousel topic
Replace spaces with hyphens.
Remove accents and special characters.

Examples:
- exports/2026-05-13-premium-implantes-autoestima/
- exports/2026-05-13-light-facetas-duvidas/
- exports/2026-05-13-light-implante-doi/

Inside the folder, create:

carousel/
- slide_1.png
- slide_2.png
- slide_3.png
- ...

captions/
- legenda-carrossel.txt
- legenda-story.txt

Rules:
- carousel slides must be exported as 1080x1350 PNG
- carousel caption must be saved as legenda-carrossel.txt
- story caption must be saved as legenda-story.txt
- do not create a separate story PNG unless explicitly requested
- story caption must be written for reposting the carousel post in Instagram Stories
- story caption must be directly related to the carousel hook/topic
- story caption must create curiosity and encourage people to tap the reposted carousel

Story caption purpose:
- support the repost of the feed carousel
- increase post reach
- create curiosity
- drive people to open the carousel

Story caption style:
- short
- elegant
- premium
- natural
- no aggressive sales language
- minimal emojis

Examples:

For a light carousel about "Implante dói?":
"Essa é uma das dúvidas mais comuns aqui na clínica 👀"

For a premium carousel about smile transformation:
"Mais do que estética. É sobre voltar a sorrir com segurança."

For a carousel about loose dentures:
"Muita gente normaliza esse desconforto por anos."

If the user specifies a custom export folder, prioritize the user's path.

---

## Post-Export Cleanup Rules

After a successful export, always clean up the working directory automatically.

**Move into `exports/YYYY-MM-DD-mode-topic/source/`:**
- the generator script (e.g. `generate_light_v2.py`)
- the preview HTML (e.g. `carousel-light-v2.html`)
- the export script (e.g. `export_light_v2.py`)

**Delete from root:**
- any obsolete or superseded generator/export scripts that are no longer needed
- old preview HTMLs that have been replaced

**Keep in root:**
- only generator scripts and preview HTMLs for carousels not yet exported
- `gerador-carrossel-vallure.md`
- `assets/`
- `exports/`

Final structure after export:

```
exports/YYYY-MM-DD-mode-topic/
  carousel/     → slide_1.png … slide_N.png
  captions/     → legenda-carrossel.txt, legenda-story.txt
  source/       → generate_*.py, carousel-*.html, export_*.py
```

Root stays clean with only active (not yet exported) work files.

---

## Exporting Slides as Instagram-Ready PNGs

After the user approves the carousel preview, export a complete post package:
- carousel slides as PNG
- carousel caption as TXT
- story repost caption as TXT

### Critical Export Rules

1. **Use Python for HTML generation** — never use shell scripts with variable interpolation. Always use `Path.write_text()` or `open().write()`.

2. **Embed images as base64** — all user-uploaded images must be base64-encoded as `data:image/jpeg;base64,...` URIs. Check actual file format with the `file` command — a `.png` extension may contain a JPEG.

3. **Keep the 420px layout width** — use Playwright's `device_scale_factor` to scale up to 1080px output WITHOUT changing the layout viewport.

### Install Playwright (only if needed)

Before running the export script, check and install only if missing:

```bash
python3 -c "import playwright" 2>/dev/null || pip3 install playwright
python3 -c "from playwright.sync_api import sync_playwright; sync_playwright().__enter__().chromium" 2>/dev/null || python3 -m playwright install chromium
```

### Export Script

```python
import asyncio
from pathlib import Path
from playwright.async_api import async_playwright

INPUT_HTML = Path("/path/to/carousel.html")
OUTPUT_DIR = Path("/path/to/output/slides")
OUTPUT_DIR.mkdir(exist_ok=True)

TOTAL_SLIDES = 7  # Update to match your carousel

VIEW_W = 420
VIEW_H = 525
SCALE = 1080 / 420  # = 2.5714...

async def export_slides():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page(
            viewport={"width": VIEW_W, "height": VIEW_H},
            device_scale_factor=SCALE,
        )

        html_content = INPUT_HTML.read_text(encoding="utf-8")
        await page.set_content(html_content, wait_until="networkidle")
        await page.wait_for_timeout(3000)  # Wait for Google Fonts to load

        # Hide IG frame chrome, show only the slide viewport
        await page.evaluate("""() => {
            document.querySelectorAll('.ig-header,.ig-dots,.ig-actions,.ig-caption')
                .forEach(el => el.style.display='none');

            const frame = document.querySelector('.ig-frame');
            frame.style.cssText = 'width:420px;height:525px;max-width:none;border-radius:0;box-shadow:none;overflow:hidden;margin:0;';

            const viewport = document.querySelector('.carousel-viewport');
            viewport.style.cssText = 'width:420px;height:525px;aspect-ratio:unset;overflow:hidden;cursor:default;';

            document.body.style.cssText = 'padding:0;margin:0;display:block;overflow:hidden;';
        }""")
        await page.wait_for_timeout(500)

        for i in range(TOTAL_SLIDES):
            await page.evaluate("""(idx) => {
                const track = document.querySelector('.carousel-track');
                track.style.transition = 'none';
                track.style.transform = 'translateX(' + (-idx * 420) + 'px)';
            }""", i)
            await page.wait_for_timeout(400)

            await page.screenshot(
                path=str(OUTPUT_DIR / f"slide_{i+1}.png"),
                clip={"x": 0, "y": 0, "width": VIEW_W, "height": VIEW_H}
            )
            print(f"Exported slide {i+1}/{TOTAL_SLIDES}")

        await browser.close()

asyncio.run(export_slides())
```

### Why This Works

- **`device_scale_factor=2.5714`** renders at high DPI — a 420px element becomes 1080px in the output. Layout stays at 420px.
- **`clip`** captures only the carousel viewport, not browser chrome.
- **`wait_for_timeout(3000)`** gives Google Fonts time to load.
- **`track.style.transition = 'none'`** disables swipe animation so slides snap instantly.

### Common Export Mistakes to Avoid

| Mistake | What goes wrong | Fix |
|---------|----------------|-----|
| Setting viewport to 1080×1350 | Layout reflows — fonts tiny, spacing breaks | Keep viewport at 420×525, use `device_scale_factor` |
| Using shell scripts to generate HTML | `$` signs and backticks get interpolated | Always use Python for HTML generation |
| Not waiting for fonts | Headings render in fallback system fonts | `wait_for_timeout(3000)` after page load |
| Not hiding IG frame chrome | Export includes header, dots, caption | Hide `.ig-header,.ig-dots,.ig-actions,.ig-caption` |
| Changing `.ig-frame` width | Entire layout shifts | Always keep at exactly 420px |
| Leaving `BRAND_PRIMARY` as variable name in CSS | Color renders as invalid / invisible | Always interpolate actual hex values into HTML |

---

## Content Memory

Avoid repeating:
- hooks
- carousel structures
- CTA styles
- emotional angles
- opening phrases
- **topics** — never generate two consecutive carousels on the same subject

**Topic variety is mandatory.** Implants are the main focus of Vallure but NOT the only topic. If the last carousel was about implants, the next one must be about something different. Track mentally what was recently covered and rotate.

Prioritize variation across multiple generations.

---

## Vallure Editorial Topics

Implant-related (use in rotation, not consecutively):
- Fear of implants
- Immediate loading
- Loose dentures / denture vs fixed prosthesis
- Bone loss
- Myths about implants
- Who can get implants
- Full mouth rehabilitation

Non-implant topics (actively prioritize these for variety):
- Facetas de porcelana — smile aesthetics, veneer transformation
- Clareamento dental — myths, safety, results
- Bruxismo — teeth grinding, consequences, treatment
- Saúde gengival — gum health, prevention
- Manutenção da prótese — care and longevity
- Autoestima e odontologia — emotional connection with oral health
- Chewing difficulties — quality of life impact
- Post-surgery recovery — what to expect
- Smiling confidence — self-esteem beyond treatment type
- Modern dental technology — digital planning, precision

**Rule:** At least every other carousel must be on a non-implant topic. If in doubt, choose non-implant.

---

## Brand Presence Rules

Always maintain subtle but consistent Vallure branding across the carousel.

Rules:
- branding must feel natural and premium
- avoid excessive logo repetition
- logo usage should vary across generations

The logo should appear strategically:
- sometimes prominently
- sometimes subtly
- sometimes only as a small signature
- sometimes replaced by strong visual identity cues

Possible logo usage:
- subtle watermark
- icon-only logo
- elegant corner branding
- small signature
- minimal footer branding

Brand recognition should come mainly from:
- visual identity
- color palette
- typography
- premium composition
- photography style

NOT from repeating large logos on every slide.

---

## Premium Visual Quality Rules

The carousel must never look like plain text slides.

Each slide must include at least two visual elements besides text, such as:
- web photo (lifestyle, smile, dental aesthetic)
- gradient background
- Vallure logo
- elegant card
- gold line
- subtle frame
- soft shadow
- geometric shape
- texture overlay
- icon
- premium divider
- real clinic photo (only when contextually relevant)
- before/after image (only for transformation carousels)

Use real assets whenever relevant:
- fotos-clinica/
- fotos-dentista/
- before-after/
- logo files

Visual direction:
- premium dental clinic
- luxury editorial
- elegant beige/gold/black palette
- refined spacing
- sophisticated overlays
- high-end Instagram carousel

Avoid:
- plain empty slides
- text-only slides
- generic Canva-like layouts
- excessive white space without intention
- weak brand presence
- overly simple compositions

Every slide should feel intentionally designed, not just formatted.

---

## Layout Variation Rules

Avoid repetitive carousel compositions across generations.

The carousel should feel like a premium editorial system — not a fixed template.

Vary between generations:
- CTA layout
- logo positioning
- slide composition
- image placement
- text alignment
- card structure
- spacing rhythm
- use of gradients
- visual density
- background style
- framing elements
- photo cropping
- typography emphasis

Do NOT always:
- place the logo in the same position
- use the same CTA layout
- use the same gradient structure
- use the same final slide composition
- repeat identical spacing patterns
- repeat identical text hierarchy

Allowed variations:
- centered layouts
- asymmetric layouts
- editorial magazine style
- split-screen composition
- full-bleed photography
- floating cards
- minimal luxury style
- dark cinematic compositions
- light editorial compositions

Brand consistency must come from:
- color palette
- typography
- premium feeling
- photography style
- elegance

NOT from repeating the exact same layout.

---

## Design Principles

1. **Every slide is export-ready** — arrow and progress bar are part of the slide image
2. **Light/dark alternation** — creates visual rhythm across swipes
3. **Heading + body font pairing** — display font for impact, body for readability
4. **Brand-derived palette** — all colors stem from one primary, keeping everything cohesive
5. **Progressive disclosure** — progress bar fills and arrow guides forward
6. **Last slide is special** — no arrow, full progress bar, clear CTA
7. **Consistent components** — same tag style, list style, spacing across all slides
8. **Content padding clears UI** — body text never overlaps progress bar or arrow
9. **Hook-first copy** — Slide 1 exists to stop the scroll, not to introduce the brand
10. **Iterate fast** — show preview, fix specific slides, don't rebuild from scratch
