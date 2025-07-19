# MonPrime Repository Overview

## Active Repositories

### 1. [monprime-game](https://github.com/toddllm/monprime-game) (This Repo)
**Purpose**: Central game design documentation and coordination
**Contents**: 
- Game design documents
- Technical architecture
- Development roadmap
- Cross-repository coordination

### 2. [monprime-web](https://github.com/toddllm/monprime-web)
**Purpose**: Browser-based WebXR implementation
**Status**: Initial structure created, awaiting agent development
**Issue #1**: [Initial Agent Review: Enhance Web Implementation Details](https://github.com/toddllm/monprime-web/issues/1)

### 3. [monprime-unity](https://github.com/toddllm/monprime-unity)
**Purpose**: Unity AR Foundation production build
**Status**: Task list created, awaiting agent development
**Issue #1**: [Initial Agent Review: Optimize Unity AR Implementation](https://github.com/toddllm/monprime-unity/issues/1)

### 4. [monprime-mobile](https://github.com/toddllm/monprime-mobile)
**Purpose**: React Native cross-platform mobile app
**Status**: Task structure defined, awaiting agent review
**Issue #1**: [Initial Agent Review: React Native AR Strategy](https://github.com/toddllm/monprime-mobile/issues/1)

## Pending Repositories

### 5. monprime-backend (To Be Created)
**Purpose**: Shared backend services
**Planned Contents**:
- Mon generation API
- Real-time battle server
- Player data management
- World state service

### 6. monprime-shared (To Be Created)
**Purpose**: Shared code and interfaces
**Planned Contents**:
- TypeScript/C# interfaces
- Game logic formulas
- Common constants
- Shared utilities

## Development Workflow

1. **Agents claim issues** in their assigned repository
2. **Research and propose** enhanced implementation details
3. **Update documentation** based on findings
4. **Begin implementation** following approved architecture
5. **Cross-reference** with other repositories for consistency

## Coordination Points

- **Weekly Sync**: Update progress in each repository
- **Shared Interfaces**: Define in monprime-shared
- **API Contracts**: Document in monprime-backend
- **Design Changes**: Update in monprime-game

## Getting Started for New Agents

1. Choose a repository based on your expertise
2. Review the repository's issue #1
3. Research best practices for that platform
4. Provide detailed recommendations
5. Begin tackling tasks from TASKS.md

## Communication

- **Design Questions**: Open issue in monprime-game
- **Technical Blockers**: Open issue in relevant repository
- **Cross-repo Dependencies**: Tag related repositories
- **Architecture Decisions**: Document in each repo's ARCHITECTURE.md

## Success Metrics

- Each repository has detailed architecture documentation
- All platforms can spawn and display a Mon
- Combat calculations match across platforms
- Shared backend successfully serves all clients
- Golden Book state syncs properly

## Related Projects

- [narrative-analysis](https://github.com/toddllm/narrative-analysis) - Story extraction tools
- zeldina-story.txt - Source narrative being analyzed

This multi-repository approach enables parallel development while maintaining consistency through shared services and interfaces.