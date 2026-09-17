# HANDOFF — DBest In Love (v3.4)

**Repo:** `dbest180/dbestinlove`
**Live URL:** https://dbest180.github.io/dbestinlove/
**Stack:** Jekyll → GitHub Pages via Actions (`actions/jekyll-build-pages@v1`)
**v3.4 status:** Every P0 and P1 item from v3 is implemented. What's left is either quick/independent (P2/P3) or genuinely blocked on video content existing. Only TikTok is live; Instagram/YouTube don't exist yet and nothing should reference them until they do.

---

## 1. What shipped since v3

| Item | What happened |
|---|---|
| Dead "Soon" buttons | Instagram and YouTube entries removed from `_data/links.yml`. Only TikTok remains. Re-add entries there (with a `url:`) once those accounts exist — that's the only file that needs touching. |
| Visible handle | Already satisfied — `sub:` on the TikTok button reads `@dbestinlove — new videos weekly`. No action was needed. |
| Hero photo (P0) | Implemented, but with a **stand-in image**, not a photo of the couple — see §2. Full-bleed above the `<h1>`, `srcset` at 700w/1400w, recompressed to 86 KB / 31 KB from a 604 KB original. |
| Social card (P0) | `image: /assets/img/og.jpg` added to `_config.yml`; `jekyll-seo-tag` picks it up automatically. 1200×630, 76 KB, cropped from the same stand-in photo. Not yet verified in an actual link-preview test — see §2. |
| Dead code (P3) | The `document.getElementById('year')` block removed from `assets/js/main.js` — the footer already uses `{{ site.time \| date: "%Y" }}` via Liquid, so this never fired. |
| `.gitignore` (P3) | Added: `_site/`, `.jekyll-cache/`, `.sass-cache/`. |
| `robots.txt` (P2) | Added: `User-agent: *` / `Allow: /`. No `Sitemap:` line yet — add one when `jekyll-sitemap` goes in. |

---

## 2. Open items, and what's blocking them

### Needs a person's decision or asset (not blocked on video content)

