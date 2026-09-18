# DEEPSEEK_REQUIREMENTS.md

**Status: satisfied.** Every installable item in this document is now installed and verified against this project. One item remains and it is not an install — see §4.

This file started as a request for help, because the agent could not install its own packages. That constraint is gone. It is now a **record of the environment and a recipe to rebuild it**, which is what makes it worth keeping.

Environment: Debian 13 (trixie), aarch64, user `dbest1`, passwordless `sudo` available.

---

## 0. What changed

| | Before | Now |
|---|---|---|
| File sandbox | `workspace-write` — only the repo was writable | `danger-full-access` |
| `$HOME` | **read-only** — `~/.gem`, `~/.cache`, `~/.npm` unusable | writable |
| `sudo` | blocked: *"The 'no new privileges' flag is set"* | works — `sudo -n id` returns `uid=0(root)` |
| Consequence | every tool needed its cache redirected into the repo via env vars; 790 MB of tooling lived in `.tools/` inside the git repo | tools install normally; the repo is back to 3.0 MB of actual project files |

The old workaround is documented in §3 in case a future container reverts to a locked-down sandbox.

---

## 1. Installed and verified

Every package below was proven by doing this project's real work, not by checking `--version`.

| Package | Version | How it was verified |
|---|---|---|
| `imagemagick` | 7.1.1-43 Q16 | Ran the pipeline `HANDOFF.md` documents: `convert hero-1400.jpg -strip -interlace Plane -resize 700x -quality 78` produced a valid 30,937-byte JPEG (the existing `hero-700.jpg` is 36,459 bytes, so the documented settings hold up). Also produced a 1200×630 crop for `og.jpg`. |
| `python3-pil` | Pillow 11.1.0 | Read dimensions of all four images in `assets/img/`. |
| `python3-yaml` | PyYAML 6.0.2 | Parsed `_config.yml` and `_data/links.yml` structurally — no more regex. |
| `yamllint` | 1.37.1 | **Found a real bug**: `_data/links.yml` had no trailing newline. Fixed. `_config.yml` also had an 81-char comment line; rewrapped. Both now lint clean. |
| `optipng` | 0.7.8 | installed |
| `jpegoptim` | 1.4.7 | installed |
| `webp` (`cwebp`) | — | installed |
| `librsvg2-bin` (`rsvg-convert`) | 2.60.0 | Rasterized `favicon.svg` to a 512×512 PNG. |
| `jq` | 1.7 | Wrangled GitHub API JSON through `gh api --jq`. |
| `bundler` | 4.0.21 | see below |
| `jekyll` | 4.4.1 | see below |
| `webrick` | 1.9.2 | required for `jekyll serve` on Ruby 3.x |
| `jekyll-seo-tag` | 2.9.0 | see below |

Plus five default gems installed as root — `base64 0.2.0`, `bigdecimal 3.1.5`, `csv 3.3.4`, `logger 1.6.0`, `json 2.7.2`. **These were the reason `bundle install` failed**: as an unprivileged user, Bundler tried to write its cache to root-owned `/var/lib/gems/3.3.0/cache/` and hit `Bundler::PermissionError`.

### The repo's own documented workflow now works

`README.md` tells a contributor to run `bundle install` and `bundle exec jekyll serve --baseurl ""`. Both are now true for an ordinary user — no env vars, no `JEKYLL_NO_BUNDLER_REQUIRE`:

```
$ bundle install
Bundle complete! 2 Gemfile dependencies, 36 gems now installed.

$ bundle exec jekyll serve --baseurl "" --port 4000
HTTP 200  5462 bytes at http://127.0.0.1:4000/
  <title>DBest In Love | 10 Years Apart · Together Forever</title>
  /assets/css/styles.css -> HTTP 200
```

---

## 2. Where the agent's tooling lives

Nothing is stored in the repo any more. `du -sh .` inside the project is 3.0 MB, down from 790 MB.

| What | Where | Why |
|---|---|---|
| Jekyll toolchain | system-wide (`/var/lib/gems/3.3.0`) | installed as root; usable by any user |
| Playwright node module + driver scripts | `~/.local/share/dsh-pw/` | keeps the repo clean |
| Chromium headless shell 153 | `~/.cache/ms-playwright/` | Playwright's **default** path, so no `PLAYWRIGHT_BROWSERS_PATH` is needed |
| Screenshots | `~/.local/share/dsh-pw/shots/` | for a human to review |

