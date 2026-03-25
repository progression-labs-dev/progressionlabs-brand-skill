---
name: progression-labs-brand
description: Use when creating any document, presentation, deck, report, PDF, web page, email, diagram, or digital asset for Progression Labs. Applies the exact brand colors, typography, layout, pixel mosaic hero, and design language from progressionlabs.com.
---

# Progression Labs Brand System

Apply this brand system to ALL Progression Labs digital assets: presentations, documents, PDFs, decks, diagrams, emails, web pages, and any visual deliverable.

## Brand Identity

**Company:** Progression Labs
**Positioning:** AI consultancy helping companies integrate AI solutions
**Aesthetic:** Light-first, technical-minimalist with generative art influences

## Color Hierarchy

**Primary:** Blue `#0000FF` + White `#fafafa`
**Secondary accent:** Salmon `#FFA07A`

### Full Brand Colors (5-color system)
| Name | Hex | Role |
|------|-----|------|
| **Blue** | `#0000FF` | Primary brand color |
| **Salmon** | `#FFA07A` | Secondary accent, CTAs |
| **Orchid** | `#BA55D3` | Tertiary accent |
| **Green** | `#B9E979` | Brand green |
| **Turquoise** | `#40E0D0` | Cool accent |

### UI Colors (Light Theme — default)
| Role | Value |
|------|-------|
| Background | `#fafafa` |
| Text primary | `#1a1a1a` |
| Text secondary | `#555` / `#666` |
| Text tertiary | `rgba(0,0,0,0.35)` |
| Border | `rgba(0,0,0,0.06)` |
| Card surface | `rgba(0,0,0,0.012)` |
| Section labels | `rgba(0,0,255,0.45)` |

## Typography

| Role | Stack |
|------|-------|
| **Primary** | `Inter, -apple-system, sans-serif` |
| **Monospace** | `'SF Mono', 'Fira Code', monospace` |

| Element | Size | Weight | Notes |
|---------|------|--------|-------|
| Hero headline | 40px | 300 | Light, elegant |
| Section heading | 32px | 300 | Letter-spacing -0.01em |
| Card title | 14px | 600 | |
| Body | 13.5px | 400 | Line-height 1.75 |
| Labels | 10px | 600 | UPPERCASE, letter-spacing 0.12em, monospace |
| CTA text | 10px | 600 | UPPERCASE, letter-spacing 0.1em |

**Rules:** Headlines weight 300. Bold keywords weight 600. All labels/CTAs UPPERCASE with wide tracking. Section labels in blue at 45% opacity, monospace.

## Logo

**CRITICAL: The Progression Labs logo is a rounded connector/pill shape — NOT a geometric "P" letter. Always use the actual PNG files from the repo. NEVER generate an SVG approximation or use a font glyph.**

- **Light backgrounds:** `logo-black.png` (138×110 PNG, from `public/logo-black.png` on deck-tool branch)
- **Dark/gradient backgrounds:** `logo-white.png` (138×110 PNG, from `public/logo-white.png` on deck-tool branch)
- **Usage:** `<img src="logo-white.png" alt="Progression Labs" style="width:80px; height:auto;">`
- **The old SVG path (`M0,0 H8 A12,12...`) is WRONG** — it renders a geometric P, not the actual logo
- **Wordmark:** "PROGRESSION LABS" uppercase, 10px, letter-spacing 0.15em, weight 500
- **Logo + wordmark pairing:** Logo image at 80px wide, wordmark centered below with 8px gap

## Title Slide / Cover Page (Partial Pixel Mosaic Hero)

Every presentation, deck, or document with a cover page MUST use the pixel mosaic + smooth gradient hero as the title slide background. This is the brand's signature visual, ported from the deck tool (`PixelGradientCanvas.tsx`).

### Key visual: Pixelation is PARTIAL, not full-bleed
**The pixel mosaic occupies only the top-left quadrant of the hero.** The rest is a smooth (non-pixelated) gradient. This creates the signature "frosted glass dissolving into color" effect.

```
┌─────────────────────────────┐
│ ██ ██ ██ ██ ░░ ░░           │  ← Pixelated blocks (top-left)
│ ██ ██ ██ ░░ ░░              │     with ASCII shimmer overlay
│ ██ ██ ░░ ░░                 │
│ ██ ░░ ░░     ╔═══════╗     │
│ ░░ ░░        ║ LOGO  ║     │  ← Smooth gradient (center + bottom-right)
│ ░░           ╚═══════╝     │     deep blue/purple, fading to near-black
│              Title text     │
│                             │
│                    ▓▓▓▓▓▓▓▓▓│  ← Dark gradient at bottom
└─────────────────────────────┘
```

