# MonPrime - Parallel Development Plan

## Development Strategy

Build reusable components that can work in both Web (Three.js/WebXR) and Unity, with shared backend services. Multiple agents can work on tasks simultaneously.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                   Shared Components                       │
├─────────────────────────────────────────────────────────┤
│  Mon Models  │  Game Logic  │  Network Protocol  │  API  │
└─────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┴─────────────────────┐
        ▼                                           ▼
┌─────────────────┐                       ┌─────────────────┐
│  Web Prototype  │                       │  Unity Client   │
│   (Three.js)    │                       │  (Production)   │
└─────────────────┘                       └─────────────────┘
```

## Task Groups (Can Be Done in Parallel)

### Group A: Core Data & Systems

#### Task A1: Mon Data System
**Output**: Reusable Mon generation service
```typescript
// Shared TypeScript/C# compatible interfaces
interface IMon {
  id: number;
  name: string;
  dna: string; // Procedural generation seed
  stats: IMonStats;
  abilities: string[];
  modelData: IModelData;
}

// Service that both web and Unity can use
class MonGeneratorService {
  generateFromSeed(seed: number): IMon
  generateFromDNA(dna: string): IMon
  mutate(mon: IMon, factor: number): IMon
}
```

#### Task A2: Combat Formula Engine
**Output**: Platform-agnostic combat calculations
```typescript
// Pure functions that work everywhere
class CombatEngine {
  calculateDamage(attacker: ICombatant, defender: ICombatant, move: IMove): number
  calculateCaptureChance(mon: IMon, book: IGoldenBook, conditions: ICaptureConditions): number
  calculatePhysicsImpact(force: Vector3, targetMass: number): Vector3
}
```

#### Task A3: Curse System Logic
**Output**: Reusable curse state machine
```typescript
class CurseSystem {
  curseCycle: Observable<CurseState>
  activeCurse: CurseType
  applyModifiers(gameState: IGameState): IGameState
}
```

#### Task A4: Golden Book System
**Output**: Book management logic
```typescript
class GoldenBookSystem {
  books: Map<string, IGoldenBook>
  tryCapture(mon: IMon, book: IGoldenBook): CaptureResult
  canFit(mon: IMon, book: IGoldenBook): boolean
  serialize(): string // For save/load
}
```

### Group B: Backend Services

#### Task B1: Mon Database Service
**Output**: REST/GraphQL API for Mon data
```javascript
// Node.js microservice
POST   /api/mons/generate       // Generate new Mon
GET    /api/mons/:id           // Get Mon by ID
GET    /api/mons/spawn         // Get Mons for location/time/weather
PUT    /api/mons/:id/capture   // Record capture
GET    /api/mons/search        // Search with filters
```

#### Task B2: Real-time Battle Service
**Output**: WebSocket service for combat
```javascript
// Socket.io or WebRTC data channels
Events:
- player:punch { position, direction, force }
- mon:damaged { monId, damage, knockback }
- mon:counter { monId, attackType, target }
- player:defeated { playerId, cause }
```

#### Task B3: World State Service
**Output**: Manages curse cycles and global events
```javascript
// Synchronized world state
GET    /api/world/curse        // Current curse
GET    /api/world/events       // Active events
WS     /world/subscribe        // Real-time updates
POST   /api/world/report       // Player reports position
```

#### Task B4: Save State Service
**Output**: Cross-platform save system
```javascript
// Player progress persistence
POST   /api/save/state         // Save game state
GET    /api/save/state         // Load game state
POST   /api/save/backup        // Create backup
GET    /api/save/sync          // Sync across devices
```

### Group C: Web Prototype (Three.js)

#### Task C1: Core 3D Scene
**Output**: Basic Three.js scene with camera
```javascript
class MonPrimeScene {
  scene: THREE.Scene
  camera: THREE.PerspectiveCamera
  renderer: THREE.WebGLRenderer
  controls: OrbitControls // For non-AR testing
  
  init(container: HTMLElement)
  addMon(mon: IMon, position: Vector3)
  update(deltaTime: number)
}
```

#### Task C2: WebXR Integration
**Output**: AR capabilities using WebXR
```javascript
class WebXRManager {
  session: XRSession
  referenceSpace: XRReferenceSpace
  
  async initAR()
  detectPlanes(): XRPlane[]
  hitTest(screenPoint: Vector2): XRHitResult
  placeObject(position: Vector3)
}
```

#### Task C3: Camera Feed Handler
**Output**: Device camera integration
```javascript
class CameraHandler {
  stream: MediaStream
  videoTexture: THREE.VideoTexture
  
  async requestCamera()
  createARBackground(): THREE.Mesh
  captureFrame(): ImageData
  detectEnvironment(): EnvironmentType
}
```

#### Task C4: Web Combat System
**Output**: Touch/mouse based combat
```javascript
class WebCombatSystem {
  raycaster: THREE.Raycaster
  