- **Hero photo is a placeholder.** The current hero/og image is a photo of red and blue ink merging in water — chosen deliberately for the moment (it echoes the site's flag-merge palette) but it is **not a photo of the two people in the story**. The original P0 problem HANDOFF v3 raised — "the site has no faces" — is still technically true. Swap in a real photo of the couple when one is ready: replace `assets/img/hero-1400.jpg`, `hero-700.jpg`, and `og.jpg` (same filenames, same dimensions — 1400×764, 700×382, 1200×630 — keeps `index.md` and `_config.yml` untouched), recompress the same way (`convert -strip -interlace Plane -resize <W>x -quality 78`), and update the `alt` text in `index.md` to describe the actual photo instead of the ink swirl.
- **Unverified hero-photo crop on live.** A screenshot showed the hero image cropped much tighter than intended — narrow width, tall vertical strip, losing the sides of the composition. The CSS math for the full-bleed breakout (`.hero__photo` in `styles.css`) checks out on paper, so the leading theories are a stale cached `styles.css` on the viewer's end, or the CSS/markup not both having been pushed together. **Not yet confirmed against the actual live URL at full browser width with a hard refresh.** The `.hero__photo img` height clamp was already loosened (`34vw` cap instead of `46vw`, `object-position: 50% 55%`) as a hedge either way, but this needs a real check before being called done: open the live URL, hard-refresh, resize the window, and confirm the photo spans full viewport width with `object-fit: cover` only trimming top/bottom, not sides.
- **`og:image` social card unverified.** Added to config but never actually tested by pasting the URL into a chat client, which is the one way to catch a broken preview before it matters.
- **Old unused `assets/img/hero.jpg`** (604 KB, the original untouched upload) is superseded by `hero-1400.jpg`/`hero-700.jpg` and should be deleted from the repo.
- **Sitemap** (`jekyll-sitemap` plugin) and a **404.md** page — independent, no blockers, just not done yet.

### Blocked on video content existing

- **Story "start here" chronology.** Deliberately held per the site owner's call: don't touch the `.story` section's structure until at least the first video is posted. See §3 — this is the main thing a future agent should pick up once that happens.
- **Re-evaluating the hero/social-card photo choice** once real footage exists to pull a still from, if a dedicated photo of the couple still isn't available by then.

---

## 3. For whoever picks this up once videos and scripts are ready

This is the trigger condition the site owner set: once the first few videos are drafted/posted, the story section should be updated to reflect them. When that happens:

1. **Get the video order and titles/topics** from the owner — the story section needs to tell a new TikTok arrival "start with this one."
2. **Add a short chronology to the `.story` section in `index.md`** — years or "chapter" markers, per the original HANDOFF ask. Keep it consistent with the existing prose style (EB Garamond, first-person plural, understated) rather than turning it into a bullet list; this site's whole voice is quiet and literary, not marketing copy.
3. **Check whether the hero/social-card photo should change too** — if by then there's a real photo of the couple (from a video still or otherwise), replace the ink-swirl stand-in per §2's instructions.
4. **Re-test the social card** after any image swap — paste the live URL into a chat client and confirm the preview renders.
5. **Leave Instagram/YouTube alone** unless those accounts now exist. If they do, re-add entries to `_data/links.yml` with real `url:` values — don't restore the dashed "Soon" placeholders.

---

## 4. Gotchas (carried forward, still true)

1. **kramdown does not parse markdown inside HTML blocks.** `parse_block_html: false` is the default. Text inside `<section>` or `<div>` needs real `<p>`/`<em>` tags, not `*markdown*` syntax.
2. **`url` in `_config.yml` must not include `baseurl`.** `url: "https://dbest180.github.io"` + `baseurl: "/dbestinlove"`. Doubling the path breaks every canonical tag and share preview. This was live once.
3. **Never add an `index.html`.** GitHub Pages serves it before `index.md`, silently hiding the whole site. This was the original v1 bug.
4. **Check `git status -sb` before pushing.** `main` has diverged before. `git reset --soft origin/main` then commit is the safe recovery — never `git push --force`.
5. **No Ruby in the agent environment** — no local Jekyll preview. The loop is push → `gh run watch` → check the live URL by hand. This is also why the hero-photo crop issue in §2 hasn't been confirmed yet: it can only be verified against the actual deployed page in a real browser, not from the container.
6. **`{% seo %}` reads `_config.yml`, not the page.** Title, description, and `image` all come from config unless overridden in front matter.
7. **New:** when changing anything in `assets/img/`, keep filenames stable (`hero-1400.jpg`, `hero-700.jpg`, `og.jpg`) so `index.md` and `_config.yml` never need touching for an image swap — just overwrite the files.

---

## 5. What not to do

- Don't re-add the newsletter section (removed deliberately in v2).
- Don't add an `index.html`.
- Don't hardcode the footer year — `{{ site.time | date: "%Y" }}` is correct.
- Don't rename the repo or `baseurl`.
- Don't commit `_site/`.
- Don't swap the type pairing back (EB Garamond for story / Jost for interface).
- Don't restore Instagram/YouTube "Soon" placeholders — delete-until-real was the explicit decision this round.
- Don't touch the story section's structure before a video is actually posted — that's a deliberate hold, not an oversight.

---

## 6. Definition of done for v3.4's remaining scope

- [ ] Hero photo crop confirmed correct on the live URL (full width, hard-refreshed)
- [ ] Social card confirmed in an actual link-preview test
- [ ] Old unused `assets/img/hero.jpg` removed from the repo
- [ ] Sitemap and 404 page in place
- [ ] Story chronology added once the first video is live
- [ ] Real photo of the couple in place, if/when available, replacing the ink-swirl stand-in