### Two layers composited together
1. **Smooth gradient layer (full canvas):** The base gradient covers the entire canvas — Orchid/Salmon blending to deep Blue/Purple, darkening to near-black at bottom-right
2. **Pixelated overlay (top-left only):** The same gradient colors but snapped to a block grid, with a wavy alpha mask that fades the blocks out toward center/bottom-right

### Gradient colors
- **Color A (warm):** Orchid `[186, 85, 211]` / Salmon `[255, 160, 122]` — dominates top-left
- **Color B (cool):** Blue `[0, 0, 255]` — dominates bottom-right
- **Value noise:** 3 octaves of 2D noise create organic swirling between colors
- **Luminance ramp:** near-black → deep (peak×0.06) → mid (peak×0.35) → peak → wash (mix toward white at 20%)
- **Bottom darkening:** Gradient goes to near-black at bottom edge

### Pixelation mask (wavy fade from top-left)
The pixelated blocks fade out from the top-left corner using the same wavy smoothstep as the pixel corners:

```javascript
// Block size for hero: 45px (larger than interior corner blocks)
const blockSize = 45;

// For each pixel, compute distance from top-left origin
const nx = x / canvasWidth;   // 0 at left, 1 at right
const ny = y / canvasHeight;  // 0 at top, 1 at bottom
const dist = Math.sqrt(nx*nx + ny*ny); // 0 at top-left, ~1.41 at bottom-right

// Wavy fade — pixelation dissolves organically
const wave = Math.sin(nx*3.5+1.2)*0.18 + Math.sin(nx*8+3.7)*0.1
           + Math.cos(ny*5.5+0.5)*0.12 + Math.sin(nx*12)*0.05;
const pixelMask = smoothstep(0.8, 0.0, dist + wave * 0.15);

// Where pixelMask > 0: show block-snapped color
// Where pixelMask = 0: show smooth gradient
// Blend between them: mix(smoothColor, pixelColor, pixelMask)
```

### ASCII shimmer band (on pixelated area only)
ASCII characters from `'0123456789@#$%&*+=?<>{}[]/\|LABS'` appear within the pixelated region:
- Band position: diagonal from top-left, `diag = (uv.x + 1.0 - uv.y) * 0.5`, frozen at ~0.45
- Gaussian mask: `exp(-dist^2 * 80)`
- **Only rendered where pixelMask > 0** — shimmer fades with the blocks
- Density: 40%, opacity: 50% × brightness × shimmerMask × pixelMask
- White text, Inter 500 weight
- Film grain: 6% opacity, baked into alpha (no overlay boundary)

### Implementation
Use `<canvas>` (not SVG). Set `canvas.width = window.innerWidth; canvas.height = window.innerHeight`. CSS: `position: absolute; top:0; left:0; width:100%; height:100%`.

**IMPORTANT:** The parent slide must have `position: absolute` (not `relative`) with `inset: 0` — otherwise the slide collapses to content height.

### Title slide layout
- Partial pixel mosaic canvas (z-index 1) — pixelation top-left, smooth gradient elsewhere
- **Logo only** — white `logo-white.png` centered, 100px wide, `drop-shadow(0 0 20px rgba(255,255,255,0.2))` (z-index 3)
- **No title text, no subtitle, no wordmark** on the title slide — just the logo mark on the gradient
- Footer: monospace meta in `rgba(255,255,255,0.25)` (optional)

## Pixel Corner Accent (Interior Pages)

For non-title slides/pages, use pixel corner accents as brand decorations. This is the **exact algorithm from the deck tool** (`DeckCanvas.tsx` corner-accent mode on the `deck-tool` branch of `websiteplab`).

### Parameters (updated — richer concentration with gradual staircase falloff)
| Param | Default | Description |
|-------|---------|-------------|
| `blockSize` | `25` | Pixel block size in px |
| `fadeRadius` | `0.15` | 0–1, fraction of diagonal — keep SMALL for compact corner accent (NOT spread across slide) |
| `fadeScatter` | `0` | 0–1, set to 0 for clean edge — scatter creates messy stray blocks |
| `accentColorA` | `[80, 120, 230]` | Brand blue (RGB 0-255) |
| `accentColorB` | `[80, 120, 230]` | **Same as A — single color, NOT two-tone** |
| `maxAlpha` | `0.88` | Rich, concentrated blocks near corner origin — NOT pale/translucent |
| `asciiDensity` | `0.4` | Fraction of blocks that get an ASCII character overlay |
| `asciiAlpha` | `blockAlpha * 0.6` | Character opacity tracks block opacity |
| `cornerOrigin` | alternating | `top-right` on odd slides, `bottom-left` on even |

