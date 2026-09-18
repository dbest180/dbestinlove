# DEEPSEEK_REQUIREMENTS.md

**What this is.** The agent's workspace cannot install its own packages — `$HOME` is read-only and `sudo` is blocked by the container. Anything outside the session workspace has to be installed by a human. This is that list.

**How to use it.** Run §2 against the CLI environment. §1 is already solved — please don't redo it. §3 is optional. §4 covers harness-level asks.

Last verified by running each command in the live session — not assumed.

---

## 0. Why installs are awkward here

| Constraint | Observed |
|---|---|
| Writable | the session workspace `/home/dbest1/Plus One Project/dbestinlove`, and `/tmp` |
| Read-only | `/home/dbest1` — so `~/.gem`, `~/.cache`, `~/.npm` are all unusable |
| `sudo` | unavailable: *"The 'no new privileges' flag is set, which prevents sudo from running as root"* |
| Network egress | fine — `rubygems.org`, `registry.npmjs.org`, `github.com` all return HTTP 200 |

The workaround is redirecting each tool's cache and prefix into the workspace with env vars. It works, but it's per-tool and fragile. §3.1 has the permanent fix if you'd rather I stop doing it.

---

## 1. ✅ Already working — no action needed

### Ruby + Jekyll — local build verified

`ruby 3.3.8` and `gem 3.6.7` were already present. The Jekyll toolchain installs **into the workspace**, because `gem env home` (`/var/lib/gems/3.3.0`) isn't writable and I have no sudo:

```bash
cd "/home/dbest1/Plus One Project/dbestinlove"
export GEM_HOME="$PWD/.tools/gems"      GEM_PATH="$PWD/.tools/gems"
export GEM_SPEC_CACHE="$PWD/.tools/spec-cache"   # else RubyGems writes ~/.cache/gem and dies
export XDG_CACHE_HOME="$PWD/.tools/cache"
export HOME="$PWD/.tools/home"                    # last resort for stray cache writes
export PATH="$PWD/.tools/gems/bin:$PATH"
gem install jekyll webrick jekyll-seo-tag --no-document   # done: 4.4.1 / 1.9.2 / 2.9.0
```

Then build. **`JEKYLL_NO_BUNDLER_REQUIRE=1` is required** — without it Jekyll sees the `Gemfile`, hands off to Bundler 4, and dies on default gems (`base64`, `csv`, `json`, `logger`, `bigdecimal`) that aren't in the local GEM_HOME:

```bash
export JEKYLL_NO_BUNDLER_REQUIRE=1
jekyll build --source . --destination _site     # verified working, 0.155s
```

**Result:** I can now render the site locally and read the real `_site/index.html` before pushing. This already paid for itself — it let me confirm the `og:image` fix locally instead of deploying blind.

### Headless Chromium + Playwright — verified working

Every system library Chromium needs was already present (`libnss3`, `libnspr4`, `libatk-1.0`, `libatk-bridge-2.0`, `libcups`, `libdrm`, `libxkbcommon`, `libXcomposite`, `libXdamage`, `libXrandr`, `libgbm`, `libpango-1.0`, `libcairo`, `libasound`), so the browser self-served into the workspace — nothing for you to do:

```bash
cd "/home/dbest1/Plus One Project/dbestinlove"
export npm_config_cache="$PWD/.tools/npm-cache"
export PLAYWRIGHT_BROWSERS_PATH="$PWD/.tools/browsers"
npx --yes playwright@latest install chromium     # done: Chromium headless shell 153
```

Driving the live page works, including DOM measurement and screenshots:

```bash
cd .tools/pw && export PLAYWRIGHT_BROWSERS_PATH="$PWD/../browsers"
node shot.js "https://dbest180.github.io/dbestinlove/" "$PWD/../shots"
```

**Result:** I can measure the deployed page exactly — element boxes, computed styles, horizontal overflow — at any viewport width, without pushing. This settled the long-open hero-crop question (see `HANDOFF.md`).

### Everything else already present

`git` 2.47.3 · `gh` 2.46.0, authenticated as `dbest180` with `repo` + `workflow` scope · `node` 22.23.2 / `npm` 10.9.8 · `python3` 3.13.5 · `curl` 8.14.1 · `wget` 1.25.0

`gh` is what lets me push and watch Actions runs (`gh run list`, `gh run view`). It's working — please don't disrupt its auth.

---

## 2. Needed

### 2.1 An image-capable model — **the one real remaining gap**

**The browser problem is solved** (see §1): Chromium downloads, launches, measures the live page, and captures screenshots.

**But I cannot look at what it captures.** `read_image` refuses:

> model "deepseek-v4-flash" does not declare image input; switch to an image-capable model to read images

So I have a camera and no eyes. Screenshots land on disk and only a human can read them.

