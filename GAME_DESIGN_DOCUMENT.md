# MonPrime - Game Design Document

## Game Overview

MonPrime is an AR (Augmented Reality) mobile game that combines physical interaction, creature collection, and survival mechanics. Players capture and battle "Mons" using Golden Books in a world created by Zeldina, a mysterious creator-void entity.

## Core Concept

**Genre**: AR Collection/Battle/Survival Game  
**Platform**: iOS/Android with AR capabilities  
**Target Audience**: 13+ (due to combat mechanics and potential player elimination)  
**Unique Selling Points**:
- Physical combat interaction (punching mechanics)
- High-stakes battles (elimination/respawn system)
- Golden Books capture system
- Dynamic curse cycles
- Environmental awareness

## Key Game Elements

### 1. Zeldina - The Creator
- **Type**: Creator-Void (can be any type)
- **Role**: World creator, most powerful entity
- **Special Properties**:
  - Invisible to cameras/photos
  - Owns Master Book (captures other books)
  - Can vaporize any creature instantly
  - Creates Mons in dungeons using cubes
  - Creates maps and world elements

### 2. Mons System
- **Total Mons**: 1,000,000 unique creatures
- **Capture Method**: Golden Books (not pokeballs)
- **Storage Limitations**: Some Mons don't fit in books
- **Combat**: Physical and special attacks
- **Loyalty**: Captured Mons work for the player

### 3. Combat Mechanics
- **Physical Combat**: Players can punch Mons (they get flung)
- **Mon Abilities**: Some can vaporize humans or out-punch players
- **Mandatory Battles**: Must fight or be defeated
- **Death Penalty**: Kicked from game, must reconnect to respawn
- **Boss Mons**: Higher difficulty encounters

### 4. Curse System
- **Cycle Time**: 20 seconds active, 20 seconds rest
- **Variety**: Different curse types rotate
- **Impact**: Affects gameplay, spawns, abilities

### 5. Environmental Systems
- **Day/Night Cycle**: Real-time based
- **Weather Integration**: Affects Mon spawns
- **Location Detection**: Different Mons in different environments
- **AR Integration**: Mons appear in real world

### 6. Progression System
- **Currency**: Coins from defeated Mons
- **Experience**: XP for player leveling
- **Collection**: Fill Golden Book entries
- **Team Building**: Deploy captured Mons

## Game Loop

1. **Explore** real world with AR camera
2. **Detect** Mons based on environment/time/weather
3. **Battle** (mandatory - fight or be eliminated)
4. **Capture** using Golden Books
5. **Manage** team and resources
6. **Survive** curse cycles
7. **Progress** through boss encounters

## Unique Mechanics

### Golden Book System
- Primary capture tool
- Different book types/sizes
- Master Book (Zeldina's) can capture other books
- Storage limitations create strategic decisions

### Physical Combat
- Swipe/tap gestures for punching
- Physics-based Mon reactions
- Risk/reward of close combat
- Some Mons counter-punch

### Elimination System
- High stakes battles
- Full disconnect on defeat
- Respawn penalty (loss of progress/items?)
- Encourages careful play

### Curse Cycles
- 20-second active curses
- Environmental hazards
- Stat modifications
- Special Mon spawns during curses

## Technical Requirements

### Core Technologies
- **AR Framework**: ARCore (Android) / ARKit (iOS)
- **Backend**: Real-time multiplayer support
- **Database**: 1M+ Mon variations
- **Physics Engine**: For punch/fling mechanics
- **Weather API**: Real-time weather data
- **Location Services**: GPS for spawning

### Key Features
- Camera integration with AR overlay
- Gesture recognition for combat
- Real-time environmental detection
- Persistent world state
- Cross-platform compatibility

## Monetization Strategy

### Ethical F2P Model
- **Golden Book Packs**: Different sizes/types
- **Respawn Tokens**: Skip reconnection penalty
- **Cosmetic Items**: Player/Mon customization
- **Storage Expansion**: More book space
- **NO Pay-to-Win**: All Mons catchable through play

## Risk Mitigation

### Safety Concerns
- **Warning System**: Don't play while driving/unsafe areas
- **Safe Zones**: Designated battle areas
- **Parental Controls**: Time limits, area restrictions

### Technical Challenges
- **1M Mon Variety**: Procedural generation system
- **Server Load**: Regional servers, load balancing
- **AR Accuracy**: Fallback to screen-based play

## Development Phases

### Phase 1: Core Prototype (3 months)
- Basic AR Mon detection
- Simple combat system
- Golden Book capture
- 100 prototype Mons

### Phase 2: Systems Integration (3 months)
- Curse cycle implementation
- Environmental detection
- Day/night & weather
- 1,000 Mons

### Phase 3: Content Expansion (6 months)
- Full 1M Mon generation system
- Boss encounters
- Zeldina mythology/lore
- Social features

### Phase 4: Launch Preparation (2 months)
- Balance testing
- Server stress testing
- Marketing campaign
- Community building

## Success Metrics

- **Daily Active Users**: Target 1M+ within 6 months
- **Retention**: 30-day retention > 40%
- **Monetization**: ARPU $2-5
- **Engagement**: Average 45 min/day play time
- **Safety**: < 0.1% incident rate

## Narrative Integration

The game will incorporate narrative elements from zeldina-story.txt:
- Wordvile arcade machine as origin
- Character backstories (Marco, Lily, Zack, Ava)
- Mystery of Mon creation
- Zeldina's true nature
- Portal between digital and physical worlds