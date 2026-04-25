---
layout: default
title: Home
---

<section class="hero">
  <h1>{{ site.name }}</h1>
  <p class="bio">{{ site.bio }}</p>
  <p class="location">{{ site.location }}</p>
  <div class="social-links">
    <a href="https://github.com/{{ site.github }}" target="_blank">GitHub</a>
    <a href="https://linkedin.com/in/{{ site.linkedin }}" target="_blank">LinkedIn</a>
    <a href="mailto:{{ site.email }}">Email</a>
  </div>
</section>

<section class="projects">
  <h2>Projects</h2>

  <div class="project-card project-card-with-image">
  <a href="/projects/pdg-cia-tool">
    <div class="project-image">
      <img src="/assets/images/pdg-cia-tool.gif" alt="PDG & CIA Tool Screenshot">
    </div>
    <div class="project-content">
      <h3>Program Dependence Graphs for Change Impact Analysis</h3>
      <p class="project-meta">CPSC 499: Software Analysis | December 2025 | Mann, Nico, Mandeep</p>
      <p>Built a PDG-based change impact analysis tool for Java 1.4 programs. The tool parses source code, constructs a dependency graph capturing control flow and data flow relationships, and performs reachability analysis to determine impact sets for any given line change.</p>
      <p class="tech-stack">JavaParser | AST | CFG | PDG | Reaching Definitions Analysis | GEN/KILL Algorithm</p>
    </div>
  </a>
  </div>
</section>