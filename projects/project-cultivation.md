---
layout: default
title: Project Cultivation
---

<section class="project-hero" style="background-image: url('{{ 'assets/images/cultivate/title.png' | relative_url }}');">
  <div class="overlay">
    <h1>Project Cultivation</h1>
  </div>
</section>

<section class="project-details">
  <div class="project-info-grid">
    <div><strong>Project Type</strong><br>2D Platformer Shooter</div>
    <div><strong>Date</strong><br>June 2025</div>
    <div><strong>Status</strong><br>Prototype</div>
    <div><strong>Engine</strong><br>Unity Engine 6</div>
    <div><strong>Role</strong><br>Lead Programmer / Designer</div>
    <div><strong>Team Size</strong><br>4</div>
  </div>

  <p class="project-description">
    A 2D platformer shooter set in a military facility where research into using the ability to control time was done. You must now do your best to pass the trials set in front of you, or find some way to escape this cycle of test. <em>Project Cultivation</em> focuses on fighting against the system and taking back control.
  </p>

  <div class="media-gallery">
    <img src="{{ 'assets/images/cultivate/title.png' | relative_url }}" alt="Screenshot 1">
    <img src="{{ 'assets/images/cultivate/cultivategame1.gif' | relative_url }}" alt="Screenshot 2">
  </div>
</section>

<section class="project-section fade-in">
  <h2>Introduction</h2>
  <p>
    <em>Project Cultivation</em> was developed as a prototype for a game jam involving time. My team's idea was to create a shooter that involves controlling time and using that to solve puzzles. The major goal of our project was to create a prototype that included a shooting system that felt fun to use and showcased the time mechanic.
  </p>
  <p>
    Built in Unity, the main features this game has are a time control mechanic that allows you to pause or rewind time, and a flexible shooting system allowing players to aim and shoot freely at interactable objects.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Time Control System</h2>
  <p>
    In <em>Project Cultivation</em>, I implemented a time stop mechanic that allows players to freeze the world — halting physics, movement, and other dynamic systems — while keeping the player character free to act. To make this possible, I created a flexible interface-driven system using the ITimeStoppable interface and this TimeStopObject component.
  </p>
  <p>
    Each object that should react to a time freeze simply implements ITimeStoppable and registers itself with the TimeStopManager. This makes the system scalable — any new enemy, projectile, or physics object can easily join the time system without modifying core logic.
  </p>

  <p>
    The TimeStopObject script handles physics objects by saving their velocity and stopping their motion during a time stop, then restoring it when time resumes:
  </p>

  {% highlight csharp %}
using UnityEngine;

public class TimeStopObject : MonoBehaviour, ITimeStoppable
{
    public Rigidbody rb;
    private Vector3 savedVelocity;
    [SerializeField] private bool isStopped = false;

    private void OnEnable()
    {
        TimeStopManager.Instance?.Register(this);
    }

    private void OnDisable()
    {
        TimeStopManager.Instance?.Unregister(this);
    }

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    public void OnTimeResume()
    {
        if (!isStopped) return;

        rb.isKinematic = false;
        rb.linearVelocity = savedVelocity;
        isStopped = false;
    }

    public void OnTimeStop()
    {
        if (isStopped) return;

        savedVelocity = rb.linearVelocity;
        rb.linearVelocity = Vector3.zero;
        rb.isKinematic = true;
        isStopped = true;
    }
}
  {% endhighlight %}

  <div class="gif-container fade-in">
    <img src="{{ 'assets/images/projectcultivation/combat.gif' | relative_url }}" alt="Combat Demo">
  </div>
</section>

<section class="project-section fade-in">
  <h2>World Growth Mechanic</h2>
  <p>
    The game’s defining feature, the <strong>Growth System</strong>, allows the environment 
    to evolve dynamically in response to player performance. Destroying enemies releases 
    “Seed Data” — fragments of code that reconstruct the digital flora of the world. 
    As the player gathers more, they unlock new growth zones, platforms, and secret areas.
  </p>

  <p>
    This system is both visual and mechanical. It uses procedural mesh generation and shader-based 
    morphing to let players watch the world heal in real-time — roots growing through pixels, 
    vines forming new paths, and light piercing through the corruption.
  </p>

  <div class="gif-container fade-in">
    <img src="{{ 'assets/images/projectcultivation/growth.gif' | relative_url }}" alt="World Growth System">
  </div>
</section>

<section class="project-section fade-in">
  <h2>Development Notes</h2>
  <p>
    <em>Project Cultivation</em> is developed in Unity using a modular state-driven architecture 
    similar to the system I created for <em>Vampiric Ascension</em>. 
    Each subsystem — movement, shooting, enemy AI, and growth — is built to be independent yet reactive, 
    allowing new mechanics to be layered in without breaking core systems.
  </p>
  <p>
    The visual direction draws on glitch art and neon minimalism, 
    supported by a blue-heavy color palette and chromatic aberration effects. 
    Every aspect of the game is designed to evoke a sense of digital decay and rebirth.
  </p>
</section>

<style>
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
  }


/* Code block styling */
pre {
    background-color: #202020ff;
    /* Dark grey background */
    color: #f8f8f2;
    /* Light text */
    border-radius: 8px;
    padding: 1rem;
    font-family: 'JetBrains Mono', 'Fira Code', monospace;
    font-size: 0.9rem;
    overflow-x: auto;
    /* Allow horizontal scrolling */
    white-space: pre;
    /* Preserve spacing */
    display: block;
    box-shadow: 0 2px 10px #202020ff;
    margin: 1.5rem 0;
}

/* Code inside pre inherits color and layout */
pre code {
    background: none;
    color: inherit;
    font-family: inherit;
    font-size: inherit;
}
</style>
