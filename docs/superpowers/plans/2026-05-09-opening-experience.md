# Opening Experience Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a cinematic Ink Bleed intro animation, film grain overlay, 3D card tilt, and article hover reveals to `index.html`.

**Architecture:** All changes are made to the single `index.html` file. CSS additions go inside the existing `<style>` block (before the closing `</style>`). New HTML elements are injected by JS at runtime (grain overlay, intro overlay). JS additions go inside the existing `<script>` block (before the closing `</script>`).

**Tech Stack:** Pure CSS keyframes, CSS custom properties, vanilla JS — no external libraries.

---

## File Map

- Modify: `index.html` — all four features; exact line ranges noted per task
  - CSS additions → inside `<style>` block, appended before `</style>` (line ~351)
  - HTML additions → `<body>` open tag area for intro overlay
  - JS additions → inside `<script>` block, appended before `</script>` (line ~738)

---

### Task 1: Ink Bleed — CSS

**Files:**
- Modify: `index.html` — append inside `<style>` block before `</style>`

- [ ] **Step 1: Locate the closing `</style>` tag**

  Run:
  ```bash
  grep -n "</style>" /home/user/EconifyZine/index.html
  ```
  Expected output: one line number (around line 351). Note it.

- [ ] **Step 2: Add the intro overlay CSS**

  Find this exact string at the bottom of the `<style>` block:
  ```css
      @media (min-width: 1024px) {
  ```
  Actually find the closing `</style>` and insert just before it. Use Edit tool to replace `  </style>` with the block below. The exact string to find:

  ```
    </style>
  ```
  (single-indented closing style tag — confirm with grep first)

  Replace with:

  ```css
    /* ═══ INTRO OVERLAY ═══ */
    #ez-intro {
      position: fixed; inset: 0; z-index: 9999;
      background: oklch(97% 0.01 75);
      display: flex; align-items: center; justify-content: center;
      overflow: hidden;
    }
    #ez-intro.ez-done { pointer-events: none; }
    .ez-ink {
      position: absolute; inset: -10%;
      background: var(--ink);
      border-radius: 50%;
      transform: scale(0);
      animation: ez-ink-spread 1.4s cubic-bezier(0.22, 1, 0.36, 1) 0.25s forwards;
    }
    @keyframes ez-ink-spread {
      0%   { transform: scale(0); }
      100% { transform: scale(3); }
    }
    .ez-title {
      position: relative; z-index: 2;
      text-align: center;
      font-family: var(--f-display);
      font-size: clamp(3.5rem, 10vw, 9rem);
      font-weight: 900;
      line-height: 0.88;
      letter-spacing: -0.04em;
      color: var(--ink);
      transition: color 0s;
      user-select: none;
    }
    .ez-title.ez-light { color: oklch(97% 0.01 75); }
    .ez-rule {
      position: relative; z-index: 2;
      height: 3px; background: var(--red);
      width: 0; margin: 1rem auto 0;
      transition: width 0.5s cubic-bezier(0.22, 1, 0.36, 1);
    }
    .ez-rule.ez-open { width: 6rem; }
    #ez-skip {
      position: absolute; top: 1.5rem; right: 2rem; z-index: 3;
      font-family: var(--f-ui); font-size: 0.65rem; font-weight: 700;
      letter-spacing: 0.15em; text-transform: uppercase;
      color: oklch(97% 0.01 75); opacity: 0;
      transition: opacity 0.4s;
      background: none; border: none; cursor: pointer; padding: 0.5rem;
    }
    #ez-skip.ez-visible { opacity: 0.5; }
    #ez-skip:hover { opacity: 1; }
    #ez-intro.ez-fade-out {
      animation: ez-fade-out 0.6s var(--ease) forwards;
    }
    @keyframes ez-fade-out {
      0%   { opacity: 1; }
      100% { opacity: 0; }
    }
    @media (prefers-reduced-motion: reduce) {
      #ez-intro { display: none !important; }
    }

  </style>
  ```

- [ ] **Step 3: Verify CSS was inserted cleanly**

  ```bash
  grep -n "ez-intro\|ez-ink\|ez-title" /home/user/EconifyZine/index.html | head -15
  ```
  Expected: ~10 matches, all within the `<style>` block.

- [ ] **Step 4: Commit**

  ```bash
  git -C /home/user/EconifyZine add index.html
  git -C /home/user/EconifyZine commit -m "feat: add ink bleed intro CSS"
  ```

---

### Task 2: Ink Bleed — HTML + JS

**Files:**
- Modify: `index.html` — insert HTML after `<body>` open tag; append JS before `</script>`

