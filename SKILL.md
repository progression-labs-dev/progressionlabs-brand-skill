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

## Title Slide / Cover Page (Animated Hero Gradient)

Every presentation, deck, or document with a cover page MUST use the animated hero gradient as the title slide background. This is the brand's signature visual, ported from `HeroGradientGL.tsx` on the progressionlabs.com experiment site.

### Architecture: 4-layer canvas stack
1. **WebGL gradient** — animated smooth gradient with mouse-driven + shimmer pixel reveal
2. **ASCII overlay** (Canvas 2D) — characters appear where pixel blocks are visible
3. **Film grain** (Canvas 2D) — `mix-blend-mode: overlay`, 6% opacity
4. **Content** — logo on top (z-index 3)

### Gradient: animated 5-color cycling
All 5 brand colors cycle through 9 dual-color states over 45 seconds:
- Orchid → Blue+Salmon → Green → Orchid+Turquoise → Salmon → Blue+Turquoise → Blue → Orchid+Green → Turquoise → loop
- Transitions use cubic smoothstep easing
- 3-octave value noise creates organic color swirling between the two active colors
- Luminance ramp (bottom→top): near-black `0.004` → deep `peak*0.06` → mid `peak*0.35` → peak → wash `mix(peak, white, 0.5)` → white

### Pixel reveal: diagonal shimmer only (no mouse interaction for decks)
Pixelated blocks (45px) appear via a sweeping diagonal shimmer band:
- **Diagonal shimmer:** Band sweeps top-left → bottom-right at `fract(time * 0.25)`, Gaussian `exp(-dist^2 * 120) * 0.6`
- Pixel blocks blend with shimmer mask: `mix(smoothColor, pixelColor, shimmerMask)`

**NO mouse interaction in decks.** The website hero uses mouse-driven pixel reveal, but for decks remove the `uMouse`/`uMouseActive` uniforms entirely. Decks are presentation tools — mouse hover effects are distracting. Shimmer-only is cleaner.

**Note:** The full HeroGradientGL.tsx source (in `websiteplab` repo, `deck-tool` branch) includes mouse uniforms — strip them when porting to deck HTML.

### ASCII overlay (separate Canvas 2D layer)
- Characters: `'0123456789@#$%&*+=?<>{}[]/\|LABS'`
- Grid: 45px blocks matching the WebGL grid, 40% fill chance (random per init)
- Each character positioned at block center, converted from WebGL UV to canvas coords: `py = height - (vUvY * height)`
- Visibility: same `max(mouseMask, shimmerMask)` mask as the gradient — characters only appear where pixel blocks show
- Alpha: `mask * cell.brightness` where brightness is random 0.5–1.0
- Font: `500 14px "Inter", sans-serif`, white, centered

### WebGL shader (copy-paste — ported from HeroGradientGL.tsx)

**Vertex shader:**
```glsl
attribute vec2 position;
varying vec2 vUv;
void main() {
  vUv = position * 0.5 + 0.5;
  gl_Position = vec4(position, 0.0, 1.0);
}
```