For the hero-crop question this project was carrying, I worked around it by measuring the DOM instead — `getBoundingClientRect`, resolved `object-fit`, overflow on both axes at five widths. That's arguably stricter than eyeballing, and it produced a definitive answer. But it does not generalise: measurement cannot tell anyone *"the type looks wrong,"* *"that photo is unflattering,"* or *"the composition is off."*

**Options, best first:**

1. **Run agent sessions on an image-capable model.** Then `read_image` works and the visual loop closes entirely. This is the single highest-leverage change on this page — worth more than every package below combined.
2. **Expose a vision-capable subagent** I can hand a PNG to for a second opinion.
3. **Leave it.** Workable. I'll keep writing screenshots to `.tools/shots/` for you to open, and will say plainly when a judgement is beyond measurement rather than guessing.

**Right now, if you want to see what I see:**

```
.tools/shots/live-400.png    .tools/shots/live-900.png    .tools/shots/live-1920.png
.tools/shots/live-768.png    .tools/shots/live-1400.png
```

### 2.2 ImageMagick — needed for any image work

**Why:** `HANDOFF.md` §2 documents the image pipeline as `convert -strip -interlace Plane -resize <W>x -quality 78`, but `convert` and `magick` are both missing. I cannot generate the `hero-1400` / `hero-700` / `og.jpg` assets, recompress anything, or verify image dimensions beyond hand-parsing JPEG headers.

```bash
apt-get install -y imagemagick
```

---

## 3. Optional — quality of life, not blockers

### 3.1 Make the workspace-root writable (or just a cache dir)

Right now every tool needs its cache redirected by hand into the repo, which is why `.tools/` exists and why §1's install command is five `export`s long. Either of these removes that entirely:

- Allow writes to `~/.cache`, `~/.local`, and `~/.gem`, **or**
- Leave the file sandbox as-is but grant a writable cache directory

This is the single change that would let me install most future things myself. Worth more than any individual package below.

### 3.2 Small utilities

| Package | Why | Command |
|---|---|---|
| `jq` | standalone JSON wrangling (`gh` has one embedded, so this is only for my own pipelines) | `apt-get install -y jq` |
| `python3-pil` | image dimension/inspection instead of hand-parsing JPEG headers | `apt-get install -y python3-pil` |
| `python3-yaml` | parse `_config.yml` / `links.yml` properly instead of by regex | `apt-get install -y python3-yaml` |
| `yamllint` | validate YAML before it breaks a build | `apt-get install -y yamllint` |
| `optipng` `jpegoptim` `cwebp` | optimize the hero/OG images | `apt-get install -y optipng jpegoptim webp` |
| `librsvg2-bin` | rasterize the SVG favicon for previews | `apt-get install -y librsvg2-bin` |

### 3.3 Proper Bundler setup

`bundle` only exists inside `.tools/` (Bundler 4.0.21), and Jekyll has to bypass it. If you'd rather the documented `bundle exec jekyll serve` from `README.md` actually worked, install bundler and jekyll system-wide:

```bash
gem install bundler jekyll webrick jekyll-seo-tag
```

This is cosmetic — the build works today via §1. It would just make the repo's own instructions true.

### 3.4 CI link checking

`html-proofer` (a Ruby gem) would catch dead links automatically. Worth adding only once there are more than the handful of links on the page today.

---

## 4. Harness / plugin asks

You mentioned you can turn plugins on. For this project the useful ones are:

1. **An image-capable model — the big one.** Browser automation now works (§1), so the harness can render the page and capture it. What's missing is the ability to *interpret* the result (§2.1). If you can run sessions on a vision model, or expose a vision subagent, the entire visual feedback loop closes.
2. **A wider file-sandbox mode** (see §3.1). Currently `workspace-write`, which is why installs accumulate inside the repo.
3. **Network egress is already sufficient** — nothing to enable there.

**On GitHub Pages plugins specifically** — if that's what you meant: `jekyll-seo-tag` and `jekyll-sitemap` are both on GitHub Pages' supported list, so there is nothing to "turn on" in repo settings. They're enabled purely by listing them under `plugins:` in `_config.yml`. The Pages source should stay **GitHub Actions** (the workflow already builds with `actions/jekyll-build-pages@v1`).

---

## 5. Quick checklist

```bash
# 1. images  — needed the moment anyone touches hero/OG artwork
apt-get install -y imagemagick

# 2. quality of life
apt-get install -y jq python3-pil python3-yaml yamllint optipng jpegoptim webp librsvg2-bin

# 3. optionally make the repo's own documented workflow true
gem install bundler jekyll webrick jekyll-seo-tag
```

**Not on this list, because it isn't an install:** running sessions on an image-capable model (§2.1). That's a config change, and it's worth more than everything above combined.

Nothing here is required to keep shipping — the site builds, deploys, and is verified end-to-end today. The browser and Jekyll are already solved. What remains is the ability to *see*, and ImageMagick if artwork gets touched.
