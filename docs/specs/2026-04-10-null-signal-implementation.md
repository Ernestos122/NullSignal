# Null Signal — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build three single-file HTML web apps for the Null Signal cyberpunk skirmish game: a Warband Forge (builder), Rules Webapp (rulebook), and Solo Engine (AI runner).

**Architecture:** Each app is a standalone HTML file — React 18 + Tailwind via CDN, Babel for JSX, localStorage for persistence. All three share a common game data model (tiers, weapons, abilities) embedded as JS constants. A shared JSON data file defines the canonical data, and each app embeds a copy.

**Tech Stack:** React 18 (UMD/CDN), Tailwind CSS (CDN), Babel (CDN), localStorage, JSON export/import. No build step. No node_modules.

**Design Language:** Dark cyberpunk — black/charcoal backgrounds, neon cyan/magenta/amber accents, monospace typography (Share Tech Mono for body, Orbitron for headings), subtle scanline effects, glowing borders.

---

## File Structure

```
NullSignal/
├── NullSignal_Data.json          # Canonical game data (tiers, weapons, abilities)
├── NullSignal_Forge.html         # Warband builder app
├── NullSignal_Rules.html         # Interactive rulebook
├── NullSignal_SoloEngine.html    # Solo/co-op AI engine
└── docs/
    └── specs/
        ├── 2026-04-10-null-signal-design.md
        └── 2026-04-10-null-signal-implementation.md
```

Each HTML file is self-contained. The JSON file is the source of truth for game data but each app embeds its own copy so it works offline without a server.

---

## Phase 1: Game Data + Warband Forge

The Forge is the most complex app and the heart of the project. Build it first.

### Task 1: Game Data JSON

**Files:**
- Create: `NullSignal_Data.json`

- [ ] **Step 1: Create the canonical game data file**

```json
{
  "stats": ["CMB", "REF", "GRT", "TEC", "AWR"],
  "statLabels": {
    "CMB": "Combat",
    "REF": "Reflexes",
    "GRT": "Grit",
    "TEC": "Tech",
    "AWR": "Awareness"
  },
  "diceTypes": ["d6", "d8", "d10", "d12"],
  "healthTrack": ["d10", "d8", "d6", "DOWN", "OUT"],
  "tiers": [
    {
      "id": "leader",
      "name": "Leader",
      "cost": 0,
      "health": "d10",
      "statSpread": { "d10Count": 2, "d8Count": 3 },
      "gearSlots": 2,
      "abilitySlots": 2
    },
    {
      "id": "specialist",
      "name": "Specialist",
      "cost": 3,
      "health": "d8",
      "statSpread": { "d8Count": 2, "d6Count": 3 },
      "gearSlots": 1,
      "abilitySlots": 1
    },
    {
      "id": "operative",
      "name": "Operative",
      "cost": 2,
      "health": "d8",
      "statSpread": { "d8Count": 1, "d6Count": 4 },
      "gearSlots": 0,
      "abilitySlots": 0
    },
    {
      "id": "recruit",
      "name": "Recruit",
      "cost": 1,
      "health": "d6",
      "statSpread": { "d8Count": 0, "d6Count": 5 },
      "gearSlots": 0,
      "abilitySlots": 0
    }
  ],
  "weapons": [
    { "id": "pistol", "name": "Pistol", "attackDice": 2, "rangeBand": 8, "hardCap": 16, "trait": "Compact", "traitDesc": "Easy to conceal, common backup", "melee": false, "premium": false },
    { "id": "smg", "name": "SMG", "attackDice": 3, "rangeBand": 12, "hardCap": null, "trait": "Rapid", "traitDesc": "+1 die within 6\"", "melee": false, "premium": false },
    { "id": "rifle", "name": "Rifle", "attackDice": 3, "rangeBand": 20, "hardCap": null, "trait": "Reliable", "traitDesc": "No special trait", "melee": false, "premium": false },
    { "id": "shotgun", "name": "Shotgun", "attackDice": 3, "rangeBand": 8, "hardCap": 16, "trait": "Spread", "traitDesc": "+1 die within 6\", hard cap", "melee": false, "premium": false },
    { "id": "blade", "name": "Blade", "attackDice": 2, "rangeBand": 0, "hardCap": 0, "trait": "Melee-only", "traitDesc": "Close combat weapon", "melee": true, "premium": false },
    { "id": "sniper_rifle", "name": "Sniper Rifle", "attackDice": 2, "rangeBand": 30, "hardCap": null, "trait": "Precise", "traitDesc": "+1 die beyond range band instead of penalty", "melee": false, "premium": true, "gearCost": 1 },
    { "id": "lmg", "name": "LMG", "attackDice": 4, "rangeBand": 20, "hardCap": null, "trait": "Suppressive", "traitDesc": "Target at -2\" movement next activation", "melee": false, "premium": true, "gearCost": 1 },
    { "id": "launcher", "name": "Launcher", "attackDice": 2, "rangeBand": 20, "hardCap": null, "trait": "Blast", "traitDesc": "All models within 2\" of target point take hits", "melee": false, "premium": true, "gearCost": 1 },
    { "id": "power_weapon", "name": "Power Weapon", "attackDice": 3, "rangeBand": 0, "hardCap": 0, "trait": "Piercing", "traitDesc": "Extra health degradation on failed check", "melee": true, "premium": true, "gearCost": 1 }
  ],
  "abilities": [
    { "id": "hardened", "name": "Hardened", "effect": "+1 step to GRT die for recovery and morale checks" },
    { "id": "cqb_expert", "name": "CQB Expert", "effect": "Bonus die in brawls" },
    { "id": "marksman", "name": "Marksman", "effect": "Bonus die when shooting within range band" },
    { "id": "medic", "name": "Medic", "effect": "Use Gear action: roll TEC to attempt recovery on a Down ally in contact" },
    { "id": "hacker", "name": "Hacker", "effect": "Bonus die on TEC checks for scenario interactions" },
    { "id": "networked", "name": "Networked", "effect": "Once per round, one ally within 6\" may re-roll a failed AWR check" },
    { "id": "stealthy", "name": "Stealthy", "effect": "Enemies beyond 12\" must pass AWR check to target this operative" },
    { "id": "spotter", "name": "Spotter", "effect": "Once per round, designate target: all friendly shooters gain +1 vs that target this round" },
    { "id": "resilient", "name": "Resilient", "effect": "First failed health check each game treated as pass (health still degrades)" },
    { "id": "tactician", "name": "Tactician", "effect": "Once per round, swap activation order with one friendly operative" },
    { "id": "rapid_assault", "name": "Rapid Assault", "effect": "After a successful Fight action, may move 3\"" },
    { "id": "breacher", "name": "Breacher", "effect": "Ignores cover penalty when shooting" },
    { "id": "field_repair", "name": "Field Repair", "effect": "Use Gear action: roll TEC to restore one health step to ally in contact" },
    { "id": "ghost", "name": "Ghost", "effect": "When hidden, enemies pass AWR check at -1 to spot" },
    { "id": "heavy_weapons", "name": "Heavy Weapons", "effect": "May carry one premium weapon without spending a gear slot" },
    { "id": "commander", "name": "Commander", "effect": "All friendly operatives within 6\" gain +1 to GRT for morale checks" }
  ],
  "behaviorTags": [
    {
      "id": "aggressive",
      "name": "Aggressive",
      "personality": "Closes distance, prefers melee",
      "table": [
        { "roll": "1-2", "action": "Move toward nearest visible enemy, Fight if in contact" },
        { "roll": "3-4", "action": "Move toward nearest enemy, Shoot if in line of sight" },
        { "roll": "5-6", "action": "Move toward nearest enemy + Fight if in contact (full move into melee)" }
      ]
    },
    {
      "id": "defensive",
      "name": "Defensive",
      "personality": "Holds position, shoots from cover",
      "table": [
        { "roll": "1-2", "action": "If in cover: Shoot nearest visible enemy. If not: move to nearest cover" },
        { "roll": "3-4", "action": "Hold position, Shoot nearest visible enemy" },
        { "roll": "5-6", "action": "Move to better cover within 6\", Shoot if possible" }
      ]
    },
    {
      "id": "cautious",
      "name": "Cautious",
      "personality": "Avoids risk, repositions",
      "table": [
        { "roll": "1-2", "action": "Move away from nearest enemy (maintain range), Shoot if able" },
        { "roll": "3-4", "action": "Move toward nearest objective, Interact if in contact" },
        { "roll": "5-6", "action": "Move to cover with best sight lines, Shoot nearest enemy" }
      ]
    },
    {
      "id": "flanker",
      "name": "Flanker",
      "personality": "Seeks angles, avoids frontline",
      "table": [
        { "roll": "1-2", "action": "Move toward flank/rear of nearest enemy, Shoot if able" },
        { "roll": "3-4", "action": "If behind cover with LOS to enemy rear: Shoot. Otherwise: reposition" },
        { "roll": "5-6", "action": "Move to closest unengaged enemy, Shoot or Fight" }
      ]
    }
  ],
  "scenarios": [
    { "id": "lockdown", "name": "Lockdown", "briefing": "Seize and hold critical locations (flexible: 1, 3, or 5 objective layouts)", "rounds": 6, "tableSize": "3x3" },
    { "id": "scorched_data", "name": "Scorched Data", "briefing": "Race to hack/destroy terminals before the enemy does", "rounds": 5, "tableSize": "3x3" },
    { "id": "dead_drop", "name": "Dead Drop", "briefing": "Retrieve intel packages scattered across the map, exfil to deployment zone", "rounds": 6, "tableSize": "3x3" },
    { "id": "headhunter", "name": "Headhunter", "briefing": "Eliminate high-value enemy targets", "rounds": 6, "tableSize": "3x3" },
    { "id": "crash_site", "name": "Crash Site", "briefing": "Recover cargo from a downed drone and deliver it home", "rounds": 6, "tableSize": "3x3" },
    { "id": "extraction", "name": "Extraction", "briefing": "Locate and extract a VIP from a hostile zone", "rounds": 6, "tableSize": "3x3" },
    { "id": "sweep_clear", "name": "Sweep & Clear", "briefing": "Eliminate all hostiles in a building complex (solo/co-op focused)", "rounds": 5, "tableSize": "2x2" }
  ]
}
```