**Fragment shader:**
```glsl
precision highp float;
uniform float uTime;
uniform vec2 uResolution;
varying vec2 vUv;

float ssmooth(float t) { return t * t * (3.0 - 2.0 * t); }
float hash(vec2 p) { return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453123); }

float vnoise(vec2 p) {
  vec2 i = floor(p); vec2 f = fract(p);
  f = f * f * (3.0 - 2.0 * f);
  float a = hash(i); float b = hash(i + vec2(1.0, 0.0));
  float c = hash(i + vec2(0.0, 1.0)); float d = hash(i + vec2(1.0, 1.0));
  return mix(mix(a, b, f.x), mix(c, d, f.x), f.y);
}

vec3 computeGradient(vec2 uv, float time, vec3 peakA, vec3 peakB) {
  float gp = uv.y;
  float n1 = vnoise(uv * 1.8 + vec2(time * 0.10, time * 0.07));
  float n2 = vnoise(uv * 3.5 + vec2(-time * 0.08, time * 0.12));
  float n3 = vnoise(uv * 6.0 + vec2(time * 0.15, -time * 0.06));
  float swirl = n1 * 0.5 + n2 * 0.3 + n3 * 0.2;
  float verticalBias = smoothstep(0.05, 0.95, gp);
  float colorMix = clamp(verticalBias + (swirl - 0.5) * 1.0, 0.0, 1.0);
  vec3 peak = mix(peakA, peakB, colorMix);
  float wave = (vnoise(uv * 2.0 + vec2(time * 0.06, -time * 0.04)) - 0.5) * 0.06;
  float protection = smoothstep(0.0, 0.25, gp);
  gp = clamp(gp + wave * protection, 0.0, 1.0);
  vec3 deep = peak * 0.06; vec3 mid = peak * 0.35;
  vec3 wash = mix(peak, vec3(1.0), 0.5);
  float t1 = smoothstep(0.00, 0.10, gp); float t2 = smoothstep(0.06, 0.24, gp);
  float t3 = smoothstep(0.15, 0.55, gp); float t4 = smoothstep(0.45, 0.85, gp);
  float t5 = smoothstep(0.75, 1.00, gp);
  vec3 color = mix(vec3(0.004), deep, t1);
  color = mix(color, mid, t2); color = mix(color, peak, t3);
  color = mix(color, wash, t4); color = mix(color, vec3(1.0), t5);
  return color;
}

vec3 getGradientColor(vec2 uv) {
  vec3 cO = vec3(0.729, 0.333, 0.827); // Orchid
  vec3 cS = vec3(1.000, 0.627, 0.478); // Salmon
  vec3 cG = vec3(0.725, 0.914, 0.475); // Green
  vec3 cT = vec3(0.251, 0.878, 0.816); // Turquoise
  vec3 cB = vec3(0.000, 0.000, 1.000); // Blue
  float progress = mod(uTime, 45.0) / 45.0;
  float seg = progress * 9.0;
  int idx = int(floor(seg));
  float t = ssmooth(seg - floor(seg));
  vec3 fA, fB, tA, tB;
  if (idx == 0)      { fA = cO; fB = cO; tA = cB; tB = cS; }
  else if (idx == 1) { fA = cB; fB = cS; tA = cG; tB = cG; }
  else if (idx == 2) { fA = cG; fB = cG; tA = cO; tB = cT; }
  else if (idx == 3) { fA = cO; fB = cT; tA = cS; tB = cS; }
  else if (idx == 4) { fA = cS; fB = cS; tA = cB; tB = cT; }
  else if (idx == 5) { fA = cB; fB = cT; tA = cB; tB = cB; }
  else if (idx == 6) { fA = cB; fB = cB; tA = cO; tB = cG; }
  else if (idx == 7) { fA = cO; fB = cG; tA = cT; tB = cT; }
  else               { fA = cT; fB = cT; tA = cO; tB = cO; }
  vec3 peakA = mix(fA, tA, t); vec3 peakB = mix(fB, tB, t);
  return computeGradient(uv, uTime, peakA, peakB);
}

void main() {
  vec3 smoothColor = getGradientColor(vUv);
  float blockPx = 45.0;
  vec2 grid = uResolution / blockPx;
  vec2 cellId = floor(vUv * grid);
  vec2 pixelUv = cellId / grid;
  float colOffset = hash(vec2(cellId.x, 0.0)) * 0.035;
  pixelUv.y += colOffset;
  vec3 pixelColor = getGradientColor(pixelUv);

  // Diagonal shimmer — sweeping band (no mouse interaction in decks)
  float diag = (vUv.x + 1.0 - vUv.y) * 0.5;
  float shimmerPos = fract(uTime * 0.25);
  float shimmerDist = abs(diag - shimmerPos);
  shimmerDist = min(shimmerDist, 1.0 - shimmerDist);
  float shimmerMask = exp(-shimmerDist * shimmerDist * 120.0) * 0.6;

  vec3 finalColor = mix(smoothColor, pixelColor, shimmerMask);
  gl_FragColor = vec4(finalColor, 1.0);
}
```

### JS setup (raw WebGL, no Three.js dependency)
```javascript
// Uniforms: uTime (animated elapsed), uResolution, uMouse (damped 0-1 UV), uMouseActive (0-1 lerp)
// Fullscreen quad: [-1,-1, 1,-1, -1,1, -1,1, 1,-1, 1,1]
// Mouse tracking: damped lerp at 0.08 per frame, mouseActive lerp at 0.1
// Render loop: requestAnimationFrame, update uTime = elapsed seconds
// Resize: canvas.width = offsetWidth * min(devicePixelRatio, 2)
```

### HTML structure
```html
<div class="hero-layers">
  <canvas id="heroGL"></canvas>      <!-- WebGL gradient -->
  <canvas id="heroAscii"></canvas>   <!-- ASCII overlay (pointer-events: none) -->
  <canvas id="heroGrain"></canvas>   <!-- Film grain (mix-blend-mode: overlay, opacity: 0.06) -->
</div>
```

