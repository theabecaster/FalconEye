# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Component-Specific Documentation

**IMPORTANT**: Each component has its own detailed CLAUDE.md file with specific implementation guidance:

- **`Aerie/CLAUDE.md`** - Server-specific details including:
  - Vapor server configuration and routing
  - WebSocket message handling
  - Pose detection pipeline (MediaPipe integration)
  - Coordinate transformation implementation
  - Broadcasting strategy and session management
  - Performance optimization targets
  - Testing and troubleshooting

- **`HawkSight/CLAUDE.md`** - iOS client-specific details including:
  - ARKit integration and pose capture
  - SwiftUI view architecture
  - WebSocket client implementation
  - Frame capture and transmission
  - Skeleton rendering and overlay
  - Coordinate projection (world → image)
  - UI components and state management
  - Testing on physical devices

**When working on a specific component, always read its CLAUDE.md first for detailed context.**

## Project Overview

**FalconEye** is a distributed spatial mapping system for real-time through-wall human pose visualization using multiple iPhones and a central MacBook server.

### Components
- **Aerie** - MacBook Swift server (central processing hub) running on Vapor
- **HawkSight** - iPhone client app (pose capture and visualization) using SwiftUI + ARKit

### Target Architecture
- M4 Max MacBook as Aerie server
- Multiple iPhones (12 Pro+ with LiDAR) as HawkSight clients
- WebSocket-based communication
- Target latency: <50ms end-to-end

## Build Commands

### Aerie (Server)
```bash
cd Aerie
swift build                    # Build the server
swift run                      # Run the server (listens on 0.0.0.0:8000)
swift test                     # Run tests
```

### HawkSight (iOS Client)
```bash
# Open in Xcode
open HawkSight/HawkSight.xcodeproj

# Build and run on physical device (Simulator won't work - requires LiDAR)
# Update serverURL in config to point to Aerie's local IP before running
```

## Development Workflow

### Docker Support (Aerie)
```bash
cd Aerie
docker-compose up              # Run server in Docker container
```

### Testing Environment Requirements
- Two or more iPhones with LiDAR (iPhone 12 Pro or later)
- M4 Max MacBook Pro on same WiFi network
- Physical walls for through-wall testing

## Architecture

### Network Protocol
- **Protocol**: WebSocket (JSON messages)
- **Server**: Vapor framework on port 8000
- **Endpoint**: `/ws/connect/{deviceId}`
- **Auth**: Token-based authentication with session management

### Data Flow
1. HawkSight captures ARKit pose + camera frames
2. Data transmitted to Aerie via WebSocket
3. Aerie performs pose detection (MediaPipe planned)
4. Coordinates transformed to shared world frame
5. Poses broadcast back to all HawkSight clients
6. HawkSight renders skeleton overlays on camera feed

### Coordinate Systems
- **Camera Frame**: 2D image coordinates + depth
- **Device Frame**: ARKit world tracking (4x4 transform matrix)
- **World Frame**: Shared coordinate system across all devices
- Transform pipeline: Camera → Device → World → Broadcast

### Key Technologies
- **Aerie**: Swift 6.0, Vapor 4.115+, NIO, WebSocketKit 2.16+, MediaPipe (planned)
- **HawkSight**: Swift, SwiftUI, ARKit, AVFoundation, CryptoKit

## Phase Development Status

The project follows a phased development approach (see PROJECT_OVERVIEW.md for complete roadmap):

- **Phase 0**: Infrastructure setup (✓ Complete)
- **Phase 1**: Network foundation & authentication (In Progress)
- **Phase 2**: ARKit integration & device pose transmission
- **Phase 3**: Pose detection engine (MediaPipe)
- **Phase 4**: Coordinate transforms & world frame alignment
- **Phase 5**: HawkSight pose reception & rendering (MVP)
- **Phase 6**: Latency optimization to <50ms
- **Phase 7**: Multi-person tracking & persistence
- **Phase 8**: Spatial mesh integration (LiDAR)
- **Phase 9**: Aerie dashboard & monitoring
- **Phase 10**: Quest 3 integration