- [ ] **Step 2: Commit**

```bash
git add NullSignal_Data.json
git commit -m "feat: add canonical game data JSON"
```

---

### Task 2: Forge — HTML Shell + Theme + React Scaffold

**Files:**
- Create: `NullSignal_Forge.html`

This task creates the app shell with the cyberpunk theme, React scaffold, embedded game data, and the main App component with navigation.

- [ ] **Step 1: Create the Forge HTML file**

Write `NullSignal_Forge.html` with:

**Head section:**
- Meta charset, viewport
- Title: "Null Signal — Warband Forge"
- `window.define = null` (disable AMD loader for UMD compat)
- CDN scripts: Tailwind CSS, React 18 UMD, ReactDOM 18 UMD, Babel standalone
- Google Fonts: Share Tech Mono (body), Orbitron (headings)
- CSS custom properties for cyberpunk theme:

```css
:root {
    --bg-black: #0a0a0f;
    --bg-dark: #12121a;
    --bg-panel: #1a1a2e;
    --bg-card: #16213e;
    --accent-cyan: #00d4ff;
    --accent-magenta: #ff006e;
    --accent-amber: #ffbe0b;
    --accent-green: #00f5a0;
    --accent-red: #ff3366;
    --text-primary: #e0e0e0;
    --text-dim: #8888aa;
    --text-bright: #ffffff;
    --border: #2a2a4a;
    --border-glow: rgba(0, 212, 255, 0.3);
}
body { font-family: 'Share Tech Mono', monospace; background: var(--bg-black); color: var(--text-primary); }
h1, h2, h3, .heading-font { font-family: 'Orbitron', sans-serif; }
```

- Responsive breakpoints for mobile (sidebar collapse at 768px)
- Subtle scanline overlay via CSS `::before` pseudo-element
- Scrollbar styling matching theme

**Body:**
- `<div id="root" class="h-full"></div>`
- `<script type="text/babel">` block containing:

**Game data constants** — embed the full contents of `NullSignal_Data.json` as `const GAME_DATA = { ... };`

**Helper constants:**
```jsx
const STATS = GAME_DATA.stats;
const STAT_LABELS = GAME_DATA.statLabels;
const TIERS = Object.fromEntries(GAME_DATA.tiers.map(t => [t.id, t]));
const WEAPONS = GAME_DATA.weapons;
const STANDARD_WEAPONS = WEAPONS.filter(w => !w.premium);
const PREMIUM_WEAPONS = WEAPONS.filter(w => w.premium);
const ABILITIES = GAME_DATA.abilities;
const DICE_TYPES = GAME_DATA.diceTypes;
```

**UUID generator:**
```jsx
const generateUUID = () => {
    if (typeof crypto !== 'undefined' && crypto.randomUUID) return crypto.randomUUID();
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
        const r = Math.random() * 16 | 0;
        return (c === 'x' ? r : (r & 0x3 | 0x8)).toString(16);
    });
};
```

**Factory functions:**
```jsx
const createOperative = (tierId) => ({
    id: generateUUID(),
    name: '',
    tier: tierId,
    stats: Object.fromEntries(STATS.map(s => [s, 'd6'])),
    weapon: tierId === 'recruit' ? 'rifle' : null,
    abilities: [],
});

const createWarband = () => ({
    id: generateUUID(),
    name: 'New Warband',
    budget: 10,
    operatives: [],
});
```

**localStorage helpers:**
```jsx
const STORAGE_INDEX = 'nullSignal_warbandIndex';
const STORAGE_PREFIX = 'nullSignal_warband_';

const loadIndex = () => {
    try { return JSON.parse(localStorage.getItem(STORAGE_INDEX)) || []; }
    catch { return []; }
};
const saveIndex = (index) => localStorage.setItem(STORAGE_INDEX, JSON.stringify(index));
const loadWarband = (id) => {
    try { return JSON.parse(localStorage.getItem(STORAGE_PREFIX + id)); }
    catch { return null; }
};
const saveWarband = (wb) => {
    localStorage.setItem(STORAGE_PREFIX + wb.id, JSON.stringify(wb));
    const index = loadIndex();
    const i = index.findIndex(e => e.id === wb.id);
    const entry = { id: wb.id, name: wb.name, lastModified: Date.now() };
    if (i >= 0) index[i] = entry; else index.push(entry);
    saveIndex(index);
};
const deleteWarband = (id) => {
    localStorage.removeItem(STORAGE_PREFIX + id);
    saveIndex(loadIndex().filter(e => e.id !== id));
};
```