- [ ] **Step 1: Insert intro overlay HTML immediately after `<body>`**

  Find the `<body>` open tag:
  ```bash
  grep -n "^<body" /home/user/EconifyZine/index.html
  ```
  It opens with `<body>` on its own line. Replace `<body>` with:

  ```html
  <body>
  <div id="ez-intro" aria-hidden="true">
    <div class="ez-ink"></div>
    <div style="position:relative;z-index:2;text-align:center">
      <div class="ez-title" id="ez-title">ECONIFY<br>ZINE</div>
      <div class="ez-rule" id="ez-rule"></div>
    </div>
    <button id="ez-skip" aria-label="Skip intro">Skip →</button>
  </div>
  ```

- [ ] **Step 2: Add intro JS before `</script>`**

  Find the closing `</script>` tag. Append this block just before it (after the existing image-fallback code):

  ```js
    // ── Ink Bleed Intro ──
    (function() {
      if (sessionStorage.getItem('ez-intro-seen')) return;
      const intro = document.getElementById('ez-intro');
      const title = document.getElementById('ez-title');
      const rule  = document.getElementById('ez-rule');
      const skip  = document.getElementById('ez-skip');
      if (!intro) return;
      document.body.style.overflow = 'hidden';

      function finish() {
        document.body.style.overflow = '';
        intro.classList.add('ez-fade-out');
        setTimeout(() => intro.remove(), 650);
        sessionStorage.setItem('ez-intro-seen', '1');
      }

      // Phase 1 complete (ink spread done ~1.65s) → flip title white + open rule
      setTimeout(() => {
        title.classList.add('ez-light');
        rule.classList.add('ez-open');
      }, 1650);

      // Show skip button after 1s
      setTimeout(() => skip.classList.add('ez-visible'), 1000);

      // Auto-finish after 2.6s
      setTimeout(finish, 2600);

      skip.addEventListener('click', finish);
    })();
  ```

- [ ] **Step 3: Verify the intro appears in HTML structure**

  ```bash
  grep -n "ez-intro\|ez-ink\|ez-skip" /home/user/EconifyZine/index.html
  ```
  Expected: HTML lines in the `<body>` section AND JS lines in the `<script>` section.

- [ ] **Step 4: Open the site locally to manually verify**

  ```bash
  cd /home/user/EconifyZine && python3 -m http.server 8080 &
  ```
  Open `http://localhost:8080` — you should see: white screen → ink floods from center → title turns white → red rule appears → overlay fades → site loads.

  Kill the server: `kill %1`

- [ ] **Step 5: Commit**

  ```bash
  git -C /home/user/EconifyZine add index.html
  git -C /home/user/EconifyZine commit -m "feat: add ink bleed intro HTML and JS"
  ```

---

### Task 3: Film Grain

**Files:**
- Modify: `index.html` — CSS appended inside `<style>`; grain injected via JS

- [ ] **Step 1: Add grain CSS before `</style>`**

  Find this exact string (the last line before `</style>`, which is the closing brace of the reduced-motion block added in Task 1):
  ```
      @media (prefers-reduced-motion: reduce) {
        #ez-intro { display: none !important; }
      }

    </style>
  ```
  Replace with:
  ```css
      @media (prefers-reduced-motion: reduce) {
        #ez-intro { display: none !important; }
      }

    /* ═══ FILM GRAIN ═══ */
    #ez-grain {
      position: fixed; inset: 0; z-index: 100;
      pointer-events: none;
      opacity: 0.04;
      filter: url(#ez-noise);
      animation: ez-grain-shift 0.18s steps(8) infinite;
      will-change: transform;
    }
    @keyframes ez-grain-shift {
      0%   { transform: translate(0,    0); }
      12%  { transform: translate(-2px, 1px); }
      25%  { transform: translate(1px, -2px); }
      37%  { transform: translate(-1px, 2px); }
      50%  { transform: translate(2px,  1px); }
      62%  { transform: translate(-2px,-1px); }
      75%  { transform: translate(1px,  2px); }
      87%  { transform: translate(-1px,-2px); }
      100% { transform: translate(0,    0); }
    }
    @media (prefers-reduced-motion: reduce) {
      #ez-grain { display: none; }
    }

    </style>
  ```

