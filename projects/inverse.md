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
    <div><strong>Role</strong><br>Lead Programmer / Designer</div>
    <div><strong>Team Size</strong><br>5</div>
  </div>

  <p class="project-description">
    A 3D platformer focused on solving puzzles split between two worlds, your goal is to solve the puzzles and reach the end in both worlds. 
  </p>

  <div class="media-gallery">
    <img src="{{ 'assets/images/inverse/title.png' | relative_url }}" alt="Screenshot 1">
    <img src="{{ 'assets/images/inverse/inversegame1.gif' | relative_url }}" alt="Screenshot 2">
  </div>
</section>

<section class="project-section fade-in">
  <h2>Introduction</h2>
  <p>
    <em>Inverse</em> was developed as a prototype for a game jam involving reflections. My team's idea was to use literal reflections as a mechanism but also the idea of a relfection of ones self as part of the thematic.
  </p>
  <p>
    Built in Unity, the main features this game has are a character swapping system, and a light bending mechanism.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Character Swap System</h2>
  <p>
    In <em>Inverse</em> project, the player controls two fully independent characters and can switch between them at any time. Each character has their own movement controller, animations, stats, and physics body. The swap mechanic is designed not just for cosmetic variety, but to encourage puzzle-solving and traversal using each character's unique abilities.
  </p>
  <p>
    When the player swaps:
<ul>
<li>The newly selected character becomes the only one with active movement input</li>

<li>Animations update automatically based on which character is currently active</li>

<li>Physics are toggled so the inactive character freezes in place (using isKinematic)</li>

<li>The camera dynamically refocuses using Cinemachine, smoothly shifting its Follow and LookAt targets</li>

<li>The inactive character remains present in the world, allowing scenarios such as cooperative platforming puzzles, positional switching, and multi-step level traversal where character placement matters.</li>
</ul>
  </p>

  {% highlight csharp %}
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
        
       pCam = GetComponentInChildren<Camera>();

        if(guiManager == null)
        {
            guiManager = GetComponentInParent<GameManager>().GetComponentInChildren<IGUIManager>();

        }
       

    }

    public void Start()
    {
        
    }

    private void Update()
    {
        if (guiManager == null)
        {
            guiManager = GetComponentInParent<GameManager>().GetComponentInChildren<IGUIManager>();

        }
    }



    void OnEnable() => controls.Enable();
    void OnDisable() => controls.Disable();

    void SwapCharacter()
    {
        Debug.Log(characterController.Name + " " + characterController.ID.ToString() + " Character Swap Pressed");
        // Swap active character
        activeCharacter = (activeCharacter == characterA) ? characterB : characterA;
        SetActiveCharacter(activeCharacter);
        guiManager.SwitchCharacterImage();
    }

    void SetActiveCharacter(GameObject character)
    {
       
        // Enable PlayerInput on the active character and disable on the other.
        characterA.GetComponent<CharacterController3D>().enabled = (character == characterA);
        characterB.GetComponent<CharacterController3D>().enabled = (character == characterB);
        characterA.GetComponentInChildren<Animator>().enabled = (character == characterA);
        characterB.GetComponentInChildren<Animator>().enabled = (character == characterB);
        characterA.GetComponent<Rigidbody>().isKinematic = (character != characterA);
        characterB.GetComponent<Rigidbody>().isKinematic = (character != characterB);
        pCam.GetComponent<CinemachineFreeLook>().Follow = activeCharacter.transform;
        pCam.GetComponent<CinemachineFreeLook>().LookAt = activeCharacter.transform;


    }

    public void SetupCharacters(CharacterController3D[] characterList)
    {
        allCharacters = characterList.Select(cc => cc.gameObject).ToArray();
        characterA = allCharacters.FirstOrDefault(go => go.GetComponent<CharacterController3D>().ID == 1);
        characterB = allCharacters.FirstOrDefault(go => go.GetComponent<CharacterController3D>().ID == 2);

        if (characterA != null && characterB != null)
        {
            activeCharacter = characterA;
            SetActiveCharacter(activeCharacter);
            characterController = activeCharacter.GetComponent<CharacterController3D>();

            controls.Player.SwapCharacter.performed += ctx => SwapCharacter();
            pCam.GetComponent<CinemachineFreeLook>().Follow = activeCharacter.transform;
            pCam.GetComponent<CinemachineFreeLook>().LookAt = activeCharacter.transform;
        }
        else
        {
            Debug.LogError("Characters with specified IDs not found.");
        }
    }


}
  {% endhighlight %}

  <div class="gif-container fade-in">
    <img src="{{ 'assets/images/inverse/swap.gif' | relative_url }}" alt="Combat Demo">
  </div>