**Export/Import helpers:**
```jsx
const exportJSON = (wb) => {
    const blob = new Blob([JSON.stringify(wb, null, 2)], { type: 'application/json' });
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = `${wb.name || 'warband'}.json`;
    a.click();
    URL.revokeObjectURL(a.href);
};
const importJSON = (file) => new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = (e) => { try { resolve(JSON.parse(e.target.result)); } catch { reject(new Error('Invalid JSON')); } };
    reader.onerror = () => reject(new Error('File read error'));
    reader.readAsText(file);
});
```

**Calculation helpers:**
```jsx
const getPointsUsed = (wb) => wb.operatives.reduce((sum, op) => sum + (TIERS[op.tier]?.cost || 0), 0);
const getPointsRemaining = (wb) => wb.budget - getPointsUsed(wb);
const getGearSlotsUsed = (op) => {
    const weapon = WEAPONS.find(w => w.id === op.weapon);
    const hasHeavyWeapons = op.abilities.includes('heavy_weapons');
    if (weapon?.premium && !hasHeavyWeapons) return weapon.gearCost || 0;
    return 0;
};
const getGearSlotsTotal = (op) => TIERS[op.tier]?.gearSlots || 0;
const getAbilitySlotsTotal = (op) => TIERS[op.tier]?.abilitySlots || 0;
const validateStatAssignment = (op) => {
    const tier = TIERS[op.tier];
    if (!tier) return false;
    const d8Count = STATS.filter(s => op.stats[s] === 'd8').length;
    return d8Count === tier.statSpread.d8Count;
};
```

**Minimal SVG Icon library** (inline, same pattern as Pulp Alley Forge — Plus, Trash, Save, Upload, Download, X, Edit, Search, ChevronDown, ArrowLeft, Users, Dice, Shield, Crosshair, Cpu, Eye).

**App component — scaffold only for now:**
```jsx
const App = () => {
    const [warbandIndex, setWarbandIndex] = useState(loadIndex());
    const [activeWarbandId, setActiveWarbandId] = useState(null);
    const [warband, setWarband] = useState(null);
    const [view, setView] = useState('list'); // 'list' | 'edit'

    // Auto-save on warband change
    useEffect(() => {
        if (warband) {
            saveWarband(warband);
            setWarbandIndex(loadIndex());
        }
    }, [warband]);

    const openWarband = (id) => {
        const wb = loadWarband(id);
        if (wb) { setWarband(wb); setActiveWarbandId(id); setView('edit'); }
    };

    const createNew = () => {
        const wb = createWarband();
        saveWarband(wb);
        setWarband(wb); setActiveWarbandId(wb.id); setView('edit');
        setWarbandIndex(loadIndex());
    };

    const deleteWb = (id) => {
        deleteWarband(id);
        setWarbandIndex(loadIndex());
        if (activeWarbandId === id) { setWarband(null); setActiveWarbandId(null); setView('list'); }
    };

    const backToList = () => { setView('list'); setWarband(null); setActiveWarbandId(null); };

    return (
        <div className="h-screen flex flex-col" style={{ background: 'var(--bg-black)' }}>
            <Header warband={warband} onBack={backToList} view={view} />
            <div className="flex-1 overflow-hidden">
                {view === 'list' ? (
                    <WarbandList index={warbandIndex} onOpen={openWarband} onCreate={createNew} onDelete={deleteWb} />
                ) : (
                    <WarbandEditor warband={warband} setWarband={setWarband} />
                )}
            </div>
        </div>
    );
};
```

**Header component:**
```jsx
const Header = ({ warband, onBack, view }) => (
    <header className="flex items-center justify-between px-4 py-3"
        style={{ background: 'var(--bg-dark)', borderBottom: '1px solid var(--border)' }}>
        <div className="flex items-center gap-3">
            {view === 'edit' && (
                <button onClick={onBack} className="p-1 rounded hover:bg-white/10">
                    <Icons.ArrowLeft size={20} />
                </button>
            )}
            <h1 className="heading-font text-lg tracking-widest" style={{ color: 'var(--accent-cyan)' }}>
                NULL SIGNAL
            </h1>
            <span className="text-xs" style={{ color: 'var(--text-dim)' }}>WARBAND FORGE</span>
        </div>
        {warband && (
            <div className="text-sm" style={{ color: 'var(--text-dim)' }}>
                {getPointsUsed(warband)}/{warband.budget} pts
            </div>
        )}
    </header>
);
```

**Placeholder WarbandList and WarbandEditor** (will be fleshed out in subsequent tasks):
```jsx
const WarbandList = ({ index, onOpen, onCreate, onDelete }) => (
    <div className="p-6 max-w-2xl mx-auto">
        <div className="flex justify-between items-center mb-6">
            <h2 className="heading-font text-xl" style={{ color: 'var(--accent-cyan)' }}>YOUR WARBANDS</h2>
            <button onClick={onCreate} className="flex items-center gap-2 px-4 py-2 rounded"
                style={{ background: 'var(--accent-cyan)', color: 'var(--bg-black)', fontWeight: 'bold' }}>
                <Icons.Plus size={16} /> New Warband
            </button>
        </div>
        {index.length === 0 ? (
            <p style={{ color: 'var(--text-dim)' }}>No warbands yet. Create one to get started.</p>
        ) : (
            <div className="space-y-2">
                {index.sort((a, b) => (b.lastModified || 0) - (a.lastModified || 0)).map(entry => (
                    <div key={entry.id} className="flex items-center justify-between p-3 rounded cursor-pointer hover:border-opacity-60"
                        style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)' }}
                        onClick={() => onOpen(entry.id)}>
                        <span>{entry.name}</span>
                        <button onClick={(e) => { e.stopPropagation(); onDelete(entry.id); }}
                            className="p-1 rounded hover:bg-red-500/20" style={{ color: 'var(--accent-red)' }}>
                            <Icons.Trash2 size={16} />
                        </button>
                    </div>
                ))}
            </div>
        )}
    </div>
);

const WarbandEditor = ({ warband, setWarband }) => (
    <div className="p-6">
        <p style={{ color: 'var(--text-dim)' }}>Editor coming soon...</p>
    </div>
);

ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```

- [ ] **Step 2: Verify in browser**

Open `NullSignal_Forge.html` in a browser. Verify:
- Cyberpunk dark theme renders (dark background, cyan header text)
- "YOUR WARBANDS" list page shows with "New Warband" button
- Clicking "New Warband" creates a warband and switches to the editor placeholder
- Back arrow returns to list
- Warband persists on page reload (check localStorage)
- Delete button removes the warband

