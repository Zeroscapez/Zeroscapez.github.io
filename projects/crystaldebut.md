---
layout: default
title: Crystal's Debut
accent: "#ff6ec7"
---

<section class="project-hero" style="background-image: url('{{ 'assets/images/crystaldebut/MainMenu.webp' | relative_url }}');">
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
    <div><strong>Project Type</strong><br>Pirate Jam 18 · Post-Jam Demo</div>
    <div><strong>Date</strong><br>Jan 2026 – Present</div>
    <div><strong>Status</strong><br>In Development</div>
    <div><strong>itch.io Page</strong><br><a href="https://crestoriashiro.itch.io/eldritch-vtuber" target="_blank">Play Crystal's Debut Demo</a></div>
    <div><strong>Engine</strong><br>Unity Engine</div>
    <div><strong>Role</strong><br>Project Manager, Lead Programmer, Lead Game Designer</div>
    <div><strong>Team Size</strong><br>5</div>
  </div>

  <p class="project-description">
    You play as Crystal, a newly hired streamer, working through chat's requests in real time to keep your audience devoted. Requests play out on Crystal's in-game computer — minigames, apps, messages from contacts, and messing with her pet gerbil Kevin — while something stranger stirs underneath the stream.
  </p>

  <div class="media-gallery">
    <img src="{{ 'assets/images/crystaldebut/CrystalDebut1.jpg' | relative_url }}" alt="Crystal's Debut screenshot 1" loading="lazy">
    <img src="{{ 'assets/images/crystaldebut/CrystalDebut2.jpg' | relative_url }}" alt="Crystal's Debut screenshot 2" loading="lazy">
    <img src="{{ 'assets/images/crystaldebut/CrystalDebut3.jpg' | relative_url }}" alt="Crystal's Debut screenshot 3" loading="lazy">
    <img src="{{ 'assets/images/crystaldebut/CrystalDebut4.jpg' | relative_url }}" alt="Crystal's Debut screenshot 4" loading="lazy">
    <img src="{{ 'assets/images/crystaldebut/MainMenu.webp' | relative_url }}" alt="Crystal's Debut main menu" width="1920" height="1080" loading="lazy">
  </div>
</section>

<section class="project-section fade-in">
  <h2>Download &amp; Play</h2>
  <iframe class="itch-widget" src="https://itch.io/embed/4238606?linkback=true&amp;dark=true" title="Crystal's Debut on itch.io" loading="lazy"><a href="https://crestoriashiro.itch.io/eldritch-vtuber">Crystal's Debut on itch.io</a></iframe>
  <p class="embed-note">Windows download via itch.io.</p>
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
  <p>
    After winning the jam, we kept going. The jam prototype grew into a fuller demo: a working desktop with its own programs, a contacts-and-messaging app, three more minigames, a tutorial, save/load, adaptive audio through FMOD, and the Devotion system that now drives the stream.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Designing the Stream: Requests and Devotion</h2>
  <p>
    Every request is authored as data, not code: a <code>RequestData</code> asset holds the viewer's name, a time limit, its rewards (viewers, points, subscribers, Devotion), the Devotion lost if it expires, and the dialogue nodes to play when it starts, completes, or runs out.
  </p>
  <p>
    The design tool I'm happiest with is the <strong>conflicting request</strong>. A request can name a rival request, and finishing one fails the other. Two viewers can ask for opposite things at the same time, and there's no way to please both — the player has to decide whose devotion matters more.
  </p>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
// Inside FinalizeRequest(), after the rewards are paid out
if (request.Data.ConflictingRequest != null)
{
    _requestsById.TryGetValue(request.Data.ConflictingRequest.RequestID, out var conflicting);

    if (conflicting != null &amp;&amp; conflicting.Status == RequestStatus.Active)
        FinalizeRequest(conflicting, RequestStatus.Failed);
}
    </code></pre>
  </div>

  <p>
    Devotion is the stream's health. It climbs through five tiers — Indifferent, Interested, Invested, Devoted, and Fanatical — but how high it can go in a single stream is capped by a longer-term karma value that carries across streams. One great stream can't make an audience Fanatical; that takes trust built up over time, so the player's past choices keep mattering.
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
    By the demo, that command layer had grown to 19 custom commands, and the Yarn scripts direct almost the whole game: starting the stream and its requests (<code>start_stream</code>, <code>start_request</code>, <code>complete_request</code>), moving Devotion (<code>add_devotion</code>, <code>lose_devotion</code>), opening programs on the desktop (<code>open_program</code>), unlocking contacts (<code>unlock_contact</code>), cueing music and sound (<code>play_bgm</code>, <code>play_sfx</code>), and walking new players through the interface with a tutorial pointer (<code>show_pointer</code>, <code>wait_for_pointer</code>).
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
    <video class="lazy-video" data-src="{{ 'assets/video/crystaldebut/chat.mp4' | relative_url }}" poster="{{ 'assets/video/crystaldebut/chat.webp' | relative_url }}" width="1280" height="714" muted loop playsinline preload="none" aria-label="In-game messaging app demo"></video>
  </div>
</section>

<section class="project-section fade-in">
  <h2>An In-Game Desktop</h2>
  <p>
    Most of the game happens on Crystal's computer, so it works like one: a start menu, a taskbar, desktop icons and folders, and programs that open in their own windows. Alongside the browser and <em>Frogicord</em>, her messaging app, the desktop hosts the game's minigames — <em>Wordify</em>, a Memory game, and <em>Molewhac</em>, a whac-a-mole built with teammate Damien. Programs can be opened by the player or by the script itself, so a request can put exactly the right window in front of the player at the right moment.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Wordle Minigame Breakdown</h2>
  <p>
    One of the minigames embedded in <em>Crystal's Debut</em> is <em>Wordify</em>, a fully functional Wordle
    clone with its own on-screen keyboard, playable directly on Crystal's in-game computer. The game pulls from two word
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
    <video class="lazy-video" data-src="{{ 'assets/video/crystaldebut/wordle.mp4' | relative_url }}" poster="{{ 'assets/video/crystaldebut/wordle.webp' | relative_url }}" width="1278" height="722" muted loop playsinline preload="none" aria-label="Wordle minigame demo"></video>
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

<section class="project-section fade-in">
  <h2>Tooling: The Debug Overlay</h2>
  <p>
    A game run by script and state needs a fast way to jump to any situation, so I built an in-game debug overlay the whole team could use. It shows the live stream (viewers, points, subscribers, Devotion and its tier), the installed programs, Yarn variables, every request and its status, and the player's contacts — and each section has controls to change them: start or end the stream, add or remove viewers and Devotion, open any program, and save or load the game. Testing a late-game request no longer meant replaying the whole stream to get there.
  </p>
</section>
