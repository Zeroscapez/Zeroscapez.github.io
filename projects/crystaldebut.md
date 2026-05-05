---
layout: default
title: Crystal's Debut
---

<section class="project-hero" style="background-image: url('{{ 'assets/images/crystaldebut/MainMenu.png' | relative_url }}');">
  <div class="overlay">
    <h1>Crystal's Debut</h1>
  </div>
</section>

<div class="winner-banner">
  <div class="winner-trophy">
    <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
      <path d="M12 2C9.24 2 7 4.24 7 7v4c0 2.61 1.67 4.83 4 5.65V18H9v2h6v-2h-2v-1.35C15.33 15.83 17 13.61 17 11V7c0-2.76-2.24-5-5-5zm3 9c0 1.65-1.35 3-3 3s-3-1.35-3-3V7c0-1.65 1.35-3 3-3s3 1.35 3 3v4zM5 7H3v4c0 1.86.8 3.53 2.08 4.71L6.5 14.3C5.57 13.42 5 12.18 5 11V7zm14 0v4c0 1.18-.57 2.42-1.5 3.3l1.42 1.41C20.2 14.53 21 12.86 21 11V7h-2z"/>
    </svg>
  </div>
  <div class="winner-text">
    <h1 class="winner-headline">Winner — Pirate Jam 18</h1>
   
  </div>
</div>

<section class="project-details">
  <div class="project-info-grid">
    <div><strong>Project Type</strong><br>Pirate Jam 18</div>
    <div><strong>Date</strong><br>Jan 2026</div>
    <div><strong>Status</strong><br>In Development</div>
    <div><strong>itch.io Page</strong><br><a href="https://crestoriashiro.itch.io/eldritch-vtuber" target="_blank">Play Crystal's Debut Demo</a></div>
    <div><strong>Engine</strong><br>Unity Engine</div>
    <div><strong>Roles</strong><br>Project Manager, Lead Programmer, Lead Game Designer</div>
  </div>

  <h4 class="project-description">
 The player plays as Crystal, a newly hired streamer and follows requests from chat. Requests include interacting with the computer on the screen to complete actions to raise her approval meter. The actions range from minigames, interacting with applications on the computer, to messing with Crystal’s pet Gerbil Kevin.
  </h4>

  <div class="media-gallery">
    <img src="{{ 'assets/images/crystaldebut/CrystalDebut1.jpg' | relative_url }}" alt="Screenshot 1">
    <img src="{{ 'assets/images/crystaldebut/CrystalDebut2.jpg' | relative_url }}" alt="Screenshot 1">
    <img src="{{ 'assets/images/crystaldebut/CrystalDebut3.jpg' | relative_url }}" alt="Screenshot 2">
    <img src="{{ 'assets/images/crystaldebut/CrystalDebut4.jpg' | relative_url }}" alt="Screenshot 2">
    <img src="{{ 'assets/images/crystaldebut/MainMenu.png' | relative_url }}" alt="Screenshot 3">
  </div>
</section>

<section class="project-section fade-in">
  <h2>Introduction</h2>
  <p>
    <em>Crystal's Debut</em> was developed for Pirate Jam 18, drawing inspiration from 
    <em>Needy Streamer Overload</em> and a shared passion for VTuber culture. The project 
    blends charming minigames and endearing characters with an undercurrent of corporate 
    pressure and eldritch interference — a deliberate contrast that gives the game its tone.
  </p>
  <p>
    The goal was to create something mechanically approachable but narratively layered, 
    where every viewer request feels consequential and Crystal's world slowly reveals itself 
    to be stranger than it first appears.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Dialogue & Request System — Yarn Spinner 3</h2>
  <p>
    To handle Crystal's chat requests and main dialogue, I integrated 
    <a href="https://yarnspinner.dev" target="_blank">Yarn Spinner 3</a> — a narrative 
    scripting tool built for Unity that allows dialogue to be written in a clean, 
    readable format and driven at runtime without hardcoding conversation logic into C#.
  </p>
  <p>
    The primary challenge was learning Yarn Spinner's node and command architecture from 
    scratch and designing a system where dialogue could trigger game actions seamlessly. 
    To bridge the gap between narrative and gameplay, I wrote a suite of custom C# commands 
    registered with Yarn Spinner's command dispatcher. These commands allowed Yarn scripts 
    to directly invoke request logic — launching minigames, updating the approval meter, 
    and interacting with in-game objects — without any manual intervention from other systems.
  </p>
  <p>
    This approach kept all request sequencing and branching inside the Yarn scripts 
    themselves, making it straightforward to add, reorder, or adjust requests without 
    touching the underlying C# codebase.
  </p>

  <p>
  A key design challenge was distinguishing between Crystal's main dialogue and messages 
  delivered through her in-game messaging app. Rather than building a separate system, 
  I extended the existing Yarn Spinner integration by tagging specific lines with 
  <code>#message</code> directly in the Yarn scripts. A custom dialogue presenter listens 
  for this tag at runtime and routes those lines into the messaging app UI instead of the 
  standard dialogue display — keeping all conversation logic in one place while allowing 
  the presentation layer to vary depending on context.