- [ ] **Step 3: Commit**

```bash
git add NullSignal_Forge.html
git commit -m "feat: Forge app shell with theme, navigation, persistence"
```

---

### Task 3: Forge — Warband Editor (Name, Budget, Operative Management)

**Files:**
- Modify: `NullSignal_Forge.html` (replace the `WarbandEditor` placeholder)

- [ ] **Step 1: Implement WarbandEditor component**

Replace the placeholder `WarbandEditor` with the full editor. This includes:

**WarbandEditor** — the main editing view, two-column layout (operative list + detail panel):
```jsx
const WarbandEditor = ({ warband, setWarband }) => {
    const [selectedOpId, setSelectedOpId] = useState(null);
    const selectedOp = warband.operatives.find(op => op.id === selectedOpId);

    const updateWarband = (changes) => setWarband(prev => ({ ...prev, ...changes }));
    const updateOp = (opId, changes) => setWarband(prev => ({
        ...prev,
        operatives: prev.operatives.map(op =>
            op.id === opId ? { ...op, ...changes } : op
        ),
    }));
    const addOperative = (tierId) => {
        const op = createOperative(tierId);
        setWarband(prev => ({ ...prev, operatives: [...prev.operatives, op] }));
        setSelectedOpId(op.id);
    };
    const removeOperative = (opId) => {
        setWarband(prev => ({
            ...prev,
            operatives: prev.operatives.filter(op => op.id !== opId),
        }));
        if (selectedOpId === opId) setSelectedOpId(null);
    };

    const pointsUsed = getPointsUsed(warband);
    const pointsRemaining = getPointsRemaining(warband);
    const hasLeader = warband.operatives.some(op => op.tier === 'leader');

    return (
        <div className="h-full flex flex-col md:flex-row overflow-hidden">
            {/* Left panel: warband overview + operative list */}
            <div className="w-full md:w-80 flex-shrink-0 overflow-y-auto p-4 space-y-4"
                style={{ background: 'var(--bg-dark)', borderRight: '1px solid var(--border)' }}>

                {/* Warband name */}
                <input value={warband.name} onChange={(e) => updateWarband({ name: e.target.value })}
                    className="w-full px-3 py-2 rounded heading-font text-lg"
                    style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)',
                        color: 'var(--accent-cyan)', outline: 'none' }}
                    placeholder="Warband Name" />

                {/* Budget display */}
                <div className="flex justify-between items-center px-3 py-2 rounded"
                    style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)' }}>
                    <span style={{ color: 'var(--text-dim)' }}>Budget</span>
                    <span style={{ color: pointsRemaining < 0 ? 'var(--accent-red)' : 'var(--accent-green)' }}>
                        {pointsUsed} / {warband.budget} pts
                    </span>
                </div>

                {/* Budget adjustment */}
                <div className="flex items-center gap-2">
                    <span className="text-xs" style={{ color: 'var(--text-dim)' }}>Game Size:</span>
                    {[{ label: 'Small', pts: 6 }, { label: 'Standard', pts: 10 }, { label: 'Large', pts: 15 }].map(sz => (
                        <button key={sz.pts} onClick={() => updateWarband({ budget: sz.pts })}
                            className="px-2 py-1 text-xs rounded"
                            style={{
                                background: warband.budget === sz.pts ? 'var(--accent-cyan)' : 'var(--bg-panel)',
                                color: warband.budget === sz.pts ? 'var(--bg-black)' : 'var(--text-dim)',
                                border: '1px solid var(--border)',
                            }}>
                            {sz.label} ({sz.pts})
                        </button>
                    ))}
                </div>

                {/* Add operative buttons */}
                <div className="space-y-1">
                    <div className="text-xs uppercase tracking-wider mb-2" style={{ color: 'var(--text-dim)' }}>
                        Add Operative
                    </div>
                    {GAME_DATA.tiers.map(tier => {
                        const disabled = tier.id === 'leader' && hasLeader;
                        const canAfford = tier.cost <= pointsRemaining || tier.cost === 0;
                        return (
                            <button key={tier.id}
                                onClick={() => !disabled && canAfford && addOperative(tier.id)}
                                disabled={disabled || !canAfford}
                                className="w-full flex justify-between items-center px-3 py-2 rounded text-sm"
                                style={{
                                    background: 'var(--bg-panel)', border: '1px solid var(--border)',
                                    opacity: (disabled || !canAfford) ? 0.4 : 1,
                                    cursor: (disabled || !canAfford) ? 'not-allowed' : 'pointer',
                                }}>
                                <span>{tier.name}</span>
                                <span style={{ color: 'var(--text-dim)' }}>
                                    {tier.cost === 0 ? 'Free' : `${tier.cost} pts`}
                                </span>
                            </button>
                        );
                    })}
                </div>

                {/* Operative list */}
                <div className="space-y-1">
                    <div className="text-xs uppercase tracking-wider mb-2" style={{ color: 'var(--text-dim)' }}>
                        Roster ({warband.operatives.length} operatives)
                    </div>
                    {warband.operatives.map(op => (
                        <OperativeListItem key={op.id} op={op}
                            selected={selectedOpId === op.id}
                            onSelect={() => setSelectedOpId(op.id)}
                            onRemove={() => removeOperative(op.id)} />
                    ))}
                </div>
            </div>

            {/* Right panel: operative detail */}
            <div className="flex-1 overflow-y-auto p-4">
                {selectedOp ? (
                    <OperativeDetail op={selectedOp} warband={warband} updateOp={updateOp} />
                ) : (
                    <div className="flex items-center justify-center h-full"
                        style={{ color: 'var(--text-dim)' }}>
                        Select an operative or add one to begin
                    </div>
                )}
            </div>
        </div>
    );
};
```

**OperativeListItem** — clickable roster entry:
```jsx
const TIER_COLORS = {
    leader: 'var(--accent-amber)',
    specialist: 'var(--accent-cyan)',
    operative: 'var(--accent-green)',
    recruit: 'var(--text-dim)',
};

const OperativeListItem = ({ op, selected, onSelect, onRemove }) => (
    <div onClick={onSelect} className="flex items-center justify-between px-3 py-2 rounded cursor-pointer"
        style={{
            background: selected ? 'var(--bg-card)' : 'var(--bg-panel)',
            border: selected ? '1px solid var(--accent-cyan)' : '1px solid var(--border)',
            boxShadow: selected ? '0 0 8px var(--border-glow)' : 'none',
        }}>
        <div>
            <span className="text-xs font-bold mr-2" style={{ color: TIER_COLORS[op.tier] }}>
                {TIERS[op.tier]?.name?.toUpperCase()}
            </span>
            <span>{op.name || 'Unnamed'}</span>
        </div>
        <button onClick={(e) => { e.stopPropagation(); onRemove(); }}
            className="p-1 rounded hover:bg-red-500/20" style={{ color: 'var(--accent-red)' }}>
            <Icons.Trash2 size={14} />
        </button>
    </div>
);
```

