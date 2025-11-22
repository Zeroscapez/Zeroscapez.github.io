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
    <div><strong>itch.io Page</strong><br><a href="https://crestoriashiro.itch.io/project-cultivation" target="_blank">Play Project Cultivation</a></div>
    <div><strong>Github</strong><br><a href="https://github.com/Zeroscapez/Project_Cultivation" target="_blank">View Project</a></div>
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
  <h2>Time Stop System</h2>
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
    <img src="{{ 'assets/images/cultivate/timestopgameplay.gif' | relative_url }}" alt="Combat Demo">
  </div>
</section>

<section class="project-section fade-in">
  <h2>Rewind Mechanic & Rewind Ghost</h2>
  <p>
    The <strong>Rewind Mechanic</strong> in <em>Project Cultivation</em> was designed to complement the Time Stop system by giving players the ability to reverse their recent actions — restoring their position to a point in the past while maintaining gameplay flow. 
    This mechanic records the player’s position over time and lets them “snap back” to a previous moment using the <code>Rewind()</code> function.
  </p>

  <p>
    When activated, the system finds the closest recorded position to the target rewind time, updates the player’s position, and clears old history to begin tracking anew. 
    This ensures precise, frame-accurate rewinds without creating desynchronization or unpredictable movement.
  </p>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
void Rewind()
{
    float targetTime = Time.time - rewindTime;
    PositionRecord rewindRecord = positionHistory[0];

    foreach (var record in positionHistory)
    {
        if (Mathf.Abs(record.time - targetTime) < Mathf.Abs(rewindRecord.time - targetTime))
        {
            rewindRecord = record;
        }
    }

    transform.position = rewindRecord.position;
    rb.linearVelocity = Vector3.zero;

    // Clear old trail & start fresh
    positionHistory.Clear();
    positionHistory.Add(new PositionRecord { position = transform.position, time = Time.time });
}
    </code></pre>
  </div>
<div class="gif-container fade-in">
    <img src="{{ 'assets/images/cultivate/rewindgameplay.gif' | relative_url }}" alt="Combat Demo">
  </div>
  <p>
    To help players visualize their past path, I implemented the <strong>Rewind Ghost</strong> — a transparent echo of the player’s previous self. 
    The ghost updates its position to match the historical record of where the player was at the rewind target time. 
    This adds both visual clarity and an immersive time-travel aesthetic to the mechanic.
  </p>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
void UpdateRewindGhost()
{
    if (positionHistory.Count == 0 || rewindGhost == null) return;

    float targetTime = Time.time - rewindTime;
    PositionRecord closest = positionHistory[0];

    foreach (var record in positionHistory)
    {
        if (Mathf.Abs(record.time - targetTime) < Mathf.Abs(closest.time - targetTime))
        {
            closest = record;
        }
    }

    rewindGhost.position = closest.position;
}
    </code></pre>
  </div>

  <div class="gif-container fade-in">
    <div class="gif-container fade-in">
    <img src="{{ 'assets/images/cultivate/timedoublegameplay.gif' | relative_url }}" alt="Combat Demo">
  </div>
  </div>
</section>


<section class="project-section fade-in">
  <h2>Slow Time Mechanic</h2>
  <p>
    In <em>Project Cultivation</em>, the Fast Forward mechanic works as a global time manipulator that temporarily slows the entire game world while maintaining smooth control responsiveness. This creates the illusion that the player is moving faster than everything else — enemies, projectiles, and physics all slow down, while the player experiences precise, high-speed mobility.

  </p>
  <p>
    The <code>WorldSlowdownManager</code> handles all timing adjustments using Unity’s Time.timeScale system. When activated, it multiplies time by a slowFactor, effectively reducing how quickly everything else updates. This slowdown persists for a few seconds (slowdownLength) before time returns to normal. To prevent abuse, a cooldown timer enforces a delay before the ability can be used again.
  </p>


  <div class="code-block fade-in">
    <pre><code class="language-csharp">
  using UnityEngine;

public class WorldSlowdownManager : MonoBehaviour
{
    public static WorldSlowdownManager Instance;

    public float slowFactor = 0.2f;
    public float slowdownLength = 3f;
    public float cooldown = 5f;

    private float slowEndTime;
    public float lastSlowTime = Mathf.NegativeInfinity;
    private bool isSlowing = false;

    public float CooldownRemaining
    {
        get
        {
            // Only start tracking cooldown *after* slow ends
            if (isSlowing) return cooldown;

            float cooldownEnd = lastSlowTime + cooldown;
            float remaining = cooldownEnd - Time.unscaledTime;
            return Mathf.Max(0f, remaining);
        }
    }
    void Awake()
    {
        if (Instance != null) Destroy(gameObject);
        Instance = this;
    }

    void Update()
    {
        if (isSlowing && Time.unscaledTime >= slowEndTime)
        {
            ResetTime();
        }

        UIController.Instance.restrictFastForward.SetActive(isSlowing || CooldownRemaining > 0);


    }


    public void TriggerSlowdown()
    {
        if (isSlowing || Time.unscaledTime < lastSlowTime + cooldown) return;

        Time.timeScale = slowFactor;
        Time.fixedDeltaTime = 0.02f * Time.timeScale;

        slowEndTime = Time.unscaledTime + slowdownLength;
        isSlowing = true;
    }


    private void ResetTime()
    {
        Time.timeScale = 1f;
        Time.fixedDeltaTime = 0.02f;
        isSlowing = false;
        lastSlowTime = Time.unscaledTime; 
    }

    public bool IsSlowing => isSlowing;
}
</code></pre>
  </div>

<div class="gif-container fade-in">
    <img src="{{ 'assets/images/cultivate/slowdown.gif' | relative_url }}" alt="Combat Demo">
  </div>

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
     background: #1a1a1a;
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
  
    margin: 1.5rem 0;
}

/* Code inside pre inherits color and layout */
pre code {
    background: none;
    color: inherit;
    font-family: inherit;
    font-size: inherit;
}

.gif-container img {
   
    box-shadow: 0 0 20px rgba(74, 178, 230, 0.81);
   
}

.gif-container img:hover {
    transform: scale(1.03);
    box-shadow: 0 0 25px rgba(81, 197, 255, 1);
}
</style>
