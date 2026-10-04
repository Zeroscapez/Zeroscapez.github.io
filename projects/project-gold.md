---
layout: default
title: Project Gold
accent: "#e6b64d"
---

<section class="project-hero project-hero--placeholder">
  <div class="overlay">
    <h1>Project Gold</h1>
    <p class="hero-status">In Prototyping</p>
  </div>
</section>

<section class="project-details">
  <div class="project-info-grid">
    <div><strong>Project Type</strong><br>RPG — Black Star Creatives</div>
    <div><strong>Date</strong><br>August 2026</div>
    <div><strong>Status</strong><br>Prototyping</div>
    <div><strong>Engine</strong><br>Unity Engine 6</div>
    <div><strong>Role</strong><br>Systems Programmer · Designer</div>
  </div>

  <p class="project-description">
    <em>Project Gold</em> is a turn-based RPG in development at Black Star Creatives, the indie studio I co-founded. Every playable character is aligned to one of the Seven Deadly Sins, and climbing the emotion ladder in battle means giving in to that sin — gaining power along with its flaw — until it resolves into its opposing virtue. I designed that Emotion System, and I'm building the combat systems, data model, and editor tooling behind it.
  </p>

  <p class="media-note">Gameplay footage is coming once the project moves out of prototyping.</p>
</section>

<section class="project-section fade-in">
  <h2>Introduction</h2>
  <p>
    <em>Project Gold</em> is where my design and engineering work meet most directly. I designed its central mechanic, the Emotion System, and I'm building everything needed to make it playable and tunable: the combat systems that run it, the data model underneath them, and a custom editor tool for balancing it all.
  </p>
  <p>
    This page walks through the design first, then the systems and tooling behind it.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Designing the Emotion System</h2>
  <p>
    Every playable character is aligned to one of the Seven Deadly Sins. Climbing the emotion ladder in battle is <em>giving in</em> to that sin: power grows, but so does a characteristic flaw — until, at the top, the sin resolves into its opposing virtue. The whole arc in one line, from my design spec:
  </p>
  <p>
    <em>L1 restraint → L2–L3 the sin stirring → L4 the sin in full (power + its flaw) → L5 the virtue (mastery, flaw gone).</em>
  </p>
  <p>I split the system into two tracks so it can never punish players for engaging with it:</p>
  <ul>
    <li><strong>Emotion Level (1–5)</strong> is a <em>state</em>, not a resource. It climbs with momentum and gates passives, the Sin State, and Resolute — and nothing an enemy does can knock you down a level.</li>
    <li><strong>Emotion Points (EP)</strong> are the fuel for emotion skills, spent in whole charges of 100. Spending EP never changes your level, so cashing in a skill can't knock you out of a buff or out of Resolute.</li>
  </ul>
  <p>
    The charge ceiling per level is 1, 2, 3, 3, 4 — Level 4 deliberately matches Level 3. The Sin State is the struggle; the extra capacity waits for Resolute, as the payoff for pushing through it. Levels 4 and 5 are fixed landmarks for every character; what changes from character to character is what their sin does to them there.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Case Study: Logos and Dragon's Blood</h2>
  <p>
    Logos is <strong>Greed → Charity</strong>: she hoards because she's shy and afraid to share. Her innate passive, <em>Dragon's Blood</em>, turns that into a mechanic. Every landed hit adds a Dragon's Eye stack (more on crits and weakness hits), and each stack adds flat damage — the fuller the hoard, the harder she hits. Her Emotion Level changes the terms as she climbs:
  </p>
  <ul>
    <li><strong>L3:</strong> crits and weakness hits pay an extra stack.</li>
    <li><strong>L4 — Sin State:</strong> per-stack damage doubles, but every hit she takes strips the hoard. She's at her most dangerous and her most fragile at the same time — Greed's flaw in one rule.</li>
    <li><strong>L5 — Resolute:</strong> the cap comes off and the hoard can no longer be stripped. In the full design, this is where she finally shares it: generosity was what her greed was hiding all along.</li>
  </ul>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
public override void OnTookDamage(CombatActor self, CombatActor attacker, DamageBreakdown breakdown)
{
    // Plundered — but only in the Sin State. Below it the hoard is safe, and Resolute lifts the flaw
    // again, which is why IsInSinState is an exact level match rather than a threshold.
    if (!self.IsInSinState) return;
    if (breakdown.finalDamage &lt;= 0) return;

    self.AddStacks(this, -stacksLostWhenHit);
}