## Key Performance Metrics

| Metric | Target | Phase |
|--------|--------|-------|
| End-to-end latency | <50ms | 6 |
| Frame rate | 30 FPS | 1 |
| Multi-person support | 3+ simultaneous | 7 |
| Devices supported | 3+ | 1 |
| Network bandwidth | <10 Mbps | 2 |
| Pose detection accuracy | >90% | 3 |

## Important Implementation Notes

### Shared Data Models
All network communication uses Codable Swift models shared between Aerie and HawkSight:
- AuthRequest, AuthResponse
- DeviceInfo, Message wrappers
- CameraFrame, Joint, PoseMatrix structures
- Models must serialize/deserialize consistently across both projects

### Pose Detection
- MediaPipe running on Aerie (M4 acceleration)
- 13 key joints tracked: head, shoulders, elbows, wrists, hips, knees, ankles
- Confidence scores per joint (0-1)
- Multi-person detection support (3+ people)

### Rendering (HawkSight)
- Skeletons overlay on live camera feed
- Joints: 6-8px radius circles (green)
- Limbs: cyan lines connecting joints
- Real-time at camera frame rate (30 FPS)

### Session Management
- Sessions created on successful auth
- 1-hour timeout for inactive sessions
- Session revoked on disconnect
- Concurrent connections supported

## Project Structure

```
FalconEye/
├── Aerie/                    # Server (Vapor app)
│   ├── Sources/Aerie/
│   │   ├── entrypoint.swift  # Main entry point
│   │   ├── configure.swift   # Server config (port 8000)
│   │   └── routes.swift      # HTTP/WebSocket routes
│   ├── Package.swift         # Swift Package Manager manifest
│   └── Tests/
├── HawkSight/                # iOS client
│   ├── HawkSight/
│   │   ├── HawkSightApp.swift
│   │   └── ContentView.swift
│   └── HawkSight.xcodeproj/
└── PROJECT_OVERVIEW.md       # Detailed phase roadmap
```

## Common Issues

### HawkSight won't connect to Aerie
- Verify both devices on same WiFi network
- Update serverURL to Aerie's local IP address (not localhost)
- Check firewall settings on MacBook
- Verify Aerie is running: `swift run` in Aerie directory

### Build errors in Aerie
- Ensure Swift 6.0+ installed: `swift --version`
- Run `swift package resolve` to fetch dependencies
- Clean build: `rm -rf .build && swift build`

### HawkSight requires physical device
- Simulator doesn't support ARKit LiDAR features
- Must use iPhone 12 Pro or later with LiDAR sensor
- Enable Developer Mode in iOS Settings

## Working with Specific Components

### When working on Aerie (Server)
1. Read `Aerie/CLAUDE.md` for detailed server architecture
2. Navigate to `Aerie/` directory
3. Use commands: `swift build`, `swift run`, `swift test`
4. Reference Aerie-specific sections for WebSocket handlers, pose detection, and coordinate transforms

### When working on HawkSight (iOS Client)
1. Read `HawkSight/CLAUDE.md` for detailed iOS client architecture
2. Open `HawkSight/HawkSight.xcodeproj` in Xcode
3. Reference HawkSight-specific sections for ARKit integration, rendering, and network protocol
4. Always test on physical device (not Simulator)

### When working on cross-component features
1. Read both `Aerie/CLAUDE.md` and `HawkSight/CLAUDE.md`
2. Pay attention to shared data models (must be consistent)
3. Verify message formats match on both sides
4. Test end-to-end integration

## Reference Documentation

- **PROJECT_OVERVIEW.md**: Complete 10-phase development roadmap with detailed deliverables
- **Aerie/CLAUDE.md**: Detailed server implementation guide
- **HawkSight/CLAUDE.md**: Detailed iOS client implementation guide
- ARKit: https://developer.apple.com/arkit/
- Vapor: https://docs.vapor.codes/
- MediaPipe: https://mediapipe.dev/