</p>

  <div class="gif-container fade-in">
    <img src="/assets/images/crystaldebut/chat.gif" alt="Graze mechanic demo" />
  </div>

  
 

 
</section>

<section class="project-section fade-in">
  <h2>Wordle Minigame Breakdown</h2>
  <p>
    One of the minigames embedded in <em>Crystal's Debut</em> is a fully functional Wordle 
    clone, playable directly on Crystal's in-game computer. The game pulls from two word 
    lists loaded at runtime — a common solutions list and a broader valid words list — 
    and selects a random target word each time the minigame is triggered.
  </p>
  <p>
    The trickiest part of the implementation was the two-pass letter evaluation system in 
    <code>OnRowSubmit()</code>. A naive single-pass approach produces incorrect results when 
    the guess contains duplicate letters — for example, guessing "APPLE" when the answer is 
    "CRANE" could incorrectly mark both P's as wrong-spot rather than incorrect. To handle 
    this accurately, the evaluation runs in two passes. The first pass checks for exact 
    matches, marking any correct letters and removing them from a tracked 
    <code>remaining</code> string. The second pass then checks the leftover letters for 
    wrong-spot or incorrect states, consuming each match from <code>remaining</code> as it 
    goes. This ensures duplicate letters are accounted for precisely and never double-counted.
  </p>

  <div class="gif-container fade-in">
    <img src="/assets/images/crystaldebut/wordle.gif" alt="Wordle minigame demo" />
  </div>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
// Pass 1 — exact matches
for (int i = 0; i &lt; row.tiles.Length; i++)
{
    WordleTile tile = row.tiles[i];

    if (tile.letter == targetWord[i])
    {
        tile.SetState(correctState);
        remaining = remaining.Remove(i, 1).Insert(i, " ");
    }
}

// Pass 2 — wrong spot or incorrect
for (int i = 0; i &lt; row.tiles.Length; i++)
{
    WordleTile tile = row.tiles[i];

    if (tile.state == correctState) continue;

    if (remaining.Contains(tile.letter))
    {
        tile.SetState(wrongSpotState);
        int index = remaining.IndexOf(tile.letter);
        remaining = remaining.Remove(index, 1).Insert(index, " ");
    }
    else
    {
        tile.SetState(incorrectState);
    }
}
    </code></pre>
  </div>

  
</section>


<style>
  .winner-banner {
    display: flex;
    align-items: center;
    gap: 1rem;
    background: #FAEEDA;
    border: 1px solid #EF9F27;
    border-radius: 10px;
    padding: 1rem 1.25rem;
    margin: 1.5rem 0 2rem;
  }
  .winner-trophy {
    width: 44px;
    height: 44px;
    background: #BA7517;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }
  .winner-trophy svg {
    width: 22px;
    height: 22px;
    fill: #FAEEDA;
  }
  .winner-headline {
    font-size: 30px;
    font-weight: 600;
    color: #633806;
    margin: 0 0 3px;
  }
  .winner-sub {
    font-size: 13px;
    color: #854F0B;
    margin: 0;
  }
  .project-section h2 {
    font-size: 1.6rem;
    font-weight: 700;
    color: #fff;
    border-bottom: 2px solid #4AB3F4;
    display: inline-block;
    margin-bottom: 1rem;
  }
  .gif-container img {
    width: 80%;
    max-width: 800px;
    border-radius: 8px;
    display: block;
    margin: 1.5rem auto;
    box-shadow: 0 0 20px rgba(160, 31, 153, 0.81);
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }
  .gif-container img:hover {
    transform: scale(1.03);
    box-shadow: 0 0 25px rgb(255, 104, 217);
  }
  pre {
    background: #1a1a1a;
    color: #f8f8f2;
    border-radius: 8px;
    padding: 1rem;
    font-family: 'JetBrains Mono', 'Fira Code', monospace;
    font-size: 0.9rem;
    overflow-x: auto;
    white-space: pre-wrap;
    word-break: break-word;
    margin: 1.5rem 0;
  }
  pre code {
    background: none;
    color: inherit;
    font-family: inherit;
    font-size: inherit;
  }
</style>