public override int ModifyOutgoingDamage(CombatActor self, CombatActor target, int damage)
{
    int bonus = self.GetStacks(this) * damagePerStack;

    // From the Sin State on, the hoard hits twice as hard — which is exactly why losing it costs her twice.
    if (self.emotionLevel &gt;= CombatActor.SinStateLevel)
    {
        bonus *= 2;
    }

    return damage + bonus;
}
    </code></pre>
  </div>

  <p>
    One detail worth calling out: <code>IsInSinState</code> is true <em>only</em> at Level 4, not "Level 4 and up." Resolute is supposed to lift the flaw, so a threshold check would have quietly kept stripping her hoard at Level 5 — the opposite of what the design promises.
  </p>
</section>

<section class="project-section fade-in">
  <h2>A Passive System Built for Iteration</h2>
  <p>
    Characters get designed faster than they get built, so the passive system needed to let design run ahead of code without the two drifting apart:
  </p>
  <ul>
    <li><strong>Described before built.</strong> The base <code>PassiveData</code> class is concrete, not abstract. A plain asset with a name, description, and per-level notes is a valid "designed, not yet implemented" passive that can be slotted onto a character and read in the tools today; passives with real behaviour subclass it and override only the hooks they need.</li>
    <li><strong>Descriptions can't drift.</strong> Implemented passives generate their per-level description from their own serialized numbers, so retuning a value retunes the text. Hand-written notes only cover levels that aren't built yet.</li>
    <li><strong>No stat hook, on purpose.</strong> A passive that modified stats would be called from the stat calculation, and a passive that reads a stat would call straight back into it. Stat changes go through the existing modifier layers instead, which can't recurse.</li>
    <li><strong>Shared assets stay read-only.</strong> Passives are ScriptableObjects shared by every combatant, so per-battle state like Dragon's Eye stacks lives on each <code>CombatActor</code>, keyed by the passive — never on the asset, where it would leak between characters and into the next play session.</li>
  </ul>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
// Built from the serialized fields rather than written out, so retuning any of them retunes the
// Party Stat Viewer's ladder readout with it.
public override string DescribeEmotionLevel(int level)
{
    if (level == CombatActor.SinStateLevel)
    {
        string stacks = stacksLostWhenHit == 1 ? "1 stack" : $"{stacksLostWhenHit} stacks";
        return $"Sin State. Per-stack damage doubles ({damagePerStack} → {damagePerStack * 2} each), but every " +
               $"hit taken strips {stacks}. Augments cost no Dragon's Eye at all from here.";
    }

    // ...one branch per level this passive affects; null for levels where it does nothing
}
    </code></pre>
  </div>
</section>

<section class="project-section fade-in">
  <h2>A Data-Driven Character, Skill, and Gear Model</h2>
  <p>
    Characters, skills, and gear are all ScriptableObject-based, which keeps the whole system designer-editable without touching code. The core split is between <code>CharacterData</code> — shared identity and base stats (STR/MAG/ACT/DEF/AGI/MP/HP/EP), usable by enemies too — and <code>PartyMemberData</code>, which subclasses it to add party-only concerns: an equipped weapon, equipped accessories, and self-taught skill unlocks. A <code>PartyRoster</code> holding a plain <code>List&lt;CharacterData&gt;</code> accepts <code>PartyMemberData</code> instances polymorphically, so the party/enemy boundary is enforced by the type system rather than by convention.
  </p>
  <p>
    Weapons level from 1 to 5, unlocking one of four skills per level and applying hand-authored, non-linear stat modifiers at each level — some stats go up, others deliberately go down, since a real weapon-upgrade curve isn't just "+1 to everything." Accessories layer on top with both flat and percentage modifiers to stats and elemental resistances. All of it feeds into a single computed method:
  </p>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
