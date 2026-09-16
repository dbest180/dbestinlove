# HANDOFF — DBest In Love (v3)

**Repo:** `dbest180/dbestinlove`
**Live URL:** https://dbest180.github.io/dbestinlove/
**Stack:** Jekyll → GitHub Pages via Actions (`actions/jekyll-build-pages@v1`)
**v3 goal:** Make the page convert TikTok traffic — a face, a hook, and a reason to follow.

---

## 1. The name

**"The Plus One Project" is retired.** It read like a product launch, not a person telling you something true. Nobody's first instinct on reading it is *"who are these two?"* — which is the only question that matters when the traffic is arriving from a TikTok story.

**New name: DBest In Love.** It matches the handle `@dbestinlove`, so the bio link and the site reinforce one name instead of splitting recognition across two. Descriptor: **a 10-year long-distance (LDR) journey**.

The name appears in more places than you'd expect. Change all of them together or the site goes inconsistent:

| File | Where |
|---|---|
| `_config.yml` | `title`, `author`, `description` |
| `index.md` | `<h1 class="hero__title">`, footer copyright line |
| `assets/css/styles.css` | header comment |
| `assets/js/main.js` | header comment |
| `README.md`, `HANDOFF.md` | docs |

**Deliberately unchanged:** the repo name, `baseurl: /dbestinlove`, and therefore the live URL. Every link already sitting in a TikTok bio or a pinned comment keeps working. Don't rename the repo unless you're ready to break those.

**Open decision — where the descriptor lives.** "10 yr LDR journey" currently sits in the meta description (`_config.yml`), not on the page, because the hero already reads *"10 Years Apart"* one line under the title and repeating it reads redundant. If you want it visible, the natural slot is the kicker above the `<h1>` — swap `A love story, told out loud` for `A 10-Year LDR Journey`. Your call.

---

## 2. Where v3 starts

Live and verified at commit `5617393`:

- White-dominant palette with flag-merge accents (T&T red → plum → Old Glory blue)
- EB Garamond for story content, Jost for the interface
- Hero → 3 link buttons → story → footer with Liquid-rendered year
- `jekyll-seo-tag` wired; canonical and `og:url` correct
- Build green in ~50 seconds

Everything below is a v3 addition. Nothing here is a bug fix — v2 is clean.

---

## 3. v3 action items

### P0 — the page has no faces

`assets/img/hero.jpg` exists: a real **1408×768 JPEG, 604 KB** — and **nothing references it**. The site is a love story with no photograph of the two people in it. For someone who just watched a video and tapped the bio link, that's the single biggest reason to leave.

- [ ] Put a real photo above the fold
- [ ] Recompress — 604 KB is heavy for a hero; target under 150 KB, and serve two widths
- [ ] Write `alt` text describing the *moment*, not the file
- [ ] Consider a second photo in the story section; faces are the whole product here

### P0 — link previews are blank

There is no `og:image`, so pasting the site into a bio, a DM, or a comment yields a bare text card. You are asking people to share a link that looks broken.

- [ ] Make a **1200×630** social card (a frame from the story works fine, with the name set in EB Garamond)
- [ ] Add `image: /assets/img/og.jpg` to `_config.yml` — `jekyll-seo-tag` picks it up automatically
- [ ] Test it by pasting the URL into a chat; don't assume

### P1 — two of the three buttons are dead

Instagram and YouTube render as dashed "Soon" placeholders with no `url:`. They sit directly beneath the primary CTA, in the most valuable real estate on the page, and they advertise absence. Pick one:

- [ ] Delete them until the accounts exist (cleanest), **or**
- [ ] Point them at the TikTok profile for now, **or**
- [ ] Change `sub:` to something that earns the click ("Coming once we're settled")

### P1 — every path should end at the same action

The page has three exits to TikTok and one dead end at the bottom. Make the follow impossible to miss:

- [ ] Keep the primary button in the merge gradient — it's working
- [ ] Add the handle `@dbestinlove` as visible text somewhere; people search the handle, not the URL
- [ ] Make sure the `text-link` CTA at the end of the story is genuinely the last thing people read

### P1 — the story needs a "start here"

TikTok viewers arrive mid-story, out of order, with no idea which video was first. The story section is four paragraphs of prose with no entry point.

- [ ] Add a short chronology (years, or "chapter" markers) so a new arrival can orient in seconds
- [ ] Consider naming the first video, so the page can say "start with this one"

### P2 — discoverability

