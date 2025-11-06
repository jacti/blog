---
title: jacti-log
layout: hextra-home
toc: false
sidebar:
  exclude: true
---

<style>
  :root {
    --landing-slate-dark: #475569;
    --landing-slate-medium: #64748b;
    --landing-slate-light: #94a3b8;
    --landing-slate-lighter: #cbd5e1;
    --landing-slate-bg: #1e293b;
    --landing-teal-dark: #0d9488;
    --landing-teal-medium: #14b8a6;
    --landing-teal-light: #06b6d4;
  }

  .landing-page {
    color: var(--landing-slate-dark);
    background-color: #f8fafc;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', sans-serif;
    line-height: 1.6;
  }

  .landing-hero {
    position: relative;
    min-height: calc(100vh - var(--navbar-height));
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 2rem;
    text-align: center;
    background: linear-gradient(135deg, var(--landing-slate-bg) 0%, var(--landing-slate-dark) 100%);
    color: #ffffff;
    overflow: hidden;
    border-radius: 1rem;
  }

  .landing-hero::before {
    content: '';
    position: absolute;
    inset: 0;
    background:
      radial-gradient(circle at 20% 50%, rgba(20, 184, 166, 0.18) 0%, transparent 45%),
      radial-gradient(circle at 80% 80%, rgba(6, 182, 212, 0.18) 0%, transparent 50%);
    pointer-events: none;
  }

  .landing-hero-content {
    position: relative;
    max-width: 820px;
    z-index: 1;
  }

  .landing-motto {
    font-size: clamp(3rem, 8vw, 6rem);
    font-weight: 900;
    margin-bottom: 1.5rem;
    letter-spacing: -0.02em;
    background: linear-gradient(135deg, var(--landing-teal-light) 0%, var(--landing-teal-medium) 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    animation: landing-fade-in-up 0.8s ease-out;
  }

  .landing-tagline {
    font-size: clamp(1.2rem, 3vw, 1.6rem);
    color: var(--landing-slate-lighter);
    margin-bottom: 3rem;
    line-height: 1.8;
    animation: landing-fade-in-up 0.9s ease-out;
  }

  .landing-cta-group {
    display: flex;
    gap: 1rem;
    justify-content: center;
    margin-bottom: 4rem;
  }

  .landing-cta {
    padding: 0.85rem 1.8rem;
    border-radius: 999px;
    font-weight: 600;
    text-decoration: none;
    transition: transform 0.25s ease, box-shadow 0.25s ease;
  }

  .landing-cta.primary {
    background: var(--landing-teal-medium);
    color: white;
  }

  .landing-cta.secondary {
    background: rgba(255, 255, 255, 0.12);
    color: white;
    border: 1px solid rgba(255, 255, 255, 0.2);
  }

  .landing-cta:hover {
    transform: translateY(-3px);
    box-shadow: 0 15px 35px rgba(15, 23, 42, 0.25);
  }

  .landing-section {
    max-width: 1180px;
    margin: 4rem auto 0;
    padding: 0 1.5rem 5rem;
  }

  .landing-section-title {
    font-size: clamp(2rem, 4vw, 2.8rem);
    font-weight: 700;
    text-align: center;
    margin-bottom: 3rem;
    color: var(--landing-slate-dark);
  }

  .landing-category-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
    gap: 2rem;
    margin-bottom: 3rem;
  }

  .landing-category-card {
    background: #ffffff;
    border-radius: 16px;
    padding: 2.5rem;
    box-shadow: 0 12px 30px rgba(15, 23, 42, 0.08);
    border: 1px solid transparent;
    transition: transform 0.25s ease, box-shadow 0.25s ease, border-color 0.25s ease;
    text-decoration: none;
    color: inherit;
    display: block;
  }

  .landing-category-card:hover {
    transform: translateY(-8px);
    border-color: rgba(20, 184, 166, 0.4);
    box-shadow: 0 20px 40px rgba(15, 23, 42, 0.12);
  }

  .landing-category-icon {
    font-size: 3rem;
    margin-bottom: 1rem;
    display: block;
  }

  .landing-category-name {
    font-size: 1.5rem;
    font-weight: 700;
    margin-bottom: 0.5rem;
    color: var(--landing-slate-dark);
  }

  .landing-category-korean {
    font-size: 1rem;
    color: var(--landing-teal-medium);
    font-weight: 600;
    margin-bottom: 0.75rem;
    display: block;
  }

  .landing-category-description {
    color: var(--landing-slate-medium);
    font-size: 0.95rem;
    line-height: 1.6;
  }

  .landing-skills {
    background: linear-gradient(135deg, var(--landing-slate-dark) 0%, var(--landing-slate-bg) 100%);
    color: #ffffff;
    border-radius: 1.5rem;
    padding: 5rem 1.5rem;
  }

  .landing-skills .landing-section-title {
    color: #ffffff;
  }

  .landing-skills-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    justify-content: center;
    align-items: center;
  }

  .landing-skill-tag {
    background: rgba(20, 184, 166, 0.15);
    color: var(--landing-teal-light);
    padding: 0.75rem 1.5rem;
    border-radius: 999px;
    font-size: 0.95rem;
    font-weight: 600;
    border: 1px solid rgba(13, 148, 136, 0.65);
    transition: all 0.25s ease;
  }

  .landing-skill-tag:hover {
    background: var(--landing-teal-medium);
    color: #ffffff;
    transform: translateY(-2px);
  }

  .landing-footer {
    background: var(--landing-slate-bg);
    color: var(--landing-slate-light);
    text-align: center;
    padding: 4rem 1.5rem;
    border-radius: 1.5rem;
    margin-top: 4rem;
  }

  .landing-footer-title {
    font-size: 1.45rem;
    font-weight: 700;
    margin-bottom: 1.5rem;
    color: var(--landing-teal-medium);
  }

  .landing-contact-links {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    margin-bottom: 2rem;
    align-items: center;
  }

  .landing-contact-item {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    font-size: 1rem;
  }

  .landing-contact-item a {
    color: var(--landing-teal-light);
    text-decoration: none;
    transition: color 0.25s ease;
  }

  .landing-contact-item a:hover {
    color: var(--landing-teal-medium);
  }

  .landing-footer-motto {
    margin-top: 2rem;
    padding-top: 2rem;
    border-top: 1px solid rgba(148, 163, 184, 0.3);
    font-size: 0.92rem;
    color: var(--landing-slate-medium);
  }

  .landing-footer-motto strong {
    color: var(--landing-teal-medium);
    font-weight: 700;
  }

  @keyframes landing-fade-in-up {
    from {
      opacity: 0;
      transform: translateY(30px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  @media (max-width: 768px) {
    .landing-cta-group {
      flex-direction: column;
      width: 100%;
      padding: 0 1rem;
    }

    .landing-cta {
      width: 100%;
      text-align: center;
    }

    .landing-section {
      margin-top: 3rem;
      padding-bottom: 3.5rem;
    }

    .landing-category-grid {
      grid-template-columns: 1fr;
    }

    .landing-skill-tag {
      font-size: 0.85rem;
      padding: 0.6rem 1.25rem;
    }
  }

  @media (min-width: 768px) and (max-width: 1024px) {
    .landing-category-grid {
      grid-template-columns: repeat(2, 1fr);
    }
  }
</style>

<div class="landing-page">
  <section class="landing-hero">
    <div class="landing-hero-content">
      <h1 class="landing-motto">Just Action</h1>
      <p class="landing-tagline">능동적으로 문제를 발견하고, 더 나은 해결책을 고민하는 개발자</p>
      <div class="landing-cta-group">
        <a class="landing-cta primary" href="/learning-log/">시작하기</a>
        <a class="landing-cta secondary" href="/playground/">데모 보기</a>
      </div>
    </div>
  </section>

  <section class="landing-section" id="landing-explore">
    <h2 class="landing-section-title">Explore</h2>
    <div class="landing-category-grid">
      <a href="/learning-log/" class="landing-category-card">
        <span class="landing-category-icon">📚</span>
        <h3 class="landing-category-name">Learning Log</h3>
        <span class="landing-category-korean">러닝 로그</span>
        <p class="landing-category-description">공부한 내용을 정리하는 공간</p>
      </a>

      <a href="/debug-notes/" class="landing-category-card">
        <span class="landing-category-icon">🛠️</span>
        <h3 class="landing-category-name">Debug Note</h3>
        <span class="landing-category-korean">디버그 노트</span>
        <p class="landing-category-description">개발 트러블슈팅 과정을 아카이브합니다.</p>
      </a>

      <a href="/playground/" class="landing-category-card">
        <span class="landing-category-icon">🎮</span>
        <h3 class="landing-category-name">Playground</h3>
        <span class="landing-category-korean">플레이그라운드</span>
        <p class="landing-category-description">실험용 데모와 프로토타입을 정리합니다.</p>
      </a>

      <a href="/about/" class="landing-category-card">
        <span class="landing-category-icon">👨‍💻</span>
        <h3 class="landing-category-name">About</h3>
        <span class="landing-category-korean">소개</span>
        <p class="landing-category-description">작티의 백그라운드와 협업 방식에 대해 알아보세요.</p>
      </a>
    </div>
  </section>

  <section class="landing-section">
    <div class="landing-skills">
      <h2 class="landing-section-title">Tech Stack &amp; Skills</h2>
      <div class="landing-skills-grid">
        <span class="landing-skill-tag">TypeScript</span>
        <span class="landing-skill-tag">JavaScript</span>
        <span class="landing-skill-tag">Python</span>
        <span class="landing-skill-tag">C</span>
        <span class="landing-skill-tag">Flutter</span>
        <span class="landing-skill-tag">React</span>
        <span class="landing-skill-tag">Node.js</span>
        <span class="landing-skill-tag">Firebase</span>
        <span class="landing-skill-tag">GCP</span>
        <span class="landing-skill-tag">Amazon Bedrock</span>
        <span class="landing-skill-tag">GSAP</span>
        <span class="landing-skill-tag">LangChain</span>
      </div>
    </div>
  </section>

  <footer class="landing-section landing-footer">
    <div class="landing-footer-content">
      <h3 class="landing-footer-title">Get In Touch</h3>
      <div class="landing-contact-links">
        <div class="landing-contact-item">
          <span class="landing-contact-icon">📬</span>
          <a href="mailto:hello@girae.me">hello@girae.me</a>
        </div>
        <div class="landing-contact-item">
          <span class="landing-contact-icon">🐙</span>
          <a href="https://github.com/jacti" target="_blank" rel="noopener">github.com/jacti</a>
        </div>
      </div>
      <p class="landing-footer-motto"><strong>Just Action</strong> — 도전하고 실험하며 배우는 과정을 기록합니다.</p>
    </div>
  </footer>
</div>
