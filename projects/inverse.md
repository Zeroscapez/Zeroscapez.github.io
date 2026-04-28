---
layout: default
title: Inverse
---

<section class="project-hero" style="background-image: url('{{ 'assets/images/inverse/title.png' | relative_url }}');">
  <div class="overlay">
    <h1>Inverse</h1>
  </div>
</section>

<section class="project-details">
  <div class="project-info-grid">
    <div><strong>Project Type</strong><br>3D Puzzle Platformer</div>
    <div><strong>Date</strong><br>April 2025</div>
    <div><strong>Status</strong><br>Prototype</div>
    <div><strong>itch.io Page</strong><br><a href="https://crestoriashiro.itch.io/inverse" target="_blank">Play Inverse</a></div>
    <div><strong>Github</strong><br><a href="https://github.com/Zeroscapez/ReflectionJam" target="_blank">View Project</a></div>
    <div><strong>Engine</strong><br>Unity Engine 6</div>
    <div><strong>Role</strong><br>Project Manager, Lead Programmer, Game Designer</div>
    <div><strong>Team Size</strong><br>5</div>
  </div>

  <p class="project-description">
    A 3D platformer focused on solving puzzles split between two worlds. Your goal is to solve the puzzles and reach the end in both worlds.
  </p>

  <div class="media-gallery">
    <img src="{{ 'assets/images/inverse/title.png' | relative_url }}" alt="Inverse title screen">
    <img src="{{ 'assets/images/inverse/inversegame1.gif' | relative_url }}" alt="Inverse gameplay">
  </div>
</section>

<section class="project-section fade-in">
  <h2>Introduction</h2>
  <p>
    <em>Inverse</em> was developed as a prototype for a game jam themed around reflections. My team's concept was to use literal reflections as a core mechanic while also exploring the idea of a reflection of oneself as part of the game's theme.
  </p>
  <p>
    Built in Unity, the game's main features are a character swapping system and a light bending mechanism.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Character Swap System</h2>
  <p>
    In <em>Inverse</em>, the player controls two fully independent characters and can switch between them at any time. Each character has its own movement controller, animations, stats, and physics body. The swap mechanic is designed not just for cosmetic variety, but to encourage puzzle-solving and traversal using each character's unique abilities.
  </p>
  <p>When the player swaps:</p>
  <ul>
    <li>The newly selected character becomes the only one receiving active movement input.</li>
    <li>Animations update automatically based on which character is currently active.</li>
    <li>Physics are toggled so the inactive character freezes in place using <code>isKinematic</code>.</li>
    <li>The camera dynamically refocuses using Cinemachine, smoothly shifting its Follow and LookAt targets.</li>
    <li>The inactive character remains present in the world, enabling cooperative platforming puzzles, positional switching, and multi-step traversal where character placement matters.</li>
  </ul>

  <div class="code-block fade-in">
  <pre>
  <code class="language-csharp">
using UnityEngine;

public class PlayerManager : MonoBehaviour
{
    private GameObject[] allCharacters;
    public GameObject characterA;
    public GameObject characterB;
    public GameObject activeCharacter;
    private CharacterController3D characterController;
    public PlayerControls controls;
    public IGUIManager guiManager;
    public bool isCharacterA;
    public Camera pCam;

    private void Awake()
    {
        controls = new PlayerControls();
        pCam = GetComponentInChildren&lt;Camera&gt;();

        if (guiManager == null)
        {
            guiManager = GetComponentInParent&lt;GameManager&gt;().GetComponentInChildren&lt;IGUIManager&gt;();
        }
    }

    private void Update()
    {
        if (guiManager == null)
        {
            guiManager = GetComponentInParent&lt;GameManager&gt;().GetComponentInChildren&lt;IGUIManager&gt;();
        }
    }

    void OnEnable() =&gt; controls.Enable();
    void OnDisable() =&gt; controls.Disable();

    void SwapCharacter()
    {
        Debug.Log(characterController.Name + &quot; &quot; + characterController.ID.ToString() + &quot; Character Swap Pressed&quot;);
        activeCharacter = (activeCharacter == characterA) ? characterB : characterA;
        SetActiveCharacter(activeCharacter);
        guiManager.SwitchCharacterImage();
    }

    void SetActiveCharacter(GameObject character)
    {
        characterA.GetComponent&lt;CharacterController3D&gt;().enabled = (character == characterA);
        characterB.GetComponent&lt;CharacterController3D&gt;().enabled = (character == characterB);
        characterA.GetComponentInChildren&lt;Animator&gt;().enabled = (character == characterA);
        characterB.GetComponentInChildren&lt;Animator&gt;().enabled = (character == characterB);
        characterA.GetComponent&lt;Rigidbody&gt;().isKinematic = (character != characterA);
        characterB.GetComponent&lt;Rigidbody&gt;().isKinematic = (character != characterB);
        pCam.GetComponent&lt;CinemachineFreeLook&gt;().Follow = activeCharacter.transform;
        pCam.GetComponent&lt;CinemachineFreeLook&gt;().LookAt = activeCharacter.transform;
    }