- [ ] Add `jekyll-sitemap` to `plugins:` in `_config.yml`
- [ ] Add a `robots.txt`
- [ ] In **Settings → Pages**, confirm the source is still *GitHub Actions*

### P2 — no 404 page

A bad link currently gets GitHub's default. A one-line `404.md` with the same layout and a link home is nearly free.

### P2 — font payload

Two Google Fonts families, nine weights, render-blocking. Fine now, worth revisiting if you add analytics and see a slow mobile load: self-host or subset to the weights actually used.

### P3 — dead code

`assets/js/main.js` still looks for `document.getElementById('year')` — a footer span v2 deleted. It's guarded and harmless, but it's the last v1 remnant in the tree.

- [ ] Delete those two lines

### P3 — no `.gitignore`

There isn't one. Run `jekyll build` locally and `_site/` appears untracked, one careless `git add -A` away from being committed.

- [ ] Add `.gitignore` with `_site/`, `.jekyll-cache/`, `.sass-cache/`

### P3 — repo hygiene

`HANDOFF.md` is committed to a **public** repo. It's in `_config.yml`'s `exclude:`, so it is *not* served as part of the site — but it is readable in the repo and its history. If you'd rather it weren't:

```bash
git rm --cached HANDOFF.md && echo "HANDOFF.md" >> .gitignore
```

---

## 4. Gotchas — each of these already cost time once

**1. kramdown does not parse markdown inside HTML blocks.** The default is `parse_block_html: false`. Text inside `<section>` or `<div>` is passed through raw: paragraphs never become `<p>`, and `[links](...)` stay literal. The story section is written as explicit `<p>` and `<em>` for exactly this reason. To use markdown inside HTML, add `markdown="1"` to the tag, or set `parse_block_html: true` in `_config.yml`.

**2. `url` must not include `baseurl`.** It's `url: "https://dbest180.github.io"` + `baseurl: "/dbestinlove"`. Put the path in both and every canonical and `og:url` becomes `/dbestinlove/dbestinlove`. This was live once.

**3. Never add an `index.html`.** GitHub Pages serves `index.html` *before* `index.md`, which hides the entire Jekyll homepage with no error anywhere. That exact file was the original v1 bug — 146 lines of mockup silently winning over the real site.

**4. Check `git status -sb` before pushing.** `main` has already diverged once: the local clone was 6 commits behind *and* 1 ahead, with both sides having independently "fixed" the same problems. A plain push gets rejected; a force push would have destroyed the other side. The move that loses nothing is `git reset --soft origin/main` then commit — it replays your tree on top of the remote and stays a fast-forward. **Never `git push --force` to this branch.**

**5. No Ruby in the agent environment.** Jekyll isn't installed, so there's no local preview — the loop is push → watch the Action → read the live URL. Run `gh run watch` to follow a build. For a real local preview:

```bash
bundle install
bundle exec jekyll serve --baseurl ""
```

The empty `--baseurl` matters, or every asset 404s on localhost.

**6. A `Gemfile` is not load-bearing.** An earlier version of this document claimed the build fails on `jekyll-seo-tag` without one — it doesn't; several builds passed before it existed. Keep it for version pinning, but don't debug builds as if it were required.

**7. `{% seo %}` reads `_config.yml`, not the page.** Title, description, and image all come from config unless overridden in front matter. Changing the name there changes every share preview.

---

## 5. What not to do

- **Don't re-add the newsletter section.** Its removal in v2 was deliberate. The follow action is TikTok; a signup form is a second, weaker ask.
- **Don't add an `index.html`.** Ever. See gotcha 3.
- **Don't hardcode the year** in the footer. `{{ site.time | date: "%Y" }}` is already correct.
- **Don't rename the repo or `baseurl`** without accepting that old links break.
- **Don't commit `_site/`.**
- **Don't swap the type back.** EB Garamond for story, Jost for interface is the v3 decision — the split is what makes the page read like writing rather than an app.

---

## 6. Definition of done for v3

- [ ] A real photo above the fold
- [ ] The social card renders in a link preview
- [ ] No dead "Soon" buttons
- [ ] Sitemap and 404 in place
- [ ] Build green, and the live URL opened by hand — not just the Action log

```bash
# catches a half-finished rename
grep -rin "the plus one project" --exclude-dir=.git --exclude-dir=_site --exclude=HANDOFF.md .
```

Search the bare phrase `plus one` instead and you'll get one hit in `index.md`: *"we stopped being each other's plus one and became each other's home."* That one is deliberate — it's a common noun, it's a good line, and it reads true under any name. Leave it.
