# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The whole app is one static page, `index.html`. It is a Three.js 3D model of 강릉포교당 관음사: 극락전 on a granite 기단, the front yard and parking lot, the 4-storey brick building to the north, and 금천관 with containers to the east. You can orbit around it or walk through it. All UI text is Korean.

The README still says "산사 가람 3D 모형 (일주문부터 대웅전 내부까지)", which describes an earlier idea. The current page has no 일주문 or 대웅전. This repo is also **not** the 가상사 (`garam_src.html`, `build_all.py`, db `garam/state`) project. Those build and publish steps do not apply here.

## Running / checking

There is no build, package manager, linter or test suite. Changes are uploaded straight to GitHub ("Add files via upload").

- Serve the folder and open the page: `python3 -m http.server 8000`, then go to `http://localhost:8000/`. Opening the file directly also works.
- Syntax check for the inline script:
  ```bash
  python3 -c "import re;s=open('index.html',encoding='utf-8').read();open('/tmp/m.js','w').write(max(re.findall(r'<script>(.*?)</script>',s,re.S),key=len))" && node --check /tmp/m.js
  ```
- Visual check: load the page with Playwright Chromium (WebGL needs swiftshader/`--use-gl=swiftshader` headless), click the view buttons in `#views`, and take screenshots.

## Dependencies

- Three.js **r128** (global `THREE`, aliased `T`) from cdnjs. The API is r128-era, so don't use newer-only APIs.
- Google Fonts: Noto Sans KR and Noto Serif KR. Canvas textures that draw text are listed in `redraw` (via `ctex(...,late=true)`). They are redrawn once `document.fonts.load` resolves. When you add new Hanja or Hangul to a canvas sign, also add those characters to the `document.fonts.load` strings at the bottom of the page, or they will render with a fallback font.

## Code layout (single IIFE inside `<script>`)

Sections are marked with `/* ---------- name ---------- */` comments, in this order:

1. **helpers**: seeded `rnd()` (deterministic, `seed=11`; adding calls changes later random layouts), `clamp`, `ctex(w,h,draw,rep,late)` canvas-texture factory.
2. **textures**: every texture is procedural, drawn on canvas (`plaqueTex`, `bannerTex`, `signTex`, `facadeTex`, …). There are no image assets.
3. **materials**: the shared material table `M`, plus `lanternMats`.
4. **sky, lights**: a gradient sky shader (`skyU`), `hemi`, and a shadow-casting `sun`.
5. **ground / roof / 극락전 / 기단·계단 / 앞마당 / 주차 / 벽돌 건물·금천관**: geometry built with `box()`, `flat()`, `makeRoof()`, `buddha()`, `tree()`, `car()`.
6. **heights**: `plats` and `ramps` feed `heightAt(x,z)`, which gives the eye height in walk mode.
7. **info & labels**: the `INFO` text table (key → `{t,d}`), plus `LBL`, which places 3D-anchored HTML label buttons.
8. **camera state**: one state object `S` (`mode` 'orbit'|'walk', orbit `theta/phi/radius/target`, walk `pos/yaw/pitch/eye`, `anim`). `VIEWS` are the preset buttons; each `go()` calls `orbitTo` or `walkTo`.
9. **time of day**: `applyTime(dusk)` switches sky, fog, lights and emissives. It also reaches `window.__lantern` and `window.__inner`, which are set while the hall is built.
10. **input / loop**: pointer, wheel and keyboard input and the touch `#pad`. `frame()` moves the camera, picks the info panel by zone in walk mode, and projects the labels.

### Coordinates and collisions

- World units are meters, with y up. The hall is a `THREE.Group` at `(HX,PH,HZ)` = `(-13.5, 3.0, 0)` with `rotation.y = π/2`. Its children use **local** coordinates.
- Hall collision boxes go into `hcols` in local coordinates. They are converted to world space as `x=HX+lz, z=HZ-lx` and pushed into `colliders`. Everything else calls `addCol(x0,x1,z0,z1)` in world coordinates.
- `free(x,z)` limits walking to `x∈(-24,29), z∈(-19,24)`. Anything outside that range can't be walked to.

### Adding things

- A new place or view needs an `INFO` entry, then a `VIEWS` item and/or a `LBL` item. If it should take over the info panel while walking, add it to the zone test in `frame()`.
- Anything that should glow at dusk must be updated in `applyTime`.
- The script is minified-style dense one-liners. Use `/* */` comments, not `//`, because a `//` inside a one-line statement comments out the rest of the line.

## Content principle

The `INFO` descriptions describe the real site. Keep them factual. Don't present invented scenes or legends as fact; if you must add one, label it as such.