    public void SetupCharacters(CharacterController3D[] characterList)
    {
        allCharacters = characterList.Select(cc =&gt; cc.gameObject).ToArray();
        characterA = allCharacters.FirstOrDefault(go =&gt; go.GetComponent&lt;CharacterController3D&gt;().ID == 1);
        characterB = allCharacters.FirstOrDefault(go =&gt; go.GetComponent&lt;CharacterController3D&gt;().ID == 2);

        if (characterA != null &amp;&amp; characterB != null)
        {
            activeCharacter = characterA;
            SetActiveCharacter(activeCharacter);
            characterController = activeCharacter.GetComponent&lt;CharacterController3D&gt;();

            controls.Player.SwapCharacter.performed += ctx =&gt; SwapCharacter();
            pCam.GetComponent&lt;CinemachineFreeLook&gt;().Follow = activeCharacter.transform;
            pCam.GetComponent&lt;CinemachineFreeLook&gt;().LookAt = activeCharacter.transform;
        }
        else
        {
            Debug.LogError(&quot;Characters with specified IDs not found.&quot;);
        }
    }
}
  </code>
  </pre>
  </div>

  <div class="gif-container fade-in">
    <img src="{{ 'assets/images/inverse/swap.gif' | relative_url }}" alt="Character swap demo">
  </div>
</section>

<section class="project-section fade-in">
  <h2>Light Refraction System</h2>
  <p>
    In <em>Inverse</em>, light is used as a core puzzle mechanic. The player interacts with beams that bounce off reflective surfaces and activate switches throughout the environment. The light is dynamically rendered using a <code>LineRenderer</code>, allowing it to visually update in real time as it moves through the world.
  </p>
  <p>
    The beam begins at a defined origin point and travels forward until it hits an object. If the object is reflective, the beam bounces and continues in a new direction, enabling mirror-based puzzles and multi-step routing challenges. If the beam strikes a switch, it powers it on — allowing light to unlock doors or trigger mechanisms. Once the beam stops touching a switch, it automatically deactivates, preventing players from powering multiple switches simultaneously.
  </p>

   <div class="code-block fade-in">
  <pre>
  <code class="language-csharp">
using UnityEngine;

public class LightEmitter : MonoBehaviour
{
    public LineRenderer lineRenderer;
    public int maxBounces = 10;
    public Transform lightOrigin;
    public Material lightMaterial;
    public bool useCustomMaterial = true;

    private SwitchControl currentSwitch = null;

    void Start()
    {
        if (lineRenderer == null)
        {
            lineRenderer = gameObject.AddComponent&lt;LineRenderer&gt;();
        }

        lineRenderer.material = (useCustomMaterial &amp;&amp; lightMaterial != null)
            ? lightMaterial
            : new Material(Shader.Find(&quot;Sprites/Default&quot;));

        lineRenderer.positionCount = 0;
        lineRenderer.startWidth = 0.1f;
        lineRenderer.endWidth = 0.1f;
    }

    void Update()
    {
        CastLight(lightOrigin.position, -transform.up, maxBounces);
    }

    void CastLight(Vector3 position, Vector3 direction, int remainingBounces)
    {
        List&lt;Vector3&gt; lightPoints = new List&lt;Vector3&gt;();
        lightPoints.Add(position);

        SwitchControl hitSwitch = null;

        while (remainingBounces &gt; 0)
        {
            RaycastHit hit;
            if (Physics.Raycast(position, direction, out hit, Mathf.Infinity))
            {
                lightPoints.Add(hit.point);

                if (hit.collider.CompareTag(&quot;Reflective&quot;))
                {
                    ReflectiveSurface surface = hit.collider.GetComponent&lt;ReflectiveSurface&gt;();
                    if (surface != null)
                    {
                        direction = surface.GetReflectionDirection();
                        position = hit.point;
                        remainingBounces--;
                        continue;
                    }
                }
                else if (hit.collider.CompareTag(&quot;Switch&quot;))
                {
                    SwitchControl switchControl = hit.collider.GetComponent&lt;SwitchControl&gt;();
                    if (switchControl != null)
                    {
                        hitSwitch = switchControl;
                        switchControl.Activate();
                    }
                }

                break;
            }
            else
            {
                lightPoints.Add(position + direction * 1000f);
                break;
            }
        }

        if (currentSwitch != hitSwitch)
        {
            currentSwitch?.Deactivate();
            currentSwitch = hitSwitch;
        }

        lineRenderer.positionCount = lightPoints.Count;
        lineRenderer.SetPositions(lightPoints.ToArray());
    }
}
 </code>
  </pre>
  </div>

  <div class="gif-container fade-in">
    <img src="{{ 'assets/images/inverse/Refraction.gif' | relative_url }}" alt="Light refraction demo">
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
    box-shadow: 0 0 20px rgba(160, 31, 153, 0.81);
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }

  .gif-container img:hover {
    transform: scale(1.03);
    box-shadow: 0 0 25px rgba(160, 31, 153, 1);
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