public int GetEffectiveStat(StatType stat)
{
    int value = GetBaseStat(stat);

    if (equippedWeapon.definition != null)
    {
        int levelCount = Mathf.Clamp(equippedWeapon.currentLevel, 1, equippedWeapon.definition.levels.Length);
        for (int i = 0; i < levelCount; i++)
        {
            WeaponLevelData weaponLevel = equippedWeapon.definition.levels[i];
            if (weaponLevel == null) continue;

            foreach (StatModifier modifier in weaponLevel.statModifiers)
            {
                if (modifier.stat == stat)
                {
                    value += modifier.amount;
                }
            }
        }
    }

    float percentBonus = 0f;
    foreach (AccessoryData accessory in equippedAccessories)
    {
        if (accessory == null) continue;

        foreach (AccessoryStatModifier modifier in accessory.statModifiers)
        {
            if (modifier.stat != stat) continue;

            if (modifier.mode == ModifierMode.Flat)
                value += Mathf.RoundToInt(modifier.amount);
            else
                percentBonus += modifier.amount;
        }
    }

    return Mathf.RoundToInt(value * (1f + percentBonus));
}
    </code></pre>
  </div>

  <p>
    Weapon levels are summed cumulatively (each level stores a <em>delta</em>, not a running total), and accessory percentage bonuses stack additively before being applied once at the end — three +10% accessories add up to +30%, not a compounded ×1.1³. Compounding percentages get disproportionately strong the more gear you stack, which isn't the behavior I wanted for this system.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Catching a Real Design Flaw Before It Shipped</h2>
  <p>
    The first pass tracked a weapon's current level directly on the character — a single <code>currentWeaponLevel</code> int alongside <code>equippedWeapon</code>. It looked reasonable until I actually thought through weapon-swapping: since weapons are meant to level up independently through use, that design would tie a weapon's progress to whichever character's equip slot it happened to be sitting in. Swap weapons, and a fresh Level 1 sword would silently inherit whatever level the previous weapon had reached — or worse, a heavily-upgraded weapon would lose its progress the moment it left a character's hands.
  </p>
  <p>
    The fix was to stop treating <code>WeaponData</code> as anything other than a shared <em>definition</em> — a template describing what each of its five levels does — and introduce a proper instance/definition split:
  </p>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
[System.Serializable]
public class WeaponInstance
{
    public WeaponData definition;

    [Range(1, 5)]
    public int currentLevel = 1;
}
    </code></pre>
  </div>

  <p>
    A character's <code>equippedWeapon</code> is now a <code>WeaponInstance</code>, pairing a definition reference with its own progress — the exact same pattern already used for self-taught skill unlocks (<code>{ SkillData skill; int requiredLevel; }</code>), just applied consistently once the gap became obvious. It's a small fix in isolation, but it's the kind of thing that's cheap to correct before other systems start depending on the wrong assumption, and expensive to unwind after.
  </p>
</section>

<section class="project-section fade-in">
  <h2>Tooling: The Party Stat Viewer</h2>
  <p>
    All of this has to be tuned somewhere, so I built a custom editor window for it: the <strong>Party Stat Viewer</strong>, a <code>UnityEditor.EditorWindow</code> built with UI Toolkit (UXML/USS) rather than legacy IMGUI.
  </p>

  <div class="gif-container fade-in">
    <img src="{{ 'assets/images/project-gold/stat-viewer.webp' | relative_url }}" alt="The Party Stat Viewer showing a character's identity, innate passive, and base versus effective stats" width="1525" height="1297" loading="lazy">
  </div>

  <ul>
    <li><strong>Roster tabs</strong> with each character's portrait, underlined in their Sin's color.</li>
    <li><strong>Stats, Skills, and Gear pages.</strong> Skills and Gear disable themselves for non-party characters such as enemies, rather than showing two pages of empty panels.</li>
    <li><strong>Base vs. effective stats</strong> side by side, with any value moved by gear or bonuses shown in gold with its delta — STR 12 (+2), AGI 18 (−2).</li>
    <li><strong>An Emotion Ladder readout:</strong> for each level, its charge ceiling, the character's Level 2 stat swing, what every equipped passive does there (innate, weapon, and accessories), and which skills unlock — the design above, checked against the live data.</li>
  </ul>

  <p>
    The panel binds directly to the selected character's <code>SerializedObject</code>, so editing any field writes straight back to the asset with full Undo support. The derived panels aren't serialized fields, so one subscription redraws all of them whenever anything changes:
  </p>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
currentSerializedObject = new SerializedObject(member);
detailContainer.Bind(currentSerializedObject);

// One subscription covering every field, so edits anywhere refresh the derived panels.
detailContainer.TrackSerializedObjectValue(currentSerializedObject, _ =&gt; RefreshAll());
    </code></pre>
  </div>

  <p>
    One lesson from building it: in UI Toolkit, a mismatch between a UXML element's name and the C# query that looks it up doesn't throw — the field is just <code>null</code>, and the window half-works. After that bit the project once, the window checks every required element at startup and reports any missing ones by name, in the console and in the window itself, instead of failing silently.
  </p>
</section>