</section>


<section class="project-section fade-in">
  <h2>Light Refraction System</h2>
  <p>
    In <em>Inverse</em>, light is used as a core puzzle mechanic. The player interacts with beams that bounce off reflective surfaces and activate switches throughout the environment. The light is dynamically rendered using a LineRenderer, allowing it to visually update in real time as it moves through the world.
  </p>

  <p>
  The beam begins at a defined origin point and travels forward until it hits an object. If the object is reflective, the beam bounces and continues in a new direction, enabling mirror-based puzzles and multi-step routing challenges. If the beam strikes a switch, it powers it on, allowing light to be used as a way to unlock doors or trigger mechanisms. Once the beam stops touching that switch, it automatically deactivates, preventing players from activating multiple switches simultaneously just by moving the beam once.
  </p>
 

<div class="code-block fade-in">
{% highlight csharp %}
using UnityEngine;

public class LightEmitter : MonoBehaviour
{
    public LineRenderer lineRenderer;
    public int maxBounces = 10; // Maximum number of light bounces
    public Transform lightOrigin;
    public Material lightMaterial; 
    public bool useCustomMaterial = true;
    // Store the switch that was hit in the previous frame
    private SwitchControl currentSwitch = null;

    void Start()
    {
        if (lineRenderer == null)
        {
            lineRenderer = gameObject.AddComponent<LineRenderer>();
        }

        // Material setup
        if (useCustomMaterial && lightMaterial != null)
        {
            lineRenderer.material = lightMaterial;
        }
        else
        {
            // Default material setup
            lineRenderer.material = new Material(Shader.Find("Sprites/Default"));
        }

        // Rest of your existing Start() code...
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
        List<Vector3> lightPoints = new List<Vector3>();
        lightPoints.Add(position);

        // Local variable to hold the switch hit in this cast
        SwitchControl hitSwitch = null;

        while (remainingBounces > 0)
        {
            RaycastHit hit;
            if (Physics.Raycast(position, direction, out hit, Mathf.Infinity))
            {
                lightPoints.Add(hit.point);

                if (hit.collider.CompareTag("Reflective"))
                {
                    ReflectiveSurface surface = hit.collider.GetComponent<ReflectiveSurface>();
                    if (surface != null)
                    {
                        // Update direction and position, then decrement bounce counter
                        direction = surface.GetReflectionDirection();
                        position = hit.point;
                        remainingBounces--;
                        continue;
                    }
                }
                else if (hit.collider.CompareTag("Switch"))
                {
                    SwitchControl switchControl = hit.collider.GetComponent<SwitchControl>();
                    if (switchControl != null)
                    {
                        hitSwitch = switchControl;
                        // Activate the switch if it's hit
                        switchControl.Activate();
                    }
                }
                break; // End the loop if we hit a non-reflective surface
            }
            else
            {
                lightPoints.Add(position + direction * 1000f);
                break;
            }
        }

        // Only call Deactivate if the switch hit this frame is different from the previously hit switch.
        if (currentSwitch != hitSwitch)
        {
            if (currentSwitch != null)
            {
                currentSwitch.Deactivate();
            }
            currentSwitch = hitSwitch;
        }

        lineRenderer.positionCount = lightPoints.Count;
        lineRenderer.SetPositions(lightPoints.ToArray());
    }
}
{% endhighlight %}
</div>

<!-- Ensure the image is visually separated from the code block -->
<div style="clear: both; height: 1rem;"></div>

<div class="gif-container fade-in" style="clear: both; margin-top: 1rem;">
  <img src="{{ 'assets/images/inverse/Refraction.gif' | relative_url }}" alt="Combat Demo">
</div>
<style>
/* Prevent horizontal scrollbar for highlighted C# code by wrapping lines */
div.highlight pre,
pre[class*="language-"],
code[class*="language-"] {
    white-space: pre-wrap !important;
    word-break: break-word !important;
    overflow-x: hidden !important;
}
</style>

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
   
    box-shadow: 0 0 20px rgba(160, 31, 153, 0.81);
   
}

.gif-container img:hover {
    transform: scale(1.03);
    box-shadow: 0 0 25px rgba(160, 31, 153, 1);
}
</style>

