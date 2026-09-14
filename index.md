---
layout: default
---

<header class="hero">
  <p class="kicker">A love story, told out loud</p>
  <h1 class="hero__title">The Plus&nbsp;One Project</h1>
  <p class="hero__tagline">10 Years Apart. Together Forever.</p>
  <p class="hero__hook">
    We met on Google+. We survived 10 years, 3 countries, and 3 years of
    COVID. This is our love story.
  </p>
  <div class="distance" aria-hidden="true">
    <svg viewBox="0 0 260 24"><line x1="14" y1="12" x2="246" y2="12"/><circle cx="14" cy="12" r="5"/><circle cx="246" cy="12" r="5"/></svg>
  </div>
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

  We met in a comment section nobody remembers. Then came ten years of
  screens — 3 a.m. calls, airport goodbyes, and a pandemic that kept us
  apart for three of them.

  Different countries. Different time zones. The same stubborn decision,
  made over and over: *not yet, but not never.*

  We got married. This is what we learned on the way.

  [Read the full story →](#)
</section>

<section class="signup section reveal" id="newsletter">
  <h2 class="section-title">For anyone missing someone far away.</h2>
  <p class="signup__sub">You're not alone. Get weekly encouragement and LDR survival tips.</p>

  <form class="signup__form" action="https://buttondown.email/api/emails/embed-subscribe/dbestinlove">
    <label class="sr-only" for="email">Email address</label>
    <input id="email" name="email" type="email" inputmode="email"
           autocomplete="email" placeholder="dbestinlove@gmail.com" required>
    <button type="submit">Join</button>
  </form>
  <p class="signup__note">No spam. Unsubscribe anytime.</p>
</section>

<footer class="footer">
  <p><a href="https://www.tiktok.com/@dbestinlove" target="_blank" rel="noopener">TikTok</a></p>
  <p>© {{ site.time | date: "%Y" }} The Plus One Project</p>
  <p>Made with 💛</p>
</footer>