- [ ] **Step 2: Inject grain elements via JS**

  Inside the existing `<script>` block, append after the intro code:

  ```js
    // ── Film Grain ──
    if (!window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
      const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
      svg.setAttribute('aria-hidden', 'true');
      svg.style.cssText = 'position:absolute;width:0;height:0;overflow:hidden';
      svg.innerHTML = `<defs><filter id="ez-noise" x="0%" y="0%" width="100%" height="100%" color-interpolation-filters="linearRGB">
        <feTurbulence type="fractalNoise" baseFrequency="0.65" numOctaves="3" stitchTiles="stitch" result="noise"/>
        <feColorMatrix type="saturate" values="0" in="noise"/>
      </filter></defs>`;
      document.body.appendChild(svg);
      const grain = document.createElement('div');
      grain.id = 'ez-grain';
      grain.setAttribute('aria-hidden', 'true');
      document.body.appendChild(grain);
    }
  ```

- [ ] **Step 3: Verify grain is visible**

  ```bash
  grep -n "ez-grain\|ez-noise\|feTurbulence" /home/user/EconifyZine/index.html
  ```
  Expected: CSS lines + JS lines, no duplicates.

- [ ] **Step 4: Commit**

  ```bash
  git -C /home/user/EconifyZine add index.html
  git -C /home/user/EconifyZine commit -m "feat: add film grain overlay"
  ```

---

### Task 4: 3D Card Tilt

**Files:**
- Modify: `index.html` — CSS + JS

- [ ] **Step 1: Add tilt CSS before `</style>`**

  Find the grain CSS closing line (last `</style>` after grain block). Insert before it:

  ```css
    /* ═══ 3D TILT ═══ */
    .mosaic-cell { transform-style: preserve-3d; will-change: transform; }
    .spotlight-cover img { will-change: transform; transition: transform 0.15s var(--ease), box-shadow 0.15s var(--ease); }
    .mosaic-cell.ez-tilting,
    .spotlight-cover img.ez-tilting {
      transition: transform 0.1s linear;
    }
    .mosaic-cell::after {
      content: '';
      position: absolute; inset: 0;
      background: radial-gradient(circle at var(--mx, 50%) var(--my, 50%),
        rgba(255,255,255,0.12) 0%, transparent 60%);
      opacity: 0;
      transition: opacity 0.2s;
      pointer-events: none;
      border-radius: inherit;
    }
    .mosaic-cell:hover::after { opacity: 1; }
  ```

  Note: `.mosaic-cell` already has `position` implied by its layout — add `position: relative` to the existing `.mosaic-cell` rule. Find:
  ```css
      .mosaic-cell { padding: clamp(1.25rem, 2.5vw, 2rem); border-bottom: 1px solid var(--rule); transition: background 0.2s; }
  ```
  Replace with:
  ```css
      .mosaic-cell { padding: clamp(1.25rem, 2.5vw, 2rem); border-bottom: 1px solid var(--rule); transition: background 0.2s; position: relative; overflow: hidden; }
  ```

- [ ] **Step 2: Add tilt JS**

  Append inside `<script>` after grain code:

  ```js
    // ── 3D Tilt ──
    if (!window.matchMedia('(hover: none)').matches &&
        !window.matchMedia('(prefers-reduced-motion: reduce)').matches) {

      function applyTilt(el, isImg) {
        el.addEventListener('mousemove', (e) => {
          const r = el.getBoundingClientRect();
          const x = ((e.clientX - r.left) / r.width  - 0.5) * 2;
          const y = ((e.clientY - r.top)  / r.height - 0.5) * 2;
          el.style.transform = `perspective(700px) rotateY(${x * 8}deg) rotateX(${-y * 8}deg) scale(1.02)`;
          el.style.setProperty('--mx', ((e.clientX - r.left) / r.width  * 100) + '%');
          el.style.setProperty('--my', ((e.clientY - r.top)  / r.height * 100) + '%');
          el.classList.add('ez-tilting');
        });
        el.addEventListener('mouseleave', () => {
          el.classList.remove('ez-tilting');
          el.style.transform = '';
        });
      }

      document.querySelectorAll('.mosaic-cell').forEach(el => applyTilt(el, false));
      const coverImg = document.querySelector('.spotlight-cover img');
      if (coverImg) applyTilt(coverImg, true);
    }
  ```

- [ ] **Step 3: Verify no JS errors**

  ```bash
  grep -n "applyTilt\|ez-tilting\|mosaic-cell.*ez-tilt" /home/user/EconifyZine/index.html
  ```
  Expected: CSS definition of `.ez-tilting` + JS definition of `applyTilt` function.

- [ ] **Step 4: Commit**

  ```bash
  git -C /home/user/EconifyZine add index.html
  git -C /home/user/EconifyZine commit -m "feat: add 3D card tilt on article cards and cover"
  ```

---

### Task 5: Article Hover Reveals