**OperativeDetail** — placeholder for now, just name + tier display:
```jsx
const OperativeDetail = ({ op, warband, updateOp }) => {
    const tier = TIERS[op.tier];
    return (
        <div className="space-y-4 max-w-xl">
            <div className="flex items-center gap-3">
                <span className="heading-font text-xs uppercase tracking-wider px-2 py-1 rounded"
                    style={{ background: TIER_COLORS[op.tier], color: 'var(--bg-black)' }}>
                    {tier?.name}
                </span>
                <span className="text-xs" style={{ color: 'var(--text-dim)' }}>
                    Health: {tier?.health} | Gear: {getGearSlotsUsed(op)}/{getGearSlotsTotal(op)} |
                    Abilities: {op.abilities.length}/{getAbilitySlotsTotal(op)}
                </span>
            </div>
            <input value={op.name} onChange={(e) => updateOp(op.id, { name: e.target.value })}
                className="w-full px-3 py-2 rounded"
                style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)',
                    color: 'var(--text-bright)', outline: 'none' }}
                placeholder="Operative Name" />

            <StatAssignment op={op} updateOp={updateOp} />
            <WeaponSelect op={op} updateOp={updateOp} />
            <AbilitySelect op={op} updateOp={updateOp} />
        </div>
    );
};
```

Placeholders for `StatAssignment`, `WeaponSelect`, `AbilitySelect`:
```jsx
const StatAssignment = ({ op, updateOp }) => (
    <div className="p-3 rounded" style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)' }}>
        <div className="text-xs uppercase tracking-wider mb-2" style={{ color: 'var(--text-dim)' }}>Stats — assign next task</div>
    </div>
);
const WeaponSelect = ({ op, updateOp }) => (
    <div className="p-3 rounded" style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)' }}>
        <div className="text-xs uppercase tracking-wider mb-2" style={{ color: 'var(--text-dim)' }}>Weapon — assign next task</div>
    </div>
);
const AbilitySelect = ({ op, updateOp }) => (
    <div className="p-3 rounded" style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)' }}>
        <div className="text-xs uppercase tracking-wider mb-2" style={{ color: 'var(--text-dim)' }}>Abilities — assign next task</div>
    </div>
);
```

- [ ] **Step 2: Verify in browser**

Open `NullSignal_Forge.html`. Verify:
- Two-column layout: left sidebar with warband controls, right panel for operative detail
- Can name the warband (input at top of sidebar)
- Budget display shows points used/total, changes color when over budget
- Game size buttons switch budget (6/10/15)
- "Add Operative" buttons for each tier — Leader disabled after one is added, tier disabled if can't afford
- Clicking an operative in the roster list highlights it and shows detail panel
- Delete button on each operative removes it
- Changes persist on page reload

- [ ] **Step 3: Commit**

```bash
git add NullSignal_Forge.html
git commit -m "feat: Forge warband editor with roster management"
```

---

### Task 4: Forge — Stat Assignment

**Files:**
- Modify: `NullSignal_Forge.html` (replace `StatAssignment` placeholder)

- [ ] **Step 1: Implement StatAssignment component**

Replace the `StatAssignment` placeholder. The player clicks stats to toggle them between d6 and d8 until the correct number of d8s are assigned per tier.

```jsx
const StatAssignment = ({ op, updateOp }) => {
    const tier = TIERS[op.tier];
    const d8Needed = tier?.statSpread.d8Count || 0;
    const d8Assigned = STATS.filter(s => op.stats[s] === 'd8').length;
    const isComplete = d8Assigned === d8Needed;

    const toggleStat = (stat) => {
        if (d8Needed === 0) return; // Recruits: all d6, no toggling
        const current = op.stats[stat];
        if (current === 'd8') {
            // Downgrade to d6
            updateOp(op.id, { stats: { ...op.stats, [stat]: 'd6' } });
        } else if (d8Assigned < d8Needed) {
            // Upgrade to d8
            updateOp(op.id, { stats: { ...op.stats, [stat]: 'd8' } });
        }
    };

    const STAT_ICONS = { CMB: Icons.Crosshair, REF: Icons.Shield, GRT: Icons.Dice, TEC: Icons.Cpu, AWR: Icons.Eye };

    return (
        <div className="p-3 rounded" style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)' }}>
            <div className="flex justify-between items-center mb-3">
                <span className="text-xs uppercase tracking-wider" style={{ color: 'var(--text-dim)' }}>
                    Stats
                </span>
                {d8Needed > 0 && (
                    <span className="text-xs" style={{ color: isComplete ? 'var(--accent-green)' : 'var(--accent-amber)' }}>
                        {d8Assigned}/{d8Needed} d8 assigned
                    </span>
                )}
            </div>
            <div className="grid grid-cols-5 gap-2">
                {STATS.map(stat => {
                    const die = op.stats[stat];
                    const isD8 = die === 'd8';
                    const canUpgrade = !isD8 && d8Assigned < d8Needed;
                    const Icon = STAT_ICONS[stat];
                    return (
                        <button key={stat} onClick={() => toggleStat(stat)}
                            className="flex flex-col items-center p-2 rounded transition-all"
                            style={{
                                background: isD8 ? 'rgba(0,212,255,0.15)' : 'var(--bg-card)',
                                border: isD8 ? '1px solid var(--accent-cyan)' : '1px solid var(--border)',
                                cursor: d8Needed === 0 ? 'default' : 'pointer',
                                opacity: (d8Needed === 0 || (!isD8 && !canUpgrade)) ? 0.5 : 1,
                            }}>
                            {Icon && <Icon size={16} className="mb-1" style={{ color: isD8 ? 'var(--accent-cyan)' : 'var(--text-dim)' }} />}
                            <span className="text-xs font-bold">{stat}</span>
                            <span className="text-xs mt-1" style={{
                                color: isD8 ? 'var(--accent-cyan)' : 'var(--text-dim)',
                                fontWeight: isD8 ? 'bold' : 'normal',
                            }}>{die}</span>
                        </button>
                    );
                })}
            </div>
            {d8Needed === 0 && (
                <p className="text-xs mt-2" style={{ color: 'var(--text-dim)' }}>
                    Recruits have all stats at d6.
                </p>
            )}
        </div>
    );
};
```

- [ ] **Step 2: Verify in browser**

- Leader shows 5 stat buttons, 3 must be toggled to d8 — counter shows "3/3 d8 assigned" when complete
- Specialist: 2 d8 slots
- Operative: 1 d8 slot
- Recruit: all greyed out at d6, no clicking
- Toggling a d8 stat back to d6 frees a slot
- Can't assign more d8s than the tier allows

- [ ] **Step 3: Commit**

```bash
git add NullSignal_Forge.html
git commit -m "feat: Forge stat assignment with tier-based d8 slots"
```

---

### Task 5: Forge — Weapon Selection

**Files:**
- Modify: `NullSignal_Forge.html` (replace `WeaponSelect` placeholder)

- [ ] **Step 1: Implement WeaponSelect component**