### Title slide layout
- Hero gradient canvas stack (z-index 1)
- **Logo only** — white `logo-white.png` centered, 64px wide, `drop-shadow(0 0 20px rgba(255,255,255,0.2))` (z-index 3)
- **No title text, no subtitle, no wordmark** on the title slide — just the logo mark on the gradient
- Footer: monospace meta in `rgba(255,255,255,0.25)` (optional)
- Parent slide: `position: absolute; inset: 0` — NOT relative

## Animated Pixel Corner Accent (Interior Pages)

For non-title slides/pages, use **animated** pixel corner accents. These use Canvas 2D with `requestAnimationFrame` — same clean uniform block look as the original static version, but with animated color cycling through the 5 brand colors and a subtle diagonal shimmer.

### Key design rules
- **ONE color at a time** — all blocks are the same color at any given moment, cycling through brand colors over 45 seconds. NEVER mix two colors simultaneously (e.g. red + blue blocks = cheap)
- **Uniform opacity** — blocks get their alpha from distance falloff only. No per-block brightness variation beyond the staircase smoothstep
- **Subtle shimmer** — diagonal band passes through, gently brightening existing blocks by 15% max. It is MULTIPLICATIVE (`blockAlpha * (1 + shimmer)`) — it must NOT reveal blocks that should be invisible
- **20px blocks** — finer grain than the hero's 45px
- **Alternating corners** — `top-right` on odd slides, `bottom-left` on even

