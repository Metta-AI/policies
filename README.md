# Metta AI — Policies

A collection of self-improving AI policy frameworks for game-playing agents.

---

## Cyborg Policy Framework

**Location:** [`policies/cyborg-policy-framework/`](policies/cyborg-policy-framework/)

A generalizable framework for building game-playing AI agents that improve themselves
through a closed-loop pipeline: play → analyze → evolve → repeat. The framework
separates game-agnostic infrastructure from game-specific logic using a plugin
architecture.

### The Evolution Loop

The agent plays games, learns from mistakes, and rewrites its own code — automatically.

<p align="center">
  <img src="assets/evolution-loop.png" alt="Evolution Loop — Play, Analyze, Evolve, Score" width="700" />
</p>

### What It Does

An agent plays a game. After the game, an LLM (like Claude) analyzes what happened,
scores performance, and identifies mistakes. Then a code evolution agent makes targeted
edits to the agent's decision logic and strategy wiki. The improved agent plays the next
game. Over hundreds of games, the agent compounds small improvements into significant
capability gains.

### Plugin Architecture

Game-agnostic framework core connects to any game through a plugin interface.

<p align="center">
  <img src="assets/plugin-architecture.png" alt="Plugin Architecture — Framework Core + Game Plugins" width="700" />
</p>

### Key Features

- **Plugin architecture** — game-agnostic core + game-specific plugins
- **Three-tier memory** — working (volatile), episodic (event log), strategic (learned facts)
- **LLM strategic advisor** — background thread consults an LLM when significant events occur
- **Self-improving code** — evolution agent makes surgical `str_replace` edits to the brain
- **Transferable wiki** — skills and strategy docs that apply across games
- **Weekly lifecycle** — fork from base, evolve during the week, promote successes, reset
- **Score tracking** — automatic difficulty escalation when performance plateaus

### Quick Start

```bash
# 1. Copy the template to create a new game
cp -r policies/cyborg-policy-framework/games/new_game/ \
      policies/cyborg-policy-framework/games/settlers/

# 2. Implement the 5 core files:
#    brain.py     — per-tick decision logic
#    triggers.py  — event detection for LLM consultation
#    harness.py   — LLM context building and directive parsing
#    policy.py    — bridge to your game engine
#    game_config.py — editable files, prompts, evolution config

# 3. Fork a weekly policy branch
python policies/cyborg-policy-framework/scripts/policy_manager.py fork --game settlers

# 4. Run the evolution loop
./policies/cyborg-policy-framework/scripts/continuous_loop.sh settlers

# 5. At week end: report, promote, reset
python policies/cyborg-policy-framework/scripts/reporting_agent.py --game settlers
python policies/cyborg-policy-framework/scripts/promote_patterns.py --game settlers
python policies/cyborg-policy-framework/scripts/policy_manager.py reset --game settlers
```

### Architecture

```
cyborg-policy-framework/
├── framework/              # Game-agnostic infrastructure
│   ├── base_brain.py       # ABC: decide(snapshot) → Command
│   ├── base_policy.py      # perceive → decide → execute loop
│   ├── base_harness.py     # LLM daemon thread + directive mailbox
│   ├── base_triggers.py    # Priority-based event detection
│   ├── base_memory.py      # 3-tier memory system
│   ├── base_evolution.py   # str_replace tool loop + git workflow
│   ├── base_analysis.py    # Post-game LLM scoring pipeline
│   ├── score_tracker.py    # Rolling avg + difficulty escalation
│   ├── providers.py        # LLM provider abstraction (Bedrock, OpenRouter, Anthropic)
│   ├── types.py            # Command, Directive, GameConfig
│   └── wiki/               # Transferable knowledge base
│       ├── skills/         # Frame parsing, movement, memory, LLM patterns
│       ├── strategy/       # Opening, resources, adaptation, mistakes
│       └── mechanics/      # Evolution loop, triggers, memory tiers
│
├── games/                  # Game plugins
│   └── new_game/           # Template — copy to create a new game
│
├── scripts/                # Pipeline orchestration
│   ├── continuous_loop.sh  # Play → analyze → evolve → repeat
│   ├── record_score.py     # Score tracking + escalation
│   ├── policy_manager.py   # Weekly fork/reset lifecycle
│   ├── reporting_agent.py  # Change documentation
│   └── promote_patterns.py # Extract + promote successful patterns
│
└── docs/
    └── POLICY_GENERATION.md  # Full architecture documentation
```

### The Evolution Pipeline

```
Game 1:  Score 3/10  — "Agent never switched to builder role"
  → Evolution: Lower role_switch_threshold from 200 to 150

Game 2:  Score 5/10  — "Switched too early, no resources gathered"
  → Evolution: Add guard clause checking resources > 0 before switch

Game 3:  Score 6/10  — "Good gathering but wrong priority"
  → Evolution: Update wiki/strategy/opening.md with priority rules

Game 4:  Score 7/10  — "Good early game but endgame collapsed"
  → Evolution: Add late_game check in scripted_role_check()

Game 5+: Score 8/10  → Escalate to longer games (more steps)
```

### Transferable Skills

The framework wiki documents skills that transfer across any game:

| Skill | File | What It Teaches |
|---|---|---|
| Frame Parsing | `wiki/skills/frame-parsing.md` | Observations → typed snapshot dicts |
| Movement | `wiki/skills/movement.md` | A* pathfinding, frontier exploration, stuck detection |
| Memory | `wiki/skills/memory-management.md` | Three-tier memory (working, episodic, strategic) |
| LLM Integration | `wiki/skills/llm-integration.md` | Consultation model, context building, token budgets |
| Role Selection | `wiki/skills/role-selection.md` | Role switching, hysteresis, scripted vs LLM transitions |

### Weekly Lifecycle

Each week: fork, play, evolve, promote the best patterns, reset. Knowledge compounds.

<p align="center">
  <img src="assets/weekly-lifecycle.png" alt="Weekly Lifecycle — Fork, Play+Evolve, Report, Promote, Reset" width="700" />
</p>

```
Week N:
  Fork → Play games → Evolution improves brain + wiki → Report → Promote → Reset

Week N+1:
  Base now includes promoted patterns from week N.
  Fresh fork = clean game code + improved framework knowledge.
```

Successful patterns are automatically promoted from game-specific evolution into the
framework wiki, benefiting all future games across all plugins.

### Documentation

- [Full Architecture & Pipeline](policies/cyborg-policy-framework/docs/POLICY_GENERATION.md)
- [New Game Template](policies/cyborg-policy-framework/games/new_game/README.md)
- [Framework Wiki — Skills](policies/cyborg-policy-framework/framework/wiki/skills/)
- [Framework Wiki — Strategy](policies/cyborg-policy-framework/framework/wiki/strategy/)
- [Framework Wiki — Mechanics](policies/cyborg-policy-framework/framework/wiki/mechanics/)

---

## Adding New Policy Frameworks

This repo supports multiple policy frameworks. To add a new one, create a new directory
under `policies/` alongside `cyborg-policy-framework/`.
