## Project Overview

---

## Core Design Pillars

1. **Player-Driven Creation**: No pre-made items. Every sword, potion, spell, armor piece, and consumable is crafted by players.
2. **Radical Customization**: Players customize keybinds, HUD layout, ability bar organization, visual themes, and accessibility settings.
3. **True Multiplayer**: Real-time cooperative and competitive play, shared persistent world, player-driven trading and economy.
4. **2.5D Isometric Vision**: Top-down isometric perspective allowing vertical height, depth, and layered environmental interaction.
5. **Emergent Gameplay**: Systems enable unexpected interactions; crafters and fighters coexist with equal value.

---

## Architecture & Technical Foundation

### Backend Requirements

**Server Type**: Node.js + WebSocket or custom game server (Colyseus, Agones, or custom)

**Core Systems**:
- **Real-time Synchronization**: Low-latency player position, animation, spell effects, and item state
- **Item Persistence**: Database (MongoDB, PostgreSQL) storing all player-created items with stats, recipes, and ownership
- **Crafting Engine**: System managing recipes, ingredient validation, and item creation with crafted item ownership/trading
- **Combat Resolution**: Server-authoritative hit detection, damage calculation, and cooldown management to prevent cheating
- **Economy & Trading**: Secure player-to-player trading, auction house (if applicable), and currency system
- **Account System**: Player authentication, character slots, and progression persistence

**Database Schema** (minimum entities):
- Users, Characters, Inventory
- Items (template), ItemInstances (with unique stats/ownership)
- Recipes, CraftingQueues
- Spells (templates), SpellInstances (custom per character)
- World State (dynamic obstacles, NPC state, world events)
- Transactions (trading ledger)

### Client Requirements

**Tech Stack**: Three.js, Babylon.js, or Godot (with HTML5 export) for 2.5D rendering

**Core Modules**:
- 2.5D Isometric Renderer with depth sorting and dynamic lighting
- Input Manager with rebindable keybindings and input buffering
- UI Framework supporting customizable HUD, drag-and-drop, and theme application
- Animation Controller for character states (idle, walking, casting, damaged, dead)
- Network Layer for smooth interpolation between server ticks
- Local Storage for keybinds, UI preferences, and settings

---

## Gameplay Systems

### Character & Progression

**Character Creation**:
- Name, appearance (customizable avatar parts: hair, skin, build)
- Choose a starting "path" (not a class lock—these are merely starting guides):
  - **Warrior**: Melee focus, heavy armor affinity
  - **Mage**: Spell casting, mana-intensive
  - **Rogue**: Speed, stealth, single-target burst
  - **Crafter**: Starting with crafting recipes and tools (still combat-viable)
- Players can respec abilities and paths freely at any time

**Progression**:
- Experience earned from combat, crafting, and exploration
- Leveling increases base stats: Health, Mana, Stamina, Armor
- Ability points allow players to unlock or upgrade spells/techniques
- No gear tiers or mandatory progression paths

### Combat System

**Movement & Positioning**:
- WASD or rebindable keys for 8-directional movement
- Click-to-move as alternative input
- Stamina cost for running; regenerates when standing still
- Height/elevation affects range and line-of-sight (environmental verticality)

**Attacking**:
- Left-click or rebindable key for basic attack (melee or ranged, depends on equipped weapon)
- Basic attack animation, travel time, and hit-lag feedback
- Server validates hit detection; client predicts for visual feedback

**Ability Bar & Cooldowns**:
- Customizable hotbar (default: 1-9 keys, rebindable)
- Players drag spells/abilities onto hotbar slots
- Global cooldown (0.5s) prevents ability spam
- Per-ability cooldowns vary by item/spell (0.5s to 60s+)
- Mana/stamina costs displayed and enforced server-side

**Damage & Effects**:
- Damage calculated as: Base Damage × (1 + Stat Multipliers) ± RNG variance
- Status effects (burning, frozen, poisoned, stunned, bleeding) stack with diminishing returns
- Healing scales with Wisdom or Vitality (crafters decide)
- Damage numbers float above targets; customizable color and size

### Crafting System

**Crafting Categories**:
1. **Weapons**: Swords, axes, bows, staves, daggers, hammers
2. **Armor**: Helmets, chest, gloves, legs, boots
3. **Accessories**: Rings, amulets, belts (stat bonuses, special effects)
4. **Potions & Consumables**: Health, mana, stamina potions; buffs; antidotes
5. **Spells & Runes**: Teachable spells; runes that modify abilities
6. **Tools**: Crafting tools, gathering tools, cooking equipment

**Recipe Creation** (by players with Crafter role or crafting recipes):
- Define inputs: Base material + modifiers + special components
- Define outputs: Item name, stats, visual appearance, special effects
- Example recipe:
  ```
  Iron Longsword
  Inputs: Iron Ingot (3), Wooden Handle (1), Leather Grip (1)
  Outputs: Longsword | Damage: 15–22 | Weight: 8 | Cooldown: 0.8s
  Crafting Time: 30s
  ```

**Crafting Process**:
1. Player selects recipe from their known recipes
2. System checks inventory for all required materials
3. Player confirms; items consumed; crafting begins
4. Crafting happens over time (seconds to hours, configurable per recipe)
5. On completion, new item appears in inventory with unique ID
6. Crafter is recorded as the item's creator (appears on item tooltip)

**Item Properties** (defined per recipe):
- **Stats**: Damage, Armor, Health, Mana, Stamina, element resist, etc.
- **Weight**: Affects movement speed when heavy
- **Rarity**: Common, Uncommon, Rare, Epic, Legendary (visual color indicator)
- **Special Effects**: On-hit procs, stat bonuses, unique abilities
- **Durability**: Items degrade with use; repaired by crafters or consumed when broken
- **Visual Model**: Customizable color, glow, particle effects

### Spell & Ability System

**Spell Creation** (by players with Spellcraft recipes):
- Define: Name, icon, cost (mana/stamina), cooldown, range, effect
- Example spell:
  ```
  Fireball
  Cost: 50 mana
  Cooldown: 4s
  Range: 15 tiles
  Effect: Damage 20–30 fire damage; ignites enemies; damage over 5s
  Animation: 0.6s cast time; projectile; AoE on impact
  ```

**Ability Customization**:
- Players can learn multiple spells (up to a limit; points determine how many active abilities)
- Each ability placed on hotbar with custom keybind
- Abilities scale with player stats (Spellpower, Intelligence, etc.)
- Cooldown, range, and effect are server-authoritative; client shows cooldown timer

**Spell Modifiers** (Runes):
- Runes found/crafted modify spells at cast-time
- Example: Frostbolt (basic) + Amplify Rune = Frostbolt deals 20% more damage
- Multiple runes can stack (crafters define rules)

### Exploration & World

**World Structure**:
- Persistent procedurally generated or hand-crafted zones
- Multiple biomes: Forest, Mountains, Desert, Dungeon, Urban
- Dungeons and dungeons reset periodically or per-instance
- Players can create temporary camps/lodges (player housing, optional)

**Environmental Interaction**:
- Breakable crates, barrels → loot or materials
- Switches, levers → environmental puzzles
- Vendors (NPC or player-run shops)
- Gathering nodes: Wood, stone, herbs, ore → crafting materials
- Dynamic events triggered by server (world bosses, invasions)

**Fast Travel**:
- Waypoints at major towns; players unlock via exploration
- Teleport cost (currency or items)

---

## Customization Systems

### Keybindings

**Input Manager**:
- Every action rebindable: Movement, attack, abilities, interact, menu, etc.
- Presets available: WASD, Arrow Keys, Gamepad layouts
- Conflict detection (prevent two actions on same key)
- Save and load multiple profiles
- Mouse sensitivity and gamepad deadzone customization

**Rebinding UI**:
- Settings > Controls
- Table showing action name | current keybind
- Click to rebind; press the new key; confirm
- "Reset to Defaults" button clears all custom binds

