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
  <p class="intro">Hi, my name is Mann, and I enjoy working on backend and embedded software, with a particular focus on improving the performance of backend systems. In my free time, I work on embedded projects that combine software and hardware, allowing me to learn how a complete product comes to market and understand the underlying principles behind smart devices.</p>
</section>

<section class="projects">
  <h2>Projects</h2>

  <div class="project-card project-card-with-image">
    <div class="project-image">
      <img src="/assets/images/fin-track.gif" alt="Fin-Track Application Screenshot">
    </div>
    <div class="project-content">
      <a href="/projects/fin-track">
        <h3>Fin-Track: CalgaryHacks 2025</h3>
      </a>
      <p class="project-meta">December 2025 </p><p class="award-badge-inline">Winner: Best Hardware Implementation</p>
      <p class="tech-stack">React | Next.js | Supabase | Plaid API | ESP32</p>
      <p>An offline-first personal finance system with a physical ESP32 LCD display showing real-time banking balances. Combined expense tracking, subscription management, and financial literacy quizzes.</p>
    </div>
  </div>

  <div class="project-card project-card-with-image">
    <div class="project-image">
      <img src="/assets/images/pdg-cia-tool.gif" alt="PDG & CIA Tool Screenshot">
    </div>
    <div class="project-content">
      <a href="/projects/pdg-cia-tool">
        <h3>Program Dependence Graphs for Change Impact Analysis</h3>
      </a>
      <p class="project-meta">December 2025 </p>
      <p class="tech-stack">Java | JavaParser | AST | CFG | PDG</p>
      <p>Built a PDG-based change impact analysis tool for Java 1.4 programs. The tool parses source code, constructs a dependency graph capturing control flow and data flow relationships, and performs reachability analysis to determine impact sets for any given line change.</p>
    </div>
  </div>

  <div class="project-card project-card-with-image">
    <div class="project-image">
      <img src="/assets/images/notflix-cover.png" alt="Notflix Screenshot">
    </div>
    <div class="project-content">
      <a href="/projects/notflix">
        <h3>Notflix: A Distributed Streaming Platform</h3>
      </a>
      <p class="project-meta">CPSC559 Final Report </p>
      <p class="tech-stack">Go | net/rpc | SQLite | React | Bully Algorithm</p>
      <p>A multi-node, fault tolerant video streaming service demonstrating distributed systems: HTTP/rpc communication, primary backup replication with quorum commit, leader election, heartbeats with circuit breaker, and pre-leadership log sync for fault tolerance.</p>
    </div>
  </div>
</section>