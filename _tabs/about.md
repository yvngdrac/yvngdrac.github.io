---
layout: default
title: About
permalink: /about/
---

<div class="page-content">
  <h1 class="page-title">About Me</h1>
  
  <div class="about-content">
    <div class="about-intro">
      <p>Hi, I'm <strong>yvngdrac</strong>, a cybersecurity enthusiast passionate about offensive security, CTF challenges, and penetration testing.</p>
      <p>This blog documents my journey through various security challenges, lab environments, and real-world inspired scenarios.</p>
    </div>

    <div class="about-section">
      <h2><i class="fas fa-bullseye"></i> Focus Areas</h2>
      <ul>
        <li>Web Application Security & Penetration Testing</li>
        <li>Binary Exploitation & Reverse Engineering</li>
        <li>Network Security & Traffic Analysis</li>
        <li>Digital Forensics & Incident Response</li>
        <li>CTF Competitions & Writeups</li>
      </ul>
    </div>

    <div class="about-section">
      <h2><i class="fas fa-book"></i> Purpose of This Blog</h2>
      <p>I created this blog to:</p>
      <ul>
        <li>Document my technical writeups and methodologies</li>
        <li>Share knowledge with the security community</li>
        <li>Track my growth and learning progress</li>
        <li>Help others learn from my experiences</li>
      </ul>
      <p>All content is created for educational purposes only.</p>
    </div>

    <div class="about-section">
      <h2><i class="fas fa-trophy"></i> CTF Experience</h2>
      <ul>
        <li>HackTheBox CTF Competitions</li>
        <li>TryHackMe Challenges</li>
        <li>Local CTF Events</li>
        <li>Boot2Root Machines</li>
      </ul>
    </div>

    <div class="about-section">
      <h2><i class="fas fa-envelope"></i> Connect With Me</h2>
      <div class="social-links">
        <a href="https://github.com/yvngdrac" target="_blank" class="social-link">
          <i class="fab fa-github"></i> GitHub
        </a>
        <a href="#" class="social-link">
          <i class="fab fa-twitter"></i> Twitter
        </a>
        <a href="#" class="social-link">
          <i class="fab fa-linkedin"></i> LinkedIn
        </a>
      </div>
    </div>
  </div>
</div>

<style>
.page-content {
  max-width: 700px;
  margin: 0 auto;
}

.page-title {
  font-family: var(--mono);
  font-size: 2rem;
  color: var(--text-head);
  margin-bottom: 2rem;
  border-bottom: 2px solid var(--accent);
  display: inline-block;
  padding-bottom: 0.5rem;
}

.about-content {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 2rem;
}

.about-intro {
  font-size: 1.05rem;
  line-height: 1.7;
  margin-bottom: 2rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border);
}

.about-section {
  margin-bottom: 2rem;
}

.about-section:last-child {
  margin-bottom: 0;
}

.about-section h2 {
  font-family: var(--mono);
  font-size: 1.2rem;
  color: var(--accent);
  margin-bottom: 1rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.about-section h2 i {
  font-size: 1rem;
}

.about-section p {
  margin-bottom: 1rem;
  line-height: 1.7;
  color: var(--text);
}

.about-section ul {
  list-style: none;
  padding-left: 0;
}

.about-section li {
  padding: 0.4rem 0;
  padding-left: 1.5rem;
  position: relative;
  color: var(--text);
}

.about-section li::before {
  content: "→";
  position: absolute;
  left: 0;
  color: var(--accent);
}

.social-links {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  margin-top: 1rem;
}

.social-link {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  background: var(--tag-bg);
  border: 1px solid var(--tag-border);
  border-radius: 8px;
  color: var(--text);
  transition: all 0.15s;
}

.social-link:hover {
  color: var(--accent);
  border-color: var(--accent);
  transform: translateY(-2px);
}

.social-link i {
  font-size: 1rem;
}
</style>
