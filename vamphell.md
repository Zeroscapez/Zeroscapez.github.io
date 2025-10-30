---
layout: default
title: Vampiric Ascension
permalink: /vamphell/
---
_Bullet Hell Jam submission — a tense, fast-paced bullet hell where a sleepy vampire faces down gods._


# Overview
Vampiric Ascension is a Unity-built bullet hell inspired by Touhou that focuses on tight player control, readable-but-challenging projectile patterns, and scalable difficulty. Design goals were: responsive input, performant projectile systems, and modular pattern composition so new enemy/boss patterns could be authored quickly.

<!-- Hero banner (static) -->
<img src="/assets/images/vamphell/NHhPov.png" alt="Vampiric Ascension Banner" style="width:10%; max-width:720px; border-radius:12px; display:block; margin:1.25em auto; box-shadow:0 6px 18px rgba(0,0,0,0.6)">



---

## My role
Primary designer & developer:
- Gameplay programming (player movement, input buffering, combat feel)  
- Bullet pattern system and object pooling  
- Wave & boss encounter scripting and progression  
- UI / HUD and polish (VFX, particle FX, camera shake)  
- Playtest-driven tuning and balancing

---

## Key features
- Dynamic, modular bullet patterns (bursts, spirals, shells) with difficulty scaling  
- Player resource system (vampiric energy) that gates special abilities and adds decision-making  
- Procedural wave spawner with configurable pacing and difficulty curves  
- Optimized projectiles via object pooling for stable performance under heavy spawn load  
- Clean, readable visual language so patterns are learnable but challenging

---

## Build / Play (demo & downloads)
<!-- itch.io embed (replace URL / width/height as needed) -->
<div style="margin:1em 0;">
  <h3 style="margin-bottom:0.25em;">Play the Web Demo</h3>
  <!-- Replace https://your-itch-page.itch.io/vampiric-ascension with your actual itch.io page -->
  <iframe frameborder="0" src="https://crestoriashiro.itch.io/vampiric-pantheon" width="600" height="167" style="max-width:100%;"></iframe>
  <p style="font-size:0.9em; color:#aaa; margin-top:0.5em;">If embed doesn't load, open the demo: <a href="https://crestoriashiro.itch.io/vampiric-pantheon" target="_blank" rel="noopener">Vampiric Ascension on itch.io</a></p>
</div>


---

## Technical summary
- Engine: Unity 6
- Language: C#  
- Core systems: State Machines, Object Pooling, Procedural Wave Spawner, Scriptable pattern data  
- Tools: Unity Editor Tools, Git (GitHub)

Example systems:
- BulletSpawner: supports radial, spiral, burst, and aimed modes; pattern data stored in ScriptableObjects  
- WaveManager: schedules enemies and boss phases using configurable time/events  
- PoolManager: global object pool for bullets and small enemies

---

## Screenshots / GIFs
<!-- replace with actual assets in /assets/images/vamphell/ -->
<img src="/assets/images/vamphell/screen1.png" alt="Vampiric Ascension - gameplay 1" style="width:48%; max-width:480px; margin:10px 1%; border-radius:8px; display:inline-block;">
<img src="/assets/images/vamphell/screen2.png" alt="Vampiric Ascension - boss" style="width:48%; max-width:480px; margin:10px 1%; border-radius:8px; display:inline-block;">

---



## Challenges & solutions
- Performance under heavy projectile load → implemented object pooling and minimized per-frame allocations.  
- Pattern readability vs. difficulty → iterated on color, size, and timing so players can react while still being challenged.  
- Rapid iteration → used ScriptableObject-based pattern data so designers could author new patterns without code edits.

---


## Links & contact
- Source / repo: (link to GitHub project or private repo)  
- Build / demo: (link to itch.io or WebGL build)  
- Contact: <a href="mailto:agyeilomini@gmail.com">agyeilomini@gmail.com</a>