**CRITICAL: Use ONE color.** Both `accentColorA` and `accentColorB` must be the same blue. Two different colors (e.g. blue + purple) creates a messy multi-tone look. The variation comes from alpha/opacity, not hue.

### Logo watermark in corner accent
Draw the white PL logo (`pl-logo-white.png`) in the densest area of each corner accent:
- Size: 28px wide, maintain aspect ratio (110/138)
- Position: 1.5 block widths inward from the corner origin
- Opacity: 1.0 (full white, no glow or drop-shadow — just clean and present)
- Drawn AFTER all blocks so it sits on top

### Implementation (exact code from DeckCanvas.tsx)
Use `<canvas>`, NOT SVG. The algorithm uses `smoothstep(1 - t)` fade with scatter for organic edges.

```javascript
// Updated corner-accent with richer concentration, staircase falloff, and logo watermark
function drawCornerAccent(canvasId, cornerOrigin, opts) {
  const canvas = document.getElementById(canvasId);
  if (!canvas) return;
  const dpr = window.devicePixelRatio || 1;
  const width = window.innerWidth, height = window.innerHeight;
  canvas.width = width * dpr; canvas.height = height * dpr;
  canvas.style.width = width + 'px'; canvas.style.height = height + 'px';
  const ctx = canvas.getContext('2d'); ctx.scale(dpr, dpr);
  const blockSize = opts.blockSize || 25;
  const fadeRadius = opts.fadeRadius || 0.09;
  const maxAlpha = opts.maxAlpha || 0.88;
  const [aR, aG, aB] = opts.colorA || [80, 120, 230];
  const [bR, bG, bB] = opts.colorB || [80, 120, 230];

  let ox, oy;
  switch (cornerOrigin) {
    case 'top-left':     ox = 0;     oy = 0;      break;
    case 'top-right':    ox = width; oy = 0;      break;
    case 'bottom-left':  ox = 0;     oy = height; break;
    case 'bottom-right': ox = width; oy = height; break;
  }

  const diagonal = Math.sqrt(width * width + height * height);
  const maxDist = diagonal * fadeRadius;
  const cols = Math.ceil(width / blockSize);
  const rows = Math.ceil(height / blockSize);

  const smoothstep = (t) => {
    const c = Math.max(0, Math.min(1, t));
    return c * c * (3 - 2 * c);
  };
  const seed = (x, y) => {
    const h = Math.sin(x * 127.1 + y * 311.7) * 43758.5453123;
    return h - Math.floor(h);
  };

  for (let row = 0; row < rows; row++) {
    for (let col = 0; col < cols; col++) {
      const cx = (col + 0.5) * blockSize;
      const cy = (row + 0.5) * blockSize;
      const dist = Math.sqrt((cx - ox) ** 2 + (cy - oy) ** 2);
      const t = dist / maxDist;
      if (t >= 1.2) continue;

      // Gradual staircase: steeper near origin, long tail outward
      let blockAlpha;
      if (t <= 1.0) {
        blockAlpha = smoothstep(1 - t) * maxAlpha;
      } else {
        // Extended tail — sparse blocks beyond main radius
        blockAlpha = (1.2 - t) * 0.15 * maxAlpha;
      }
      // Per-block variation for organic feel
      blockAlpha *= (0.85 + seed(col + 700, row + 700) * 0.15);
      if (blockAlpha < 0.02) continue;

      const colorMix = seed(col + 300, row + 300);
      const r = Math.round(aR + (bR - aR) * colorMix);
      const g = Math.round(aG + (bG - aG) * colorMix);
      const b = Math.round(aB + (bB - aB) * colorMix);

      ctx.fillStyle = `rgba(${r}, ${g}, ${b}, ${blockAlpha})`;
      ctx.fillRect(col * blockSize, row * blockSize, blockSize, blockSize);

      // ASCII shimmer overlay — characters on 40% of blocks
      if (blockAlpha > 0.06 && seed(col, row) < 0.4) {
        const CHARS = '0123456789@#$%&*+=?<>{}[]/\\|LABS';
        const ch = CHARS[Math.floor(seed(col + 100, row + 100) * CHARS.length)];
        ctx.fillStyle = `rgba(${r}, ${g}, ${b}, ${blockAlpha * 0.6})`;
        ctx.font = '500 ' + (blockSize * 0.5) + 'px "Inter", sans-serif';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText(ch, cx, cy);
      }
    }
  }

  // Logo watermark in densest area
  const logoImg = new Image();
  logoImg.onload = function() {
    const logoSize = 36;
    const pad = blockSize * 1.5;
    let lx, ly;
    switch (cornerOrigin) {
      case 'top-right':    lx = width - pad - logoSize/2; ly = pad + logoSize/2; break;
      case 'bottom-left':  lx = pad + logoSize/2; ly = height - pad - logoSize/2; break;
      case 'top-left':     lx = pad + logoSize/2; ly = pad + logoSize/2; break;
      case 'bottom-right': lx = width - pad - logoSize/2; ly = height - pad - logoSize/2; break;
    }
    ctx.globalAlpha = 0.7;
    ctx.filter = 'drop-shadow(0 0 6px rgba(255,255,255,0.3))';
    ctx.drawImage(logoImg, lx - logoSize/2, ly - logoSize/2, logoSize, logoSize * (110/138));
    ctx.globalAlpha = 1.0;
    ctx.filter = 'none';
  };
  logoImg.src = 'pl-logo-white.png';
}
```