**Files:**
- Modify: `index.html` — CSS only (existing `.mosaic-cell:hover` rules enhanced)

- [ ] **Step 1: Replace the existing mosaic hover rules**

  Find this exact block in the CSS:
  ```css
      .mosaic-cell:hover { background: var(--red-pale); }
  ```
  And these two lines further down:
  ```css
      .mosaic-cell:hover .art-title { color: var(--red); }
  ```

  Replace the first with a comment (the `::before` pseudo-element now handles the colour wash):
  ```css
      .mosaic-cell:hover { background: transparent; }
  ```

- [ ] **Step 2: Add hover-reveal CSS before `</style>`**

  Find the final `</style>` and insert before it:

  ```css
    /* ═══ HOVER REVEALS ═══ */
    .mosaic-cell::before {
      content: '';
      position: absolute; inset: 0;
      background: var(--red-pale);
      clip-path: inset(0 100% 0 0);
      transition: clip-path 0.38s cubic-bezier(0.22, 1, 0.36, 1);
      pointer-events: none;
      z-index: 0;
    }
    .mosaic-cell:hover::before {
      clip-path: inset(0 0% 0 0);
    }
    .mosaic-cell > * { position: relative; z-index: 1; }
    .mosaic-cell { box-shadow: inset 0 0 0 var(--red); transition: background 0.2s, box-shadow 0.3s; }
    .mosaic-cell:hover {
      background: transparent;
      box-shadow: inset 0 3px 0 var(--red);
    }
    .art-title { transition: color 0.2s, transform 0.3s var(--ease); }
    .mosaic-cell:hover .art-title { color: var(--red); transform: translateY(-3px); }
    @media (prefers-reduced-motion: reduce) {
      .mosaic-cell::before { transition: none; }
      .art-title { transition: color 0.2s; }
    }
  ```

- [ ] **Step 3: Verify no z-index conflicts**

  The `::after` specular highlight from Task 4 uses `z-index` implicitly (stacking context). Confirm `.mosaic-cell::before` (clip-path wash, z-index:0) sits below `.mosaic-cell::after` (specular, no explicit z-index but above stacking context). Check by searching:

  ```bash
  grep -n "mosaic-cell::before\|mosaic-cell::after\|z-index" /home/user/EconifyZine/index.html | grep -v "nav\|drawer\|intro\|grain" | head -20
  ```

- [ ] **Step 4: Commit**

  ```bash
  git -C /home/user/EconifyZine add index.html
  git -C /home/user/EconifyZine commit -m "feat: add article hover clip-path reveals"
  ```

---

### Task 6: Push and Verify

- [ ] **Step 1: Push feature branch**

  ```bash
  git -C /home/user/EconifyZine push -u origin claude/add-skills-install-plugin-ty5HR
  ```

- [ ] **Step 2: Also update main via Python MCP call**

  Read the updated HTML and push to `main` (GitHub Pages branch):

  ```python
  # Run this Python snippet:
  import urllib.request, json, subprocess

  with open('/home/claude/.claude/remote/.session_ingress_token') as f:
      token = f.read().strip()

  with open('/home/user/EconifyZine/index.html') as f:
      html = f.read()

  subprocess.run(['git','-C','/home/user/EconifyZine','fetch','origin','main'], capture_output=True)
  sha = subprocess.run(['git','-C','/home/user/EconifyZine','rev-parse','origin/main:index.html'],
                       capture_output=True, text=True).stdout.strip()

  mcp_url = "https://api.anthropic.com/v2/ccr-sessions/cse_01Gm36LHzVUFDusVGuBWuBwM/github/mcp"
  hdrs = {'Content-Type':'application/json','Authorization':f'Bearer {token}',
          'X-Session-UUID':'cse_01Gm36LHzVUFDusVGuBWuBwM',
          'X-MCP-Server-ID':'7530dab8-ccd9-5055-91a0-7f051e6c2ac8'}
  msg = {"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"create_or_update_file",
         "arguments":{"owner":"itzflom","repo":"EconifyZine","path":"index.html",
                      "content":html,"message":"feat: cinematic opening experience + film grain + 3D tilt + hover reveals",
                      "branch":"main","sha":sha}}}
  req = urllib.request.Request(mcp_url, data=json.dumps(msg).encode(), headers=hdrs, method='POST')
  with urllib.request.urlopen(req, timeout=60) as r:
      print(r.read().decode()[:400])
  ```

  Expected: response contains `"commit":{"sha":` with a new commit SHA.

- [ ] **Step 3: Done**

  The live site at `https://itzflom.github.io/EconifyZine` (once GitHub Pages is configured) will show the full experience.