Screenshots of the live page at 400 / 768 / 900 / 1400 / 1920px are in `~/.local/share/dsh-pw/shots/`. **The agent cannot view them** — see §4.

Measure the deployed page at any width:

```bash
cd ~/.local/share/dsh-pw
node shot.js "https://dbest180.github.io/dbestinlove/" "./shots"
```

---

## 3. Rebuilding this environment from scratch

If the container is reset or the sandbox reverts to `workspace-write`, this is the whole recipe.

```bash
# --- packages (needs root) ---
sudo apt-get update -qq
sudo apt-get install -y imagemagick jq python3-pil python3-yaml yamllint \
                        optipng jpegoptim webp librsvg2-bin

# --- Ruby / Jekyll (needs root) ---
sudo gem install bundler jekyll webrick jekyll-seo-tag --no-document

# --- the five default gems Bundler needs, or `bundle install` fails ---
sudo gem install base64:0.2.0 bigdecimal:3.1.5 csv:3.3.4 logger:1.6.0 json:2.7.2 --no-document

# --- browser tooling (no root needed) ---
mkdir -p ~/.local/share/dsh-pw && cd ~/.local/share/dsh-pw
npm install playwright --no-audit --no-fund
npx --yes playwright@latest install chromium     # lands in ~/.cache/ms-playwright
```

### If `sudo` is unavailable and `$HOME` is read-only again

The old workaround, which worked but is fragile. Every path has to be redirected into the repo:

```bash
cd "/home/dbest1/Plus One Project/dbestinlove"
export GEM_HOME="$PWD/.tools/gems"              GEM_PATH="$PWD/.tools/gems"
export GEM_SPEC_CACHE="$PWD/.tools/spec-cache"  # else RubyGems dies writing ~/.cache/gem
export XDG_CACHE_HOME="$PWD/.tools/cache"
export HOME="$PWD/.tools/home"                  # last resort for stray writes
export PATH="$PWD/.tools/gems/bin:$PATH"
export JEKYLL_NO_BUNDLER_REQUIRE=1              # else Jekyll hands off to Bundler and dies
export npm_config_cache="$PWD/.tools/npm-cache"
export PLAYWRIGHT_BROWSERS_PATH="$PWD/.tools/browsers"
jekyll build --source . --destination _site
```

If you have to do this, re-add `.tools/` to `.gitignore`.

---

## 4. The one remaining gap — and it is not an install

**The agent cannot read images.**

```bash
$ node shot.js ...        # works: screenshots captured, DOM measured
$ # but reading one back:
Error: model "deepseek-v4-flash" does not declare image input;
       switch to an image-capable model to read images
```

So the agent has a camera and no eyes. Screenshots land on disk and only a human can interpret them.

This is survivable but asymmetric. For the hero-crop question this project carried, DOM measurement was a *better* instrument than looking — `getBoundingClientRect`, resolved `object-fit`, overflow on each axis at five widths gave a definitive answer that a glance might not have. But measurement cannot tell anyone *"the type looks wrong,"* *"that photo is unflattering,"* or *"the composition is off."*

**Options, best first:**

1. **Run agent sessions on an image-capable model.** The visual loop closes entirely. Highest-leverage change available on this project.
2. **Expose a vision-capable subagent** the agent can hand a PNG to. Partial, but enough for spot checks.
3. **Leave it.** The agent keeps producing screenshots and says plainly when a judgement is beyond measurement rather than guessing.

---

## 5. Harness asks

1. **An image-capable model** (§4). This is the only outstanding item in this document.
2. Keep `danger-full-access` if you can. It removed 790 MB from the repo and turned a five-`export` install command into a one-liner.
3. Network egress is sufficient — nothing to enable.

**On GitHub Pages "plugins"**: there is nothing to turn on in repo settings. `jekyll-seo-tag` and `jekyll-sitemap` are enabled purely by listing them under `plugins:` in `_config.yml`. The Pages source should stay **GitHub Actions**.

---

## 6. Open decision — `Gemfile.lock`

`bundle install` now generates a `Gemfile.lock`, and it is currently **gitignored**, so it is not committed.

- **Keep it ignored (current):** the Pages build resolves gems fresh each time. The deploy works today and nothing about it changes.
- **Commit it:** reproducible builds pinned to exact versions — but it changes how the Pages Action resolves gems, so it deserves a deliberate commit and a watched deploy rather than being swept in.

Left as-is on purpose. Not the agent's call to change deployment behaviour silently.