### HUD Customization

**Draggable HUD Elements**:
- Health bar, mana bar, experience bar
- Hotbar (ability bar)
- Minimap and full map
- Chat window, log, notifications
- Inventory quick-access
- Buffs/Debuffs display
- Target info panel

**Per-Element Settings**:
- Toggle visibility
- Resize and reposition via drag
- Adjust opacity and scaling
- Customize colors and fonts

**HUD Themes**:
- Dark, Light, High Contrast presets
- Custom theme creation: Define colors for background, text, accent, borders
- Font selection (system defaults + web fonts)
- Save and share custom themes

### Accessibility

**Visual**:
- Color-blind friendly palettes (deuteranopia, protanopia, tritanopia)
- Adjustable UI scale (75%–200%)
- Dyslexia-friendly font option
- Reduced motion option (disables non-essential animations)

**Audio**:
- Master volume, SFX volume, music volume, dialog volume
- Closed captions for all dialog and important sounds
- Text-to-speech for chat (optional)
- Sound direction indicators (visual arrows for audio cues)

**Input**:
- Remappable keys for all actions
- Gamepad full support with customizable layout
- One-handed play option (rebind to single-hand layout)
- Hold-to-cast vs. toggle-cast options

**Difficulty**:
- Toggle PvP on/off
- Adjust AI difficulty (PvE only)
- Enemy scaling (scale to player level or fixed)
- Damage numbers always visible or on-demand

### Graphics Settings

- Resolution and fullscreen toggle
- Framerate cap (30, 60, 120, unlimited)
- Graphics quality preset (Low, Medium, High, Ultra)
- Draw distance and LOD settings
- Particle effect intensity
- Lighting quality (simple, dynamic, fancy)
- Shadows on/off
- Anti-aliasing options

---

## Social & Multiplayer Features

### Party & Grouping

**Party System**:
- Create or join party (up to 5 players)
- Party leader can invite others or set to public queue
- Shared experience and loot for group activities
- Party chat channel
- Party member list shows status and location

**Guilds** (optional):
- Player-created guilds with member roles (Leader, Officer, Member)
- Guild chat and announcements
- Guild hall (shared space, could have guild bank)
- Guild quests and events

### Trading & Economy

**Player-to-Player Trading**:
- Initiate trade with another player
- Each player adds items/currency to their side
- Both confirm before transaction executes
- Trade log recorded on server for dispute resolution

**Auction House** (optional):
- Players post items for sale
- Searchable by item type, stats, crafted-by name
- Automatic expiration after 48h or sale
- Server takes small tax on sales
- Buy-it-now or bidding systems

**Currency**:
- Gold (earned from combat, quests, selling items)
- Crafting materials as secondary currency
- Player-to-player gold transfer (optional, with limits to prevent RMT)

### Chat & Communication

**Chat Channels**:
- Local (nearby players)
- Party
- Guild
- Global (server-wide, moderated)
- Whisper (private messages)
- Trading channel

**Moderation**:
- Report button for offensive chat
- Server-side word filter
- Mute/block other players
- Admin tools for chat management

---

## Progression & Endgame

### Quests & Objectives

**Quest Types**:
1. **Tutorial Quests**: Guide new players through systems
2. **Crafting Quests**: "Craft 10 swords" or "Create a legendary item"
3. **Combat Quests**: "Defeat boss X" or "Gather 100 combat XP"
4. **Exploration**: "Visit all waypoints" or "Find hidden chest"
5. **Social**: "Join a guild" or "Trade with 5 players"

**Dynamic Events**:
- Server-triggered world events (dragon invasion, meteor shower)
- Limited-time challenges with rewards
- Leaderboards for fastest clear times or highest damage

### Progression Paths

**Power Progression**:
- Character level (1–100+) through experience
- Stat increases per level (health, mana, damage)
- Unlock ability points for new spells
- Legendary items as rare, powerful drops from world bosses

