---
layout: default
title: Greatest Shinobi
accent: "#5ecb8f"
---

<section class="project-hero" style="background-image: url('{{ 'assets/images/gshinobi/title.png' | relative_url }}');">
  <div class="overlay">
    <h1>Greatest Shinobi</h1>
  </div>
</section>

<section class="project-details">
  <div class="project-info-grid">
    <div><strong>Project Type</strong><br>Pirate Jam 15</div>
    <div><strong>Date</strong><br>July 2024</div>
    <div><strong>Status</strong><br>Complete</div>
    <div><strong>itch.io Page</strong><br><a href="https://crestoriashiro.itch.io/greatest-shinobi" target="_blank">Play Greatest Shinobi</a></div>
    <div><strong>Engine</strong><br>Unity Engine</div>
    <div><strong>Role</strong><br>Project Manager, Lead Gameplay Programmer, Designer</div>
    <div><strong>Team Size</strong><br>5</div>
  </div>

  <p class="project-description">
    A 2.5D platformer where you transform into a frog to sneak, hop, and slap your way to becoming the greatest shinobi.
  </p>

  <div class="media-gallery">
    <img src="{{ 'assets/images/gshinobi/title.png' | relative_url }}" alt="Greatest Shinobi title screen">
    <img src="{{ 'assets/images/gshinobi/gs1.jpg' | relative_url }}" alt="Greatest Shinobi screenshot 1">
    <img src="{{ 'assets/images/gshinobi/gs2.jpg' | relative_url }}" alt="Greatest Shinobi screenshot 2">
    <img src="{{ 'assets/images/gshinobi/gs3.jpg' | relative_url }}" alt="Greatest Shinobi screenshot 3">
  </div>
</section>

<section class="project-section fade-in">
  <h2>Introduction</h2>
  <p>
    <em>Greatest Shinobi</em> was built in about two weeks for Pirate Jam 15 with a team of five. The pitch was simple: a 2.5D platformer where you play as an aspiring shinobi who can transform into a frog, trading combat strength for mobility and stealth to make your way through each level.
  </p>
  <p>
    As lead gameplay programmer, I handled the core systems that made the concept work: player and enemy movement, combat, enemy AI, camera setup, animation controllers, and the frog transformation system itself.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Frog Transformation System</h2>
  <p>
    The transformation is the game's central gimmick, so I built a single <code>FormManager</code> to act as the source of truth for which form the player is in. Every frame it pushes the current form into the Animator via a <code>Frog</code> bool and enables or disables the <code>SlapAttack</code> component accordingly &mdash; frog form is built around agility and evasion, not combat, so attacking is locked out while transformed.
  </p>
  <p>
    Transforming is triggered by pickups in the world: a <code>FrogCoin</code> switches the player into frog form on contact, while a normal-form pickup (<code>GoobForm</code>) switches them back. Both use the same trigger-collider pattern &mdash; check for the player's tag, flip the shared <code>isfrog</code> flag on <code>FormManager</code>, and destroy the pickup &mdash; which kept the system easy to place and tune throughout the level design pass.
  </p>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
public class FormManager : MonoBehaviour
{
    public bool isfrog = false;
    public Animator animator;
    public SlapAttack attack;
    public PlayerMove move;

    void Update()
    {
        animator.SetBool("Frog", isfrog);
        attack.enabled = !isfrog;
    }
}

// FrogCoin.cs - pickup that flips the player into frog form
private void OnTriggerEnter(Collider other)
{
    if (other.CompareTag("Player"))
    {
        PlayerProps player = other.gameObject.GetComponent&lt;PlayerProps&gt;();
        if (player != null)
        {
            formManager.isfrog = true; // Become a Frog
        }

        Destroy(gameObject);
    }
}
    </code></pre>
  </div>
</section>

<section class="project-section fade-in">
  <h2>Combat &amp; Enemy AI</h2>
  <p>
    Combat is a melee hitbox system: <code>SlapAttack</code> listens for the attack input, plays the <code>Hit</code> animation, and sweeps an <code>OverlapSphere</code> at an attack point to find anything on the enemy layer, dealing damage to each hit.
  </p>
  <p>
    Enemies are defined with a data-driven approach &mdash; an <code>EnemyInfo</code> <code>ScriptableObject</code> holds health, move speed, and attack damage per enemy type, so new enemies could be authored as assets without touching code. Aggro is handled by pairing a trigger volume (<code>ChaseTrigger</code>) with a <code>NavMeshAgent</code>-driven <code>EnemyChase</code>: entering the trigger starts the chase and exiting stops it, while the chase logic manually faces the enemy toward the player and flips its sprite based on movement direction.
  </p>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
void Attack()
{
    Collider[] hitEnemies = Physics.OverlapSphere(attackPoint.position, attackRange, enemyLayers);
    animator.Play("Hit");

    foreach (Collider enemy in hitEnemies)
    {
        enemy.GetComponent&lt;Enemy&gt;().TakeDamage(attackDamage);
    }
}

// ChaseTrigger.cs - hands control of aggro state to nearby enemies
private void OnTriggerStay(Collider other)
{
    if (other.CompareTag("Player"))
        enemyChase.StartChase();
}

private void OnTriggerExit(Collider other)
{
    if (other.CompareTag("Player"))
        enemyChase.StopChase();
}
    </code></pre>
  </div>
</section>

<section class="project-section fade-in">
  <h2>Camera Setup</h2>
  <p>
    Levels use a Cinemachine virtual camera that automatically finds and follows whichever GameObject is tagged <code>Player</code> at runtime. Wiring the Follow target this way instead of hand-assigning it per scene meant any level could drop the camera rig in and have it work immediately &mdash; useful for a jam where levels were being built and iterated on in parallel by multiple people.
  </p>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
void Start()
{
    cinemachineCamera = GetComponent&lt;CinemachineVirtualCamera&gt;();
    GameObject player = GameObject.FindGameObjectWithTag("Player");
    if (player != null)
    {
        cinemachineCamera.Follow = player.transform;
    }
}
    </code></pre>
  </div>
</section>

<section class="project-section fade-in">
  <h2>Animation Controller</h2>
  <p>
    The player's Animator is a mirrored state machine: every human-form state (<code>Idle</code>, <code>Run</code>, <code>Jump</code>, <code>Victory</code>) has a frog-form counterpart (<code>FrogIdle</code>, <code>FrogRun</code>, <code>FrogJump</code>, <code>FrogVictory</code>), plus dedicated <code>Transform Frog</code> and <code>Transform Boy</code> states to handle the swap itself, and a <code>Hit</code> state layered in for combat feedback.
  </p>
  <p>
    All of it is driven by just three parameters &mdash; <code>Speed</code> and <code>Jumping</code> from <code>PlayerMove</code>, and <code>Frog</code> from <code>FormManager</code> &mdash; which kept the movement, combat, and transformation systems fully decoupled from the animation logic while still staying in sync.
  </p>
</section>