### Positioning (CRITICAL)
The canvas must NOT set `slideEl.style.position = 'relative'` — this breaks the slide's `position: absolute; inset: 0`. The canvas uses `position: absolute; inset: 0; pointer-events: none` so it doesn't block clicks. Use `window.innerWidth/Height` for sizing if the slide is hidden (`display: none` → `offsetWidth` returns 0).

### Icons
Use clean inline SVGs with `stroke: currentColor; stroke-width: 1.5`. NEVER use emojis.

## Slide Content Layout (CRITICAL — no top-left bunching)

**ALL slide content MUST be centered both horizontally and vertically.** Content bunched into a corner is a serious layout bug.

### Centering Rules (MANDATORY for every slide)
```css
.slide {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;       /* horizontal center */
  justify-content: center;   /* vertical center */
  text-align: center;        /* text centering */
  padding: 80px 72px;        /* generous breathing room */
}
```

### Exceptions (left-align ONLY when explicitly needed)
- **Contact/CTA slides** with name + email: left-align the contact block, but center it within the slide using a centered wrapper
- **Tables/code blocks**: left-align content inside a centered container
- **Multi-column layouts**: center the grid, left-align within columns

### Footer pinning
Footer always pins to bottom using `margin-top: auto` on the footer element. The flex centering handles everything above it.

## Design Language

### Core Principles
1. **Light-first** — `#fafafa` backgrounds, clean and minimal
2. **Blue + white primary** — blue for labels, accents, brackets
3. **Salmon as warm accent** — bullet markers, accent gradients
4. **Sharp geometry** — `border-radius: 0` on buttons/CTAs
5. **Dashed dividers** — `1px dashed rgba(0,0,0,0.08)` with centered "+" marks
6. **Ultra-thin headings** — weight 300
7. **Uppercase micro-text** — labels, CTAs in 9-11px uppercase
8. **Generous white space** — never cramped

### Components
- **Cards:** `rgba(0,0,0,0.012)` bg, `1px solid rgba(0,0,0,0.06)` border, blue corner brackets (10px L-shapes)
- **Buttons (light):** Dark bg `#1a1a1a`, white text, sharp corners
- **Buttons (dark):** White bg, dark text, sharp corners
- **Tables:** Monospace blue uppercase headers, dashed row borders
- **Blockquotes:** weight 300, salmon `#FFA07A` left border (2px)
- **Metrics:** Monospace, 28px, weight 700, blue `#0000FF`
- **Status badges:** Dead=`#dc2626`, Decaying=`#FFA07A`, Strong=`#16a34a`, New=`#0000FF`

## Applying the Brand by Asset Type

### Presentations / Decks (HTML slides, PowerPoint, Keynote)
- **Title slide:** Partial pixel mosaic hero (see above) + white logo centered — **logo only, no text**
- **Content slides:** `#fafafa` bg, blue section labels, dashed "+" dividers
- **Pixel corners:** On ALL content slides, alternating tr/bl, with PL logo watermark in densest area
- **Navigation:** Small dots (4px, 8% opacity), slide counter bottom-right
- **Tables/charts:** Use 5 brand colors in order
- **Arrow keys / spacebar** for navigation
- **GSAP animations:** Required for all HTML decks (see GSAP section below)

### GSAP Slide Animations (HTML presentation decks ONLY)

**Scope:** GSAP animations apply ONLY to interactive HTML slide decks. Do NOT add animations to PDFs, static documents, reports, emails, or any non-interactive deliverable.

Load GSAP from CDN: `<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>`

