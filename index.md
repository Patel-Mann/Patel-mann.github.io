---
layout: default
title: Home
---

<section class="hero">
  <h1>{{ site.name }}</h1>
  <p class="bio">{{ site.bio }}</p>
  <p class="location">{{ site.location }}</p>
  <div class="social-links">
    {% if site.github %}
    <a href="https://github.com/{{ site.github }}">GitHub</a>
    {% endif %}
    {% if site.linkedin %}
    <a href="https://linkedin.com/in/{{ site.linkedin }}">LinkedIn</a>
    {% endif %}
    {% if site.twitter %}
    <a href="https://twitter.com/{{ site.twitter }}">Twitter</a>
    {% endif %}
  </div>
</section>

<section class="about-preview">
  <h2>About Me</h2>
  <p>Hi! I'm a developer passionate about building great software. Welcome to my personal portfolio where I share my projects, thoughts, and experiences.</p>
  <p><a href="/about">Learn more about me →</a></p>
</section>

<section class="projects-preview">
  <h2>Featured Projects</h2>
  <p><a href="/projects">View all projects →</a></p>
</section>