### Parameters
| Param | Default | Description |
|-------|---------|-------------|
| `blockSize` | `20` | Pixel block size in px |
| `fadeRadius` | `0.15` | 0–1, fraction of diagonal |
| `maxAlpha` | `0.88` | Peak alpha near corner origin |
| `asciiDensity` | `0.4` | Fraction of blocks with ASCII character |
| `asciiAlpha` | `blockAlpha * 0.6` | Character opacity |
| `shimmerSpeed` | `0.12` | Diagonal sweep speed (half the hero's 0.25) |
| `shimmerBoost` | `0.15` | Max multiplicative brightness lift |
| `cornerOrigin` | alternating | `top-right` on odd slides, `bottom-left` on even |

### Logo watermark
- `logo-white.png`, 26px wide, aspect ratio 110/138
- Position: 2 block widths inward from corner origin
- Full opacity, no effects, drawn AFTER all blocks each frame

### Color cycling (same 9-state 45s cycle as hero)
All 5 brand colors rotate: Orchid → Blue+Salmon → Green → Orchid+Turquoise → Salmon → Blue+Turquoise → Blue → Orchid+Green → Turquoise → loop. The two slot colors are averaged into ONE unified color per frame.

### Implementation (Canvas 2D, animated — copy-paste)

Initialize one per content slide at page load. Each runs its own `requestAnimationFrame` loop.

```javascript
function initAnimatedCorner(canvasId, cornerOrigin) {
  const canvas = document.getElementById(canvasId);
  if (!canvas) return;
  const ctx = canvas.getContext('2d');
  const BLOCK_SIZE = 20;
  const CHARS = '0123456789@#$%&*+=?<>{}[]/\\|LABS';
  const logoImg = new Image(); logoImg.src = 'logo-white.png';
  const startTime = performance.now() / 1000;

  let oxScreen, oyScreen;
  switch (cornerOrigin) {
    case 'top-right': oxScreen = 1.0; oyScreen = 0.0; break;
    case 'bottom-left': oxScreen = 0.0; oyScreen = 1.0; break;
    case 'top-left': oxScreen = 0.0; oyScreen = 0.0; break;
    case 'bottom-right': oxScreen = 1.0; oyScreen = 1.0; break;
  }

  const seed = (x, y) => { const h = Math.sin(x * 127.1 + y * 311.7) * 43758.5453123; return h - Math.floor(h); };
  const smoothstep = (e0, e1, x) => { const t = Math.max(0, Math.min(1, (x - e0) / (e1 - e0))); return t * t * (3 - 2 * t); };

  const brandColors = [
    [186, 85, 211], [255, 160, 122], [185, 233, 121], [64, 224, 208], [0, 0, 255],
  ];
  const cycle = [[0,0,4,1],[4,1,2,2],[2,2,0,3],[0,3,1,1],[1,1,4,3],[4,3,4,4],[4,4,0,2],[0,2,3,3],[3,3,0,0]];

  // Returns ONE unified color (both cycle slots averaged)
  function getCycleColor(time) {
    const progress = ((time % 45) / 45);
    const seg = progress * 9;
    const idx = Math.floor(seg);
    const t = smoothstep(0, 1, seg - idx);
    const s = cycle[Math.min(idx, 8)];
    const fA = brandColors[s[0]], fB = brandColors[s[1]], tA = brandColors[s[2]], tB = brandColors[s[3]];
    const pA = fA.map((v, i) => v + (tA[i] - v) * t);
    const pB = fB.map((v, i) => v + (tB[i] - v) * t);
    return pA.map((v, i) => Math.round((v + pB[i]) / 2));
  }

  let width = 0, height = 0;
  function resize() {
    const dpr = window.devicePixelRatio || 1;
    width = canvas.parentElement.clientWidth; height = canvas.parentElement.clientHeight;
    canvas.width = width * dpr; canvas.height = height * dpr;
    canvas.style.width = width + 'px'; canvas.style.height = height + 'px';
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  }
  resize();
  window.addEventListener('resize', resize);

  function render() {
    const time = performance.now() / 1000 - startTime;
    const [r, g, b] = getCycleColor(time);
    ctx.clearRect(0, 0, width, height);

    const cols = Math.ceil(width / BLOCK_SIZE), rows = Math.ceil(height / BLOCK_SIZE);
    const fadeRadius = 0.15, maxAlpha = 0.88;
    const oxPx = oxScreen * width, oyPx = oyScreen * height;
    const diagonal = Math.sqrt(width * width + height * height);
    const maxDist = diagonal * fadeRadius;
    const shimmerPos = (time * 0.12) % 1.0;

    for (let row = 0; row < rows; row++) {
      for (let col = 0; col < cols; col++) {
        const cx = (col + 0.5) * BLOCK_SIZE, cy = (row + 0.5) * BLOCK_SIZE;
        const dist = Math.sqrt((cx - oxPx) ** 2 + (cy - oyPx) ** 2);
        const t = dist / maxDist;
        if (t >= 1.2) continue;

        let blockAlpha;
        if (t <= 1.0) { blockAlpha = smoothstep(1, 0, t) * maxAlpha; }
        else { blockAlpha = (1.2 - t) * 0.15 * maxAlpha; }
        blockAlpha *= (0.85 + seed(col + 700, row + 700) * 0.15);
        if (blockAlpha < 0.02) continue;

        // Shimmer — multiplicative only, never reveals invisible blocks
        const nx = cx / width, ny = cy / height;
        const diag = (nx + 1.0 - ny) * 0.5;
        let shimmerDist = Math.abs(diag - shimmerPos);
        shimmerDist = Math.min(shimmerDist, 1.0 - shimmerDist);
        const shimmerMask = Math.exp(-shimmerDist * shimmerDist * 200) * 0.15;
        const finalAlpha = blockAlpha * (1.0 + shimmerMask);

        ctx.fillStyle = `rgba(${r},${g},${b},${finalAlpha})`;
        ctx.fillRect(col * BLOCK_SIZE, row * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);

        if (blockAlpha > 0.06 && seed(col, row) < 0.4) {
          const ch = CHARS[Math.floor(seed(col + 100, row + 100) * CHARS.length)];
          ctx.fillStyle = `rgba(${r},${g},${b},${blockAlpha * 0.6})`;
          ctx.font = '500 ' + (BLOCK_SIZE * 0.45) + 'px "Inter", sans-serif';
          ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
          ctx.fillText(ch, cx, cy);
        }
      }
    }

    if (logoImg.complete && logoImg.naturalWidth > 0) {
      const logoSize = 26, pad = BLOCK_SIZE * 2;
      let lx, ly;
      switch (cornerOrigin) {
        case 'top-right': lx = width - pad; ly = pad; break;
        case 'bottom-left': lx = pad; ly = height - pad; break;
        case 'top-left': lx = pad; ly = pad; break;
        case 'bottom-right': lx = width - pad; ly = height - pad; break;
      }
      ctx.drawImage(logoImg, lx - logoSize/2, ly - logoSize/2, logoSize, logoSize * (110/138));
    }
    requestAnimationFrame(render);
  }
  requestAnimationFrame(render);
}

// Initialize at page load:
// for (let i = 1; i < totalSlides; i++) {
//   initAnimatedCorner('corner-' + i, i % 2 === 1 ? 'top-right' : 'bottom-left');
// }
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
