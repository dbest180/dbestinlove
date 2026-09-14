# HANDOFF — The Plus One Project (v2 Redesign)

**Repo:** `dbest180/dbestinlove`
**Live URL:** https://dbest180.github.io/dbestinlove/
**Stack:** Jekyll → GitHub Pages via Actions
**Goal:** Land v2 redesign (white-dominant, flag-merge palette) and remove all v1 remnants.

---

## Situation

The site is running **v1** (cream palette, Fraunces/Inter, newsletter section) because v2 files were either never committed or committed with corruption. The creator has a local clone of the repo. Your job is to audit the working tree, fix the issues below, and leave it ready to push.

Do not redesign anything. Do not add features. Restore the intended v2 state only.

---

## Canonical v2 State — What Each File Must Contain

### 1. `index.html` — **must not exist**

This is a leftover mockup from before we chose Jekyll. GitHub Pages serves `index.html` *before* `index.md`, so as long as it exists, the Jekyll homepage is invisible.

**Action:** `git rm index.html` (or delete from working tree if untracked).

Verify with: `ls index.html` → should error "No such file."

---

### 2. `_layouts/default.html` — verify it matches v2

Must contain:
- `theme-color: #FFFFFF`
- Font link for **Cormorant Garamond** (weights 300/400/500 + italic) and **Jost** (weights 200/300/400/500). **Not** Fraunces/Inter.
- `{% seo %}` tag (Jekyll SEO plugin)
- `{% include %}` free — content is `{{ content }}` inside `<main class="wrap">`
- Skip link to `#links`
- No `id="year"` span in footer (year is rendered via Liquid in `index.md`)

If it still references Fraunces, Inter, or `assets/js/main.js` with a different path, replace with v2 version.

---

### 3. `assets/css/styles.css` — **likely corrupted, check carefully**

**Known issue from the last commit:** v2 CSS was pasted on top of v1 without deleting v1, leaving an orphaned block. The corruption starts at a line like:

```

}  color: var(--ink);
background-color: var(--cream);

```

…and continues to the end of the file with v1 remnants (`.signup`, `.distance`, `.btn--gold`, `--cream`, `--terracotta`, `--maroon`, `--gold`, the `@media (min-width: 480px)` block for the signup form).

**Action:** Truncate the file so it ends exactly at:

```css
@media (prefers-reduced-motion: reduce) {
  .js .reveal { opacity: 1; transform: none; }
  * { animation: none !important; transition: none !important; }
}
```

Nothing after that. No trailing v1 code.

Sanity checks on what remains:

· :root defines --tt-red, --tt-black, --us-red, --us-blue, --white, --merge, --ink, --ink-soft, --ink-faint, --line, --bg, --display, --ui, --wrap, --r, --shadow-1, --shadow-2
· No references to --cream, --terracotta, --maroon, --gold, --paper, --serif, --sans
· .hero__title uses font-weight: 300 and clamp(3rem, 15vw, 7.5rem)
· .merge-bar exists with the red→black→blue horizontal gradient
· .btn--primary uses background: var(--merge)
· No .signup rules anywhere
· No .distance rules anywhere (that was v1)

Run a quick grep to confirm: grep -nE "cream|terracotta|maroon|signup|Fraunces|Inter" assets/css/styles.css → should return zero matches.

---

4. index.md — verify it matches v2

Must contain:

· Front matter: layout: default
· Hero header with <div class="merge-bar" aria-hidden="true"></div> between the <h1> and the tagline
· Hero title is The Plus One<br>Project (two lines, <br> not &nbsp;)
· Tagline uses · separator: 10 Years Apart · Together Forever
· <nav class="links reveal" id="links"> with the Liquid {% for link in site.data.links %} loop
· No <section class="signup"> — the newsletter block must be gone entirely
· Story section ends with [Watch the full story on TikTok →](https://www.tiktok.com/@dbestinlove)
· Footer uses Liquid year: © {{ site.time | date: "%Y" }} The Plus One Project
· Footer has no id="year" span (that was v1's JS-injected year)

Run: grep -n "signup\|newsletter\|Join the Community" index.md → should return zero matches.

---

5. _data/links.yml — must have exactly 3 entries

Correct entries (in order):

```yaml
- label: Watch on TikTok
  sub: "@dbestinlove — new videos weekly"
  url: https://www.tiktok.com/@dbestinlove
  icon: tiktok
  style: primary
  external: true

- label: Instagram
  sub: Behind-the-scenes & daily life
  icon: instagram
  style: soon
  badge: Soon

- label: YouTube
  sub: Longer stories & full conversations
  icon: youtube
  style: soon
  badge: Soon
```

Must NOT contain the "Join the Community" / mail / style: gold / url: "#newsletter" block. Delete it if present.

Run: grep -n "Join the Community\|newsletter\|mail" _data/links.yml → zero matches.

---

6. _config.yml — verify

Must contain:

```yaml
url: "https://dbest180.github.io"
baseurl: "/dbestinlove"
plugins:
  - jekyll-seo-tag
```

If baseurl or url are wrong, {% seo %} produces broken canonical/OG tags.

---

7. Gemfile — must exist

Required for actions/jekyll-build-pages to resolve jekyll-seo-tag. Minimum:

```ruby
source "https://rubygems.org"
gem "jekyll", "~> 4.3"
gem "jekyll-seo-tag", "~> 2.8"
```

If missing, the Pages Action fails on first build with "Could not find jekyll-seo-tag."

---

8. _includes/icon.html — verify it has 4 cases

Must handle: tiktok, mail, instagram, youtube. Even though mail is no longer used by any link, keep it — harmless, and useful if a newsletter link returns.

---

9. .github/workflows/pages.yml — verify

Must use actions/jekyll-build-pages@v1, actions/upload-pages-artifact@v3, actions/deploy-pages@v4. Permissions must include pages: write and id-token: write.

---

What NOT to Do

· Do not re-add the newsletter section to index.md.
· Do not re-add a "Join the Community" entry to links.yml.
· Do not "improve" the palette, fonts, or layout. v2 is deliberate.
· Do not touch _posts/ — doesn't exist yet, that's for later.
· Do not rename files or restructure folders.
· Do not create an index.html.

---

Deliverable

A working tree where:

1. index.html does not exist
2. styles.css ends cleanly at the prefers-reduced-motion block
3. index.md has no signup section
4. links.yml has 3 entries
5. All greps listed above return zero matches

Run these before declaring done:

```bash
# 1. No stale HTML
ls index.html 2>&1 | grep -q "No such file" && echo "OK" || echo "FAIL: index.html present"

# 2. No v1 CSS remnants
! grep -qE "cream|terracotta|maroon|--signup|\.signup" assets/css/styles.css && echo "OK" || echo "FAIL: v1 CSS in styles.css"

# 3. No newsletter in index.md
! grep -q "signup\|newsletter\|Join the Community" index.md && echo "OK" || echo "FAIL: newsletter in index.md"

# 4. links.yml has 3 entries
[ "$(grep -c '^- label:' _data/links.yml)" = "3" ] && echo "OK" || echo "FAIL: links.yml entry count"
```

All four must print OK.

Do not commit or push. Return the working tree to the creator for review. They will run git add -A && git commit && git push.

---

After Push — Expected Result

· Action runs green in ~40s
· https://dbest180.github.io/dbestinlove/ shows:
  · White background with faint red/blue ambient gradient
  · "The Plus One Project" in large thin Cormorant Garamond, two lines
  · A horizontal red→black→blue merge bar under the title
  · Tagline in spaced uppercase Jost
  · TikTok button with red-to-blue gradient background
  · Instagram and YouTube as dashed "Soon" placeholders
  · "Our Story" section linking to TikTok
  · No newsletter form
  · Footer with TikTok link and Liquid-rendered year

If any of those are false, the corresponding file above is still wrong.

```