```jsx
const WeaponSelect = ({ op, updateOp }) => {
    const tier = TIERS[op.tier];
    const gearTotal = getGearSlotsTotal(op);
    const gearUsed = getGearSlotsUsed(op);
    const hasHeavyWeapons = op.abilities.includes('heavy_weapons');
    const currentWeapon = WEAPONS.find(w => w.id === op.weapon);

    const canEquip = (weapon) => {
        if (!weapon.premium) return true;
        if (hasHeavyWeapons) return true;
        return gearTotal > 0 && (weapon.gearCost || 0) <= (gearTotal - gearUsed + (currentWeapon?.premium && !hasHeavyWeapons ? currentWeapon.gearCost || 0 : 0));
    };

    const selectWeapon = (weaponId) => {
        updateOp(op.id, { weapon: weaponId });
    };

    return (
        <div className="p-3 rounded" style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)' }}>
            <div className="flex justify-between items-center mb-3">
                <span className="text-xs uppercase tracking-wider" style={{ color: 'var(--text-dim)' }}>Weapon</span>
                {gearTotal > 0 && (
                    <span className="text-xs" style={{ color: 'var(--text-dim)' }}>
                        Gear: {gearUsed}/{gearTotal}
                    </span>
                )}
            </div>

            <div className="text-xs uppercase tracking-wider mb-1" style={{ color: 'var(--accent-green)' }}>
                Standard (Free)
            </div>
            <div className="space-y-1 mb-3">
                {STANDARD_WEAPONS.map(w => (
                    <WeaponRow key={w.id} weapon={w} selected={op.weapon === w.id}
                        enabled={true} onSelect={() => selectWeapon(w.id)} />
                ))}
            </div>

            <div className="text-xs uppercase tracking-wider mb-1" style={{ color: 'var(--accent-amber)' }}>
                Premium (1 Gear Slot)
            </div>
            <div className="space-y-1">
                {PREMIUM_WEAPONS.map(w => (
                    <WeaponRow key={w.id} weapon={w} selected={op.weapon === w.id}
                        enabled={canEquip(w)} onSelect={() => canEquip(w) && selectWeapon(w.id)} />
                ))}
            </div>
        </div>
    );
};

const WeaponRow = ({ weapon, selected, enabled, onSelect }) => (
    <div onClick={onSelect} className="flex items-center justify-between px-2 py-1.5 rounded cursor-pointer text-sm"
        style={{
            background: selected ? 'rgba(0,212,255,0.1)' : 'transparent',
            border: selected ? '1px solid var(--accent-cyan)' : '1px solid transparent',
            opacity: enabled ? 1 : 0.35,
            cursor: enabled ? 'pointer' : 'not-allowed',
        }}>
        <div>
            <span style={{ color: selected ? 'var(--accent-cyan)' : 'var(--text-primary)' }}>
                {weapon.name}
            </span>
            <span className="ml-2 text-xs" style={{ color: 'var(--text-dim)' }}>
                {weapon.attackDice}d · {weapon.melee ? 'Melee' : `${weapon.rangeBand}"`}
            </span>
        </div>
        <span className="text-xs" style={{ color: 'var(--text-dim)' }}>
            {weapon.trait}
        </span>
    </div>
);
```

- [ ] **Step 2: Verify in browser**

- Standard weapons shown for all operatives, selectable
- Premium weapons greyed out for operatives with 0 gear slots (Operative, Recruit)
- Leader (2 gear slots) can pick premium weapons
- Specialist (1 gear slot) can pick 1 premium weapon
- Selecting a weapon highlights it with cyan border
- "Heavy Weapons" ability (once implemented) bypasses gear cost — verify logic is in place

- [ ] **Step 3: Commit**

```bash
git add NullSignal_Forge.html
git commit -m "feat: Forge weapon selection with gear slot tracking"
```

---

### Task 6: Forge — Ability Selection

**Files:**
- Modify: `NullSignal_Forge.html` (replace `AbilitySelect` placeholder)

- [ ] **Step 1: Implement AbilitySelect component**

```jsx
const AbilitySelect = ({ op, updateOp }) => {
    const maxAbilities = getAbilitySlotsTotal(op);
    if (maxAbilities === 0) return null; // Operatives and Recruits get no abilities

    const currentCount = op.abilities.length;

    const toggleAbility = (abilityId) => {
        const has = op.abilities.includes(abilityId);
        if (has) {
            const newAbilities = op.abilities.filter(a => a !== abilityId);
            // If removing heavy_weapons, check if premium weapon still valid
            let weaponUpdate = {};
            if (abilityId === 'heavy_weapons') {
                const weapon = WEAPONS.find(w => w.id === op.weapon);
                if (weapon?.premium && getGearSlotsTotal(op) < (weapon.gearCost || 0)) {
                    weaponUpdate = { weapon: 'rifle' }; // Reset to default
                }
            }
            updateOp(op.id, { abilities: newAbilities, ...weaponUpdate });
        } else if (currentCount < maxAbilities) {
            updateOp(op.id, { abilities: [...op.abilities, abilityId] });
        }
    };

    return (
        <div className="p-3 rounded" style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)' }}>
            <div className="flex justify-between items-center mb-3">
                <span className="text-xs uppercase tracking-wider" style={{ color: 'var(--text-dim)' }}>Abilities</span>
                <span className="text-xs" style={{
                    color: currentCount === maxAbilities ? 'var(--accent-green)' : 'var(--accent-amber)'
                }}>
                    {currentCount}/{maxAbilities} selected
                </span>
            </div>
            <div className="space-y-1">
                {ABILITIES.map(ab => {
                    const selected = op.abilities.includes(ab.id);
                    const canSelect = selected || currentCount < maxAbilities;
                    return (
                        <div key={ab.id} onClick={() => canSelect && toggleAbility(ab.id)}
                            className="px-2 py-1.5 rounded text-sm"
                            style={{
                                background: selected ? 'rgba(0,212,255,0.1)' : 'transparent',
                                border: selected ? '1px solid var(--accent-cyan)' : '1px solid transparent',
                                cursor: canSelect ? 'pointer' : 'not-allowed',
                                opacity: canSelect ? 1 : 0.35,
                            }}>
                            <div style={{ color: selected ? 'var(--accent-cyan)' : 'var(--text-primary)' }}>
                                {ab.name}
                            </div>
                            <div className="text-xs" style={{ color: 'var(--text-dim)' }}>
                                {ab.effect}
                            </div>
                        </div>
                    );
                })}
            </div>
        </div>
    );
};
```

- [ ] **Step 2: Verify in browser**

- Ability section hidden for Operatives and Recruits
- Leader sees all 17 abilities, can pick 2 (counter: "2/2 selected" when done)
- Specialist can pick 1
- Selecting "Heavy Weapons" allows a premium weapon even with 0 natural gear slots — verify by giving a Specialist Heavy Weapons then checking premium weapon availability
- Deselecting "Heavy Weapons" resets weapon to Rifle if current weapon was premium and no longer affordable

- [ ] **Step 3: Commit**

```bash
git add NullSignal_Forge.html
git commit -m "feat: Forge ability selection with Heavy Weapons interaction"
```

---

### Task 7: Forge — Export/Import + Print

**Files:**
- Modify: `NullSignal_Forge.html` (add toolbar buttons to Header, add print function)

- [ ] **Step 1: Add export/import/print to the Header and WarbandList**

Update `Header` to include toolbar buttons when editing:
```jsx
const Header = ({ warband, onBack, view, onExport, onImport, onPrint }) => {
    const fileRef = useRef(null);
    return (
        <header className="flex items-center justify-between px-4 py-3"
            style={{ background: 'var(--bg-dark)', borderBottom: '1px solid var(--border)' }}>
            <div className="flex items-center gap-3">
                {view === 'edit' && (
                    <button onClick={onBack} className="p-1 rounded hover:bg-white/10">
                        <Icons.ArrowLeft size={20} />
                    </button>
                )}
                <h1 className="heading-font text-lg tracking-widest" style={{ color: 'var(--accent-cyan)' }}>
                    NULL SIGNAL
                </h1>
                <span className="text-xs" style={{ color: 'var(--text-dim)' }}>WARBAND FORGE</span>
            </div>
            <div className="flex items-center gap-2">
                {view === 'list' && (
                    <>
                        <input type="file" accept=".json" ref={fileRef} className="hidden"
                            onChange={(e) => { if (e.target.files[0]) onImport(e.target.files[0]); e.target.value = ''; }} />
                        <button onClick={() => fileRef.current?.click()}
                            className="p-2 rounded hover:bg-white/10" title="Import Warband">
                            <Icons.Upload size={18} style={{ color: 'var(--text-dim)' }} />
                        </button>
                    </>
                )}
                {view === 'edit' && warband && (
                    <>
                        <button onClick={onExport} className="p-2 rounded hover:bg-white/10" title="Export JSON">
                            <Icons.Download size={18} style={{ color: 'var(--text-dim)' }} />
                        </button>
                        <button onClick={onPrint} className="p-2 rounded hover:bg-white/10" title="Print Roster">
                            <Icons.Printer size={18} style={{ color: 'var(--text-dim)' }} />
                        </button>
                        <span className="text-sm" style={{ color: 'var(--text-dim)' }}>
                            {getPointsUsed(warband)}/{warband.budget} pts
                        </span>
                    </>
                )}
            </div>
        </header>
    );
};
```

**Print function** — opens a clean print-friendly window with the roster:
```jsx
const printRoster = (warband) => {
    const ops = warband.operatives.map(op => {
        const tier = TIERS[op.tier];
        const weapon = WEAPONS.find(w => w.id === op.weapon);
        const abilities = op.abilities.map(aId => ABILITIES.find(a => a.id === aId)).filter(Boolean);
        return { ...op, tierData: tier, weaponData: weapon, abilityData: abilities };
    });

    const esc = (s) => String(s ?? '').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');

    const html = `<!DOCTYPE html><html><head><meta charset="UTF-8">
    <title>${esc(warband.name)} — Null Signal Roster</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Orbitron:wght@400;700&display=swap');
        @page { margin: 1cm; }
        * { box-sizing: border-box; }
        body { font-family: 'Share Tech Mono', monospace; color: #000; background: #fff; padding: 10px; margin: 0; font-size: 11px; }
        h1 { font-family: 'Orbitron', sans-serif; font-size: 18px; margin: 0 0 4px; }
        .meta { font-size: 10px; color: #666; margin-bottom: 12px; border-bottom: 2px solid #000; padding-bottom: 6px; }
        .cards { display: flex; flex-wrap: wrap; gap: 4mm; }
        .card { width: 63mm; border: 1.5px solid #000; border-radius: 3px; padding: 0; break-inside: avoid; font-size: 9px; }
        .card-head { display: flex; justify-content: space-between; padding: 3px 5px; border-bottom: 1px solid #999; font-weight: bold; }
        .card-body { padding: 4px 5px; }
        .stat-row { display: flex; justify-content: space-between; padding: 1px 0; }
        .stat-label { font-weight: bold; }
        .weapon-row { padding: 3px 5px; border-top: 1px solid #999; font-size: 8px; }
        .ability-row { padding: 2px 5px; border-top: 1px solid #ddd; font-size: 8px; }
        .health-row { display: flex; border-top: 1.5px solid #000; font-size: 8px; font-weight: bold; }
        .health-cell { flex: 1; text-align: center; padding: 2px; border-left: 1px solid #999; }
        .health-cell:first-child { border-left: none; }
    </style></head><body>
    <h1>${esc(warband.name)}</h1>
    <div class="meta">${ops.length} operatives · ${getPointsUsed(warband)}/${warband.budget} pts</div>
    <div class="cards">${ops.map(op => {
        const healthTrack = GAME_DATA.healthTrack.slice(GAME_DATA.healthTrack.indexOf(op.tierData.health));
        return `<div class="card">
            <div class="card-head">
                <span>${esc(op.tierData.name.toUpperCase())}</span>
                <span>${esc(op.name || 'Unnamed')}</span>
            </div>
            <div class="card-body">
                ${STATS.map(s => `<div class="stat-row"><span class="stat-label">${s}</span><span>${op.stats[s]}</span></div>`).join('')}
            </div>
            ${op.weaponData ? `<div class="weapon-row"><b>${esc(op.weaponData.name)}</b> — ${op.weaponData.attackDice}d, ${op.weaponData.melee ? 'Melee' : op.weaponData.rangeBand + '"'} · ${esc(op.weaponData.trait)}</div>` : ''}
            ${op.abilityData.length > 0 ? op.abilityData.map(ab => `<div class="ability-row"><b>${esc(ab.name)}</b>: ${esc(ab.effect)}</div>`).join('') : ''}
            <div class="health-row">${healthTrack.map(h => `<div class="health-cell">${h}</div>`).join('')}</div>
        </div>`;
    }).join('')}</div></body></html>`;

    const win = window.open('', '_blank');
    if (!win) { alert('Please allow popups to print.'); return; }
    win.document.write(html);
    win.document.close();
    win.onload = () => { win.print(); win.onafterprint = () => win.close(); };
};
```

Wire up in `App`:
```jsx
// In App component, add handlers:
const handleExport = () => warband && exportJSON(warband);
const handleImport = async (file) => {
    try {
        const data = await importJSON(file);
        if (data.id && data.operatives) {
            data.id = generateUUID(); // New ID to avoid collisions
            saveWarband(data);
            setWarbandIndex(loadIndex());
            openWarband(data.id);
        }
    } catch (e) { alert('Import failed: ' + e.message); }
};
const handlePrint = () => warband && printRoster(warband);

// Pass to Header:
<Header warband={warband} onBack={backToList} view={view}
    onExport={handleExport} onImport={handleImport} onPrint={handlePrint} />
```

- [ ] **Step 2: Verify in browser**

- Export downloads a `.json` file with warband data
- Import loads a warband from JSON and opens it
- Print opens a new window with clean roster cards, triggers browser print dialog
- Roster cards show operative name, tier, stats, weapon, abilities, health track

- [ ] **Step 3: Commit**

```bash
git add NullSignal_Forge.html
git commit -m "feat: Forge export/import JSON and print roster"
```

---

### Task 8: Forge — Mobile Responsive Polish

**Files:**
- Modify: `NullSignal_Forge.html` (CSS and layout adjustments)

- [ ] **Step 1: Add mobile layout**

Add CSS for mobile (in the `<style>` block):
```css
@media (max-width: 768px) {
    .md\\:flex-row { flex-direction: column !important; }
    .md\\:w-80 { width: 100% !important; }
    select, button:not(.no-min-height), input { min-height: 44px; }
}
```

Add a mobile toggle to the `WarbandEditor` so on small screens the sidebar becomes a collapsible panel:
- When an operative is selected on mobile, show only the detail panel with a back-to-roster button
- Roster list takes full width on mobile

- [ ] **Step 2: Verify in browser**

- Resize to mobile width (< 768px): layout stacks vertically
- Sidebar takes full width, tapping an operative shows detail full-screen
- All touch targets are at least 44px
- Print still works on mobile

- [ ] **Step 3: Commit**

```bash
git add NullSignal_Forge.html
git commit -m "feat: Forge mobile responsive layout"
```

---

## Phase 2: Rules Webapp

### Task 9: Rules Webapp — Complete Implementation

**Files:**
- Create: `NullSignal_Rules.html`

The Rules webapp is a navigable, searchable rulebook. It's content-heavy but structurally simple: a sidebar nav + scrollable content sections + a quick-reference collapsible panel.

- [ ] **Step 1: Create the Rules webapp**

Write `NullSignal_Rules.html` with the same CDN stack and cyberpunk theme as the Forge.

**Structure:**
- Left sidebar: table of contents with section links (Core Stats, Health System, Combat, Turn Structure, Warband Building, Weapons, Abilities, Solo/Co-op AI, Scenarios, Quick Reference)
- Main area: all rules rendered as HTML content, each section with an `id` for anchor linking
- Sticky search bar at top — filters sections by keyword
- Quick Reference panel at bottom — collapsible, contains condensed tables (weapon stats, ability list, health track, AI behavior tables)
- Print button that renders the full rulebook in a print-friendly format

**Content:** Render every section from the design spec as HTML. Each rule, table, and example from `2026-04-10-null-signal-design.md` should appear in the webapp. Embed `GAME_DATA` for the reference tables (weapons, abilities, AI tables).

**Key components:**
- `App` — manages active section, search filter
- `Sidebar` — table of contents, clickable section links, highlights active
- `RulesContent` — renders all sections as scrollable HTML
- `QuickReference` — collapsible bottom panel with condensed tables
- `SearchBar` — filters visible sections

Mobile: sidebar collapses to a hamburger menu.

- [ ] **Step 2: Verify in browser**

- All rules sections visible and navigable
- TOC sidebar links scroll to correct sections
- Search filters sections
- Quick Reference panel toggles open/closed
- Weapon table, ability table, health track, and AI behavior tables all render correctly
- Print produces clean output
- Mobile: hamburger menu, readable on phone

- [ ] **Step 3: Commit**

```bash
git add NullSignal_Rules.html
git commit -m "feat: Rules webapp with full rulebook and quick reference"
```

---

## Phase 3: Solo Engine

### Task 10: Solo Engine — Complete Implementation

**Files:**
- Create: `NullSignal_SoloEngine.html`

The Solo Engine helps run AI opponents during solo/co-op games. It's a turn-management tool with dice rolling and behavior resolution.

- [ ] **Step 1: Create the Solo Engine**

Write `NullSignal_SoloEngine.html` with the same CDN stack and cyberpunk theme.

**Structure:**
- **AI Warband Setup:** Build an AI warband using the same tier system. For each operative, assign a behavior tag (Aggressive/Defensive/Cautious/Flanker). Option to auto-assign via d6 roll.
- **Turn Tracker:** Track current round (1–6), current activation. Show which AI operative activates next.
- **Behavior Roller:** For the current AI operative, display their behavior tag and roll the d6 reaction table. Show the result action clearly.
- **Target Priority Helper:** When the AI needs to pick a target, display the priority rules (nearest → lowest health → holding objective) as a checklist.
- **Health Tracker:** Track health die degradation for each AI operative (d10/d8/d6/DOWN/OUT).

**Key components:**
- `App` — manages AI warband state, round counter
- `AISetup` — build AI warband, assign tiers + behavior tags
- `TurnTracker` — round counter, activation order
- `BehaviorRoller` — rolls d6, displays result from the operative's behavior table with animated dice
- `TargetPriority` — displays targeting rules
- `HealthTracker` — visual health die tracker for each AI operative

**Dice animation:** Simple CSS animation — show a spinning d6 that settles on the result. Use a `setTimeout` of ~500ms for the "roll" feel.

localStorage persistence: save AI warband setup so it persists between sessions.

- [ ] **Step 2: Verify in browser**

- Can build an AI warband (add operatives, assign behavior tags)
- Turn tracker advances rounds and activations
- Behavior roller shows correct table for each tag, animates d6, displays result action
- Health tracker tracks degradation per operative
- State persists on reload
- Mobile responsive

- [ ] **Step 3: Commit**

```bash
git add NullSignal_SoloEngine.html
git commit -m "feat: Solo Engine with AI behavior roller and turn tracker"
```

---

## Phase 4: Final Polish

### Task 11: Cross-App Consistency + Final Commit

**Files:**
- Modify: All three HTML files

- [ ] **Step 1: Ensure consistent styling and data across all three apps**

- Same CSS custom properties (cyberpunk theme) in all three
- Same `GAME_DATA` embedded in all three
- Same font imports
- Cross-link between apps: each app's header has links to the other two (e.g., "Rules" and "Solo Engine" links in the Forge header)

- [ ] **Step 2: Verify all three apps in browser**

- Open each app, verify theme consistency
- Cross-links work
- All data matches (same weapons, abilities, tiers across all three)
- Mobile responsive on all three

- [ ] **Step 3: Final commit**

```bash
git add NullSignal_Forge.html NullSignal_Rules.html NullSignal_SoloEngine.html
git commit -m "polish: cross-app consistency, navigation links, theme alignment"
```

---

## Self-Review Checklist

### Spec Coverage
| Spec Section | Plan Task |
|---|---|
| 1. Core Stats | Task 1 (data), Task 4 (stat assignment) |
| 2. Health System | Task 1 (data), Task 7 (print health track), Task 10 (health tracker) |
| 3. Combat | Task 9 (rules webapp) |
| 4. Turn Structure | Task 9 (rules), Task 10 (turn tracker) |
| 5. Warband Building | Tasks 2-6 (full forge) |
| 6. Weapons | Task 1 (data), Task 5 (weapon select) |
| 7. Abilities | Task 1 (data), Task 6 (ability select) |
| 8. Solo/Co-op AI | Task 1 (data), Task 10 (solo engine) |
| 9. Scenarios | Task 1 (data), Task 9 (rules) |
| 10. Deliverables | Tasks 2-8 (Forge), Task 9 (Rules), Task 10 (Solo Engine) |
| 11. Design Principles | Embedded throughout |

### Type/Name Consistency
- `GAME_DATA` used consistently across all three apps
- `TIERS`, `WEAPONS`, `ABILITIES`, `STATS` derived identically in each app
- `createOperative()`, `createWarband()` factory functions used only in Forge
- `generateUUID()` used in Forge and Solo Engine
- localStorage keys: `nullSignal_warband*` (Forge), `nullSignal_aiWarband*` (Solo Engine)