**Animation philosophy:** Elements animate in visual hierarchy order — a "build" effect like Keynote. Use `gsap.timeline()` with `power3.out` easing, 0.6s default duration.

**Animation lock:** Set `isAnimating = true` during transitions, block navigation until complete. Prevents double-skip.

**Per-element choreography:**
| Element | Animation | Timing |
|---------|-----------|--------|
| `.label` | `opacity: 0→1, y: 12→0` | First, 0.4s |
| `.divider` | `scaleX: 0→1` | +0.08s, 0.5s |
| `.heading` | `opacity: 0→1, y: 20→0` | +0.12s, 0.6s |
| `.body-text` | `opacity: 0→1, y: 14→0` | +0.15s, 0.5s |
| `.plus-divider` | `opacity: 0→1` | +0.08s, 0.4s |
| `.card` / `.future-card` | `opacity: 0→1, y: 30→0` | stagger: 0.1s |
| `.highlight-number` | `opacity: 0→1, scale: 0.8→1` | stagger: 0.1s |
| `.stat-item` | `opacity: 0→1, y: 16→0` | stagger: 0.12s |
| `.versus-col` (left) | `opacity: 0→1, x: -30→0` | simultaneous |
| `.versus-col` (right) | `opacity: 0→1, x: 30→0` | +0.1s |
| `.brand-quote` | `opacity: 0→1, x: -20→0` | after body, 0.6s |
| `.cta-btn` | `opacity: 0→1, y: 12→0` | last, 0.5s |
| `.logo-wrap` (hero) | `opacity: 0→1, scale: 0.9→1` | 0.8s |
| `.contact-heading` | `opacity: 0→1, y: 20→0` | 0.7s |

**CSS prep (hide before animation):**
```css
.slide .content .label, .slide .content .heading,
.slide .content .body-text, .slide .content .brand-quote,
.slide .content .card, .slide .content .future-card,
.slide .content .stat-item, .slide .content .versus-col,
.slide .content .cta-btn, .slide .content .highlight-number,
.slide .content .logo-wrap, .slide .content .plus-divider,
.slide .content .contact-heading, .slide .content .contact-name,
.slide .content .contact-role, .slide .content .contact-email { opacity: 0; }
.slide .content .divider { transform: scaleX(0); }
```

**Trigger:** Call `animateSlideIn(slides[currentSlide])` inside `goToSlide()` after setting `.active`. On page load, trigger with `setTimeout(() => animateSlideIn(slides[0]), 300)`.

### Documents / PDFs / Reports (job descriptions, proposals, briefs)
- **Keep it simple** — no fancy cards, corner-bracket boxes, or SVG icons. Those are for presentations only.
- **Cover:** Partial pixel mosaic hero OR light bg with pixel corner
- **Content padding:** 64px top, 72px horizontal, 80px bottom
- **Header:** Logo + wordmark left, monospace meta right
- **Section labels:** Monospace, 10px, uppercase, blue at 45%
- **Headings:** Inter, weight 300, bold keywords weight 600
- **Body:** Inter 13.5px, line-height 1.75, color `#444`, max-width ~520px
- **Lists:** Simple bullet points in muted grey (`#999`) — NO salmon/orange dashes, keep it professional
- **Quotes:** Salmon left border, weight 300 text
- **Dividers:** Dashed "+" dividers between sections
- **NO big metric numbers** — large blue stats are for decks only, not documents
- **CTA:** One dark button at the bottom if needed
- **Pixel corner:** Single corner accent (top-right), subtle brand touch
- **Footer:** Monospace 8px, `#aaa`, pinned to bottom, dashed top border
- **Prefer 3 pages over cramming into 2**
- **NO cards with corner brackets, NO SVG icons, NO grids** — these belong in decks, not documents

### Diagrams / Flowcharts
- **Background:** `#fafafa`
- **Node borders:** `1px solid rgba(0,0,0,0.06)`, blue corner brackets
- **Node text:** Inter, dark
- **Connection lines:** Blue `#0000FF` at 30% opacity
- **Labels:** Monospace uppercase, blue at 45%
- **Accent nodes:** Salmon `#FFA07A` fill at 10% opacity

### Emails
- **Light:** `#fafafa` bg, `#1a1a1a` text
- **Logo at top**, blue-to-salmon accent line below
- **CTA:** Dark button, sharp corners
- **Footer:** `#aaa` monospace, 9px

### Social Media / Graphics
- **Dark backgrounds** with pixel mosaic accents
- **Headlines in Inter weight 300**
- **Category labels in uppercase monospace**
- **Brand gradient** as accent/overlay