**Mastery & Specialization**:
- Weapon mastery (using a type improves efficiency)
- Spell school mastery (casting type improves)
- Crafter specialization (focusing on one craft type)
- Combat role specialization (tank, DPS, support)

**Cosmetics & Prestige**:
- Seasonal cosmetics (cosmetic armor, weapon skins)
- Title/badge system (based on achievements)
- Mount/pet cosmetics (earned, not pay-to-win)
- Custom character skins and effects

---

## Technical Implementation Checklist

### Backend
- [ ] WebSocket server (or game server framework)
- [ ] User authentication & session management
- [ ] Database schema (users, items, crafting, transactions)
- [ ] Real-time state synchronization
- [ ] Server-authoritative combat validation
- [ ] Crafting queue and completion system
- [ ] Trading transaction security
- [ ] Economy monitoring (inflation detection)
- [ ] Admin tools and logging
- [ ] Backup and disaster recovery

### Client
- [ ] 2.5D isometric renderer
- [ ] Input handler with full rebinding support
- [ ] UI framework and theme system
- [ ] Network interpolation for smooth player movement
- [ ] Ability casting and animation system
- [ ] Inventory and crafting UI
- [ ] Settings/preferences persistence (localStorage)
- [ ] Accessibility features (color modes, scaling, audio)
- [ ] Chat and social systems
- [ ] Map and navigation

### Content Creation Tools (For Players)
- [ ] In-game recipe editor (UI to define items)
- [ ] Spell configuration panel
- [ ] Visual item preview before crafting
- [ ] Template system (start with examples)

---

## Monetization (Optional)

**Non-Pay-to-Win Model**:
- Cosmetics only (skins, emotes, mounts, pets)
- Battle pass with cosmetic rewards
- Server cosmetic shop
- NO stat advantages, power-ups, or pay-gated items
- Crafted items never sold for real money

**Alternative Models**:
- Free-to-play with cosmetics
- One-time purchase (no subscriptions)
- Subscription to cosmetic-only battle pass
- Player donations for server costs

---

## Success Metrics

1. **Engagement**: Daily active users, session length, retention after 1/7/30 days
2. **Creation**: Items created per day, recipes published, crafters active
3. **Economy Health**: Currency distribution, trading volume, inflation rate
4. **PvP Health**: Win rates, matchmaking quality, queue times
5. **Quality**: Bug reports, crash rates, server ping
6. **Community**: Guild formation, player-created events, social interaction

---

## Launch Roadmap (Suggested)

**Phase 1 - Core (Months 1–3)**:
- Basic 2.5D combat, keybinds, simple crafting
- 1–2 starter zones
- Party system, basic trading
- 50–100 craftable items

**Phase 2 - Content (Months 4–6)**:
- Guilds, auction house, advanced crafting
- Spell creation system
- 3–4 more zones, world bosses
- 500+ item recipes

**Phase 3 - Polish & Balance (Months 7–9)**:
- PvP arena, seasonal content
- Advanced cosmetics, mounts
- Economy balancing, inflation control
- Streamer/creator tools

**Phase 4 - Live Service**:
- Monthly content updates
- Seasonal events
- Community-driven balance patches
- Expansion zones

---

## Reference Inspirations

- **Dark Souls / Elden Ring**: Combat feel, difficulty, 2.5D movement
- **Runescape / Oldschool Runescape**: Player-driven economy, crafting importance
- **Minecraft**: Player creation, item crafting, emergent gameplay
- **Path of Exile**: Skill gems, item rarity, economy complexity
- **Guild Wars 2**: Dynamic events, customization, no gear tiers
- **Sea of Thieves**: Cooperative multiplayer, emergent storytelling

---

## Closing Notes

This game succeeds when **creators and fighters have equal value**, when **every item tells a story of its crafter**, and when **players feel genuine agency over their character and the world**. Avoid: pay-to-win mechanics, forced progression gates, and pre-made item tables that stifle player creativity.

The core loop is: **Play → Gather Materials → Craft → Trade → Use → Repeat**. Every system should reinforce this loop and make it fun at each step.

Good luck, and build something legendary! 