  handleTouch(event: TouchEvent)
  handleSwipe(gesture: SwipeGesture)
  performPunch(origin: Vector3, direction: Vector3)
  applyPhysics(mon: MonObject, force: Vector3)
}
```

#### Task C5: Web UI Components
**Output**: Reusable React/Vue components
```jsx
// Shared UI components
<GoldenBookInventory books={books} onDeploy={deployMon} />
<CurseTimer curse={currentCurse} timeRemaining={seconds} />
<MonHealthBar mon={selectedMon} />
<CaptureUI mon={targetMon} book={selectedBook} />
```

### Group D: Unity Development

#### Task D1: Unity Project Setup
**Output**: Base Unity project with packages
- AR Foundation setup
- Package manifest with dependencies
- Folder structure matching web prototype
- Build settings for iOS/Android

#### Task D2: Shared Code Integration
**Output**: C# ports of shared systems
```csharp
// Use same interfaces as TypeScript
public interface IMon { /* same as TS */ }
public class MonGeneratorService { /* same logic */ }
public class CombatEngine { /* same formulas */ }
```

#### Task D3: Unity AR Components
**Output**: AR-specific Unity features
```csharp
[RequireComponent(typeof(ARRaycastManager))]
public class UnityARMonPlacer : MonoBehaviour {
    void PlaceMonAtHit(ARRaycastHit hit) { }
}
```

#### Task D4: Unity Physics Integration
**Output**: Physics-based combat feel
```csharp
public class PhysicsPunchSystem : MonoBehaviour {
    void ApplyPunchForce(Rigidbody target, Vector3 force) {
        // Unity's physics for satisfying impacts
    }
}
```

### Group E: Shared Assets & Tools

#### Task E1: Mon Model Generator
**Output**: Tool to generate 3D models
```python
# Blender script or Houdini setup
def generate_mon_model(dna: str) -> Model:
    base_shape = select_base(dna)
    modifiers = extract_modifiers(dna)
    return apply_procedural_modifiers(base_shape, modifiers)
```

#### Task E2: Asset Pipeline
**Output**: Automated asset processing
- GLTF export for web
- FBX export for Unity
- Texture optimization
- LOD generation
- Animation retargeting

#### Task E3: Development Tools
**Output**: Tools for content creation
- Mon DNA editor
- Curse effect previewer
- Combat balance calculator
- Spawn rule validator

### Group F: Testing & Validation

#### Task F1: Unit Test Suite
**Output**: Tests for all pure logic
```javascript
describe('CombatEngine', () => {
  test('calculates type effectiveness', () => {})
  test('applies curse modifiers', () => {})
  test('validates capture probability', () => {})
})
```

#### Task F2: Integration Test Harness
**Output**: End-to-end test scenarios
- Spawn Mon → Combat → Capture flow
- Curse cycle transitions
- Save/load integrity
- Cross-platform compatibility

#### Task F3: Performance Benchmarks
**Output**: Performance tracking system
- FPS monitoring
- Memory usage tracking
- Network latency measurement
- Battery usage profiling

## Reusable Component Library

### Shared Interfaces (TypeScript/C#)
```typescript
// Core game entities
interface IGameEntity {
  id: string;
  position: Vector3;
  rotation: Quaternion;
}

interface IMon extends IGameEntity {
  stats: IMonStats;
  abilities: IAbility[];
  owner?: IPlayer;
}

interface IPlayer extends IGameEntity {
  health: number;
  inventory: IInventory;
  stats: IPlayerStats;
}

// Combat system
interface ICombatAction {
  source: IGameEntity;
  target: IGameEntity;
  type: ActionType;
  timestamp: number;
}

// Serialization
interface ISerializable {
  serialize(): string;
  deserialize(data: string): void;
}
```

### Shared Math Library
```typescript
// Works in both environments
class GameMath {
  static calculateDistance(a: Vector3, b: Vector3): number
  static randomInSphere(radius: number): Vector3
  static smoothDamp(current: number, target: number, velocity: number): number
}
```

### Event System
```typescript
// Platform-agnostic events
class GameEventBus {
  emit(event: GameEvent): void
  on(eventType: string, handler: Function): void
  off(eventType: string, handler: Function): void
}
```

## Development Workflow

### For Web Prototype
1. Run all Group C tasks in parallel
2. Connect to backend services (Group B)
3. Import shared logic (Group A)
4. Test in browser with webcam

### For Unity Build
1. Run all Group D tasks in parallel
2. Import shared C# code
3. Connect to same backend services
4. Test in Unity editor with AR simulation

### For Backend
1. Run all Group B tasks in parallel
2. Deploy as microservices
3. Use Docker for consistency
4. Share between web and Unity

## Quick Start Tasks

### Minimal Playable Web Demo
1. Task C1: Basic Three.js scene
2. Task A1: Generate 5 test Mons
3. Task C4: Click-to-punch combat
4. Task A2: Damage calculation
5. Task C5: Health bar UI

### Minimal AR Test
1. Task C2: WebXR setup
2. Task C3: Camera feed
3. Task C1: Overlay 3D Mon
4. Task C4: Touch combat
5. Deploy to HTTPS (required for WebXR)

## Platform-Specific Considerations

### Web Advantages
- Instant testing (no build time)
- Easy sharing (just a URL)
- WebXR works on many devices
- Chrome DevTools for debugging
- Hot reload development

### Unity Advantages
- Better performance
- Native AR features
- Advanced physics
- Platform-specific optimizations
- Asset streaming

## Success Metrics

### Per Component
- Code coverage > 80%
- Build time < 30 seconds
- Memory footprint < 100MB
- API response < 200ms
- FPS > 30 (web), > 60 (Unity)

### Integration
- Web ↔ Unity feature parity
- Shared save games
- Same combat outcomes
- Synchronized world state

This parallel approach lets multiple agents work simultaneously, building a web prototype for quick iteration while preparing production Unity builds, all sharing the same backend and game logic!