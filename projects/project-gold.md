---
layout: default
title: Project Gold - Combat Systems & Party Stat Viewer
accent: "#e6b64d"
---

<section class="project-hero" style="background-image: url('{{ 'assets/images/project-gold/title.png' | relative_url }}');">
  <div class="overlay">
    <h1>Project Gold: Combat Systems &amp; Party Stat Viewer</h1>
  </div>
</section>

<section class="project-details">
  <div class="project-info-grid">
    <div><strong>Project Type</strong><br>RPG — Black Star Creatives</div>
    <div><strong>Date</strong><br>August 2026</div>
    <div><strong>Status</strong><br>In Development</div>
    <div><strong>Engine</strong><br>Unity Engine 6</div>
    <div><strong>Role</strong><br>Systems Programmer · Designer</div>
  </div>

  <p class="project-description">
    <em>Project Gold</em> is a turn-based RPG in development at Black Star Creatives, built around an Emotion System as its central gimmick — every playable character is aligned to one of the Seven Deadly Sins, and climbing an emotion ladder in battle means giving in to that sin before it resolves into its opposing virtue. This page focuses on the foundational systems work I built that has to exist before any of that can be played: a data-driven character, skill, and gear model, plus a custom Unity Editor tool built with UI Toolkit for authoring and live-tuning it.
  </p>

  <div class="media-gallery">
    <img src="{{ 'assets/images/project-gold/title.png' | relative_url }}" alt="Party Stat Viewer overview">
    <img src="{{ 'assets/images/project-gold/viewer-tabs.png' | relative_url }}" alt="Sin-affinity colored tabs">
    <img src="{{ 'assets/images/project-gold/viewer-gear.png' | relative_url }}" alt="Gear and effective stats panel">
  </div>
</section>

<section class="project-section fade-in">
  <h2>Introduction</h2>
  <p>
    Before any combat logic can exist, something has to represent a character, a skill, and a weapon — and answer the boring-but-critical questions underneath: how does a weapon's stat bonus differ from an accessory's, how do skills unlock as a character or their gear levels up, and how do you tell a party member apart from an enemy in code without duplicating half the data model. That's the layer this work covers.
  </p>
  <p>
    Rather than build this blind, I paired it with a custom <strong>UI Toolkit</strong> editor window — a Party Stat Viewer that tabs through the active party, shows every stat live, and recomputes derived values (effective stats, unlocked skills) in real time as the underlying data changes. It doubled as the way I pressure-tested the data model itself: several real design flaws surfaced specifically because I could see and edit the numbers live instead of reasoning about them in the abstract.
  </p>
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
  <h2>Custom Editor Tooling with UI Toolkit</h2>
  <p>
    The Party Stat Viewer is a <code>UnityEditor.EditorWindow</code> built with UXML/USS rather than legacy IMGUI, so it works identically in and out of Play mode and can be authored declaratively. Its detail panel binds directly to each selected character's <code>SerializedObject</code> — editing any field writes straight back to the underlying asset with full Undo support, the same mechanism the built-in Inspector itself relies on:
  </p>

  <div class="code-block fade-in">
    <pre><code class="language-csharp">
currentSerializedObject = new SerializedObject(member);
detailContainer.Bind(currentSerializedObject);

detailContainer.TrackSerializedObjectValue(currentSerializedObject, _ =>
{
    RefreshEffectiveStats();
    RefreshWeaponSkills();
    RefreshSelfSkills();
});
    </code></pre>
  </div>

  <p>
    Not everything in the panel is a stored field, though — effective stats and unlocked skills are <em>computed</em>, not serialized, so they can't use the same declarative <code>PropertyField</code> binding as the raw stats. <code>TrackSerializedObjectValue</code> solves that by firing a callback whenever anything on the bound object changes, which recomputes and redraws those derived sections live as you tune weapon levels, swap accessories, or level a character up.
  </p>
  <p>
    Party tabs are generated dynamically — one per roster member — and tinted per the character's Sin affiliation (Pride, Greed, Wrath, Envy, Gluttony, Lust, Sloth), using an inline style override that deliberately wins over the stylesheet's selection-state styling, so a tab's sin color stays visible whether or not it's the active tab.
  </p>
</section>
