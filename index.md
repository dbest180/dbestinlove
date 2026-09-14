---
layout: default
---

<header class="hero">
  <p class="kicker">A love story, told out loud</p>
  <h1 class="hero__title">The Plus One<br>Project</h1>
  <div class="merge-bar" aria-hidden="true"></div>
  <p class="hero__tagline">10 Years Apart · Together Forever</p>
  <p class="hero__hook">
    We met on Google+. We survived 10 years, 3 countries, and 3 years of
    COVID. This is our love story.
  </p>
</header>

<nav class="links reveal" id="links" aria-label="Find us online">
  {% for link in site.data.links %}
    {% if link.url %}
    <a class="btn btn--{{ link.style }}"
       href="{{ link.url }}"
       {% if link.external %}target="_blank" rel="noopener"{% endif %}>
    {% else %}
    <div class="btn btn--{{ link.style }}">
    {% endif %}
      <span class="btn__icon">{% include icon.html name=link.icon %}</span>
      <span class="btn__text">
        <strong>{{ link.label }}</strong>
        <small>{{ link.sub }}</small>
      </span>
      {% if link.badge %}
        <span class="pill">{{ link.badge }}</span>
      {% else %}
        <span class="btn__arrow" aria-hidden="true">→</span>
      {% endif %}
    {% if link.url %}</a>{% else %}</div>{% endif %}
  {% endfor %}
</nav>

<section class="story section reveal">
  <h2 class="section-title">Our Story</h2>

  We met in a comment section nobody remembers. Ten years, three countries,
  and three years of COVID later — we got married.

  Different time zones. The same stubborn decision, made over and over:
  *not yet, but not never.*

  [Watch the full story on TikTok →](https://www.tiktok.com/@dbestinlove)
</section>

<footer class="footer">
  <p><a href="https://www.tiktok.com/@dbestinlove" target="_blank" rel="noopener">TikTok</a></p>
  <p>© {{ site.time | date: "%Y" }} The Plus One Project</p>
  <p>Made with 💛</p>
</footer>
