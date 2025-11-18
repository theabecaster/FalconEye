# FalconEye

A distributed spatial mapping system for real-time through-wall human pose visualization using multiple iPhones and a central MacBook server.

## Overview

FalconEye enables real-time visualization of human poses detected through walls by combining:
- **Multiple iPhone clients** with LiDAR sensors capturing camera frames and spatial data
- **Central MacBook server** performing pose detection and coordinate transformation
- **Real-time WebSocket communication** for low-latency data streaming
- **Shared world coordinate system** for multi-device pose fusion

### Use Cases
- Tactical situational awareness
- Search and rescue operations
- Security and surveillance
- Spatial computing research
- Multi-device AR experiences

## Architecture

### Components

#### Aerie - Server (MacBook)
The central processing hub that:
- Manages WebSocket connections from multiple HawkSight clients
- Performs pose detection using MediaPipe
- Transforms detected poses to a shared world coordinate system
- Broadcasts world-frame poses to all connected clients

**Technology**: Swift 6.0, Vapor 4.115+, WebSocketKit, MediaPipe

**[View Aerie Documentation →](./Aerie/README.md)**

#### HawkSight - iOS Client (iPhone)
The mobile capture and visualization app that:
- Captures camera frames and ARKit device poses
- Transmits frames to Aerie server via WebSocket
- Receives world-frame pose broadcasts
- Renders skeleton overlays on live camera feed for through-wall visualization

**Technology**: Swift, SwiftUI, ARKit, AVFoundation, CryptoKit

**[View HawkSight Documentation →](./HawkSight/README.md)**

### System Requirements

#### Aerie Server
- M4 Max MacBook Pro (or similar)
- macOS 13+
- Swift 6.0+
- 8GB+ RAM (recommended for 3+ devices)

#### HawkSight Client
- iPhone 12 Pro or later (LiDAR required)
- iOS 16+
- Developer Mode enabled
- Same WiFi network as Aerie server

## Quick Start

### 1. Start Aerie Server

```bash
cd Aerie
swift build
swift run
# Server starts on 0.0.0.0:8000
```

### 2. Configure & Run HawkSight

```bash
# Open in Xcode
open HawkSight/HawkSight.xcodeproj

# Update server URL to point to Aerie's IP address
# Build and run on physical iPhone (Simulator not supported)
```

### 3. Test Connection

```bash
# From any machine on the network
curl http://[aerie-ip]:8000
# Expected: "It works!"
```

## Network Protocol

### WebSocket Endpoint
```
ws://[server-ip]:8000/ws/connect/{deviceId}
```

### Message Flow
1. **Authentication**: Client sends auth token, server responds with session ID
2. **Frame Transmission**: Client streams camera frames + device pose at 30 FPS
3. **Pose Detection**: Server detects poses and transforms to world coordinates
4. **Broadcasting**: Server broadcasts all poses to all connected clients
5. **Rendering**: Clients render skeleton overlays on camera feed

### Message Types
- `AuthRequest` / `AuthResponse` - Device authentication
- `CameraFrame` - Image + pose data from client
- `PoseBroadcast` - Detected poses in world frame

All messages use JSON-encoded Codable Swift structs.

## Development Status

The project follows a phased development approach:

- **Phase 0**: Infrastructure setup ✓
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

See [PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md) for complete roadmap and phase details.

## Performance Targets

| Metric | Target | Status |
|--------|--------|--------|
| End-to-end latency | <50ms | Phase 6 |
| Frame rate | 30 FPS | Phase 1 |
| Multi-person support | 3+ simultaneous | Phase 7 |
| Concurrent devices | 3+ | Phase 1 |
| Network bandwidth | <10 Mbps per device | Phase 2 |
| Pose detection accuracy | >90% | Phase 3 |

## Project Structure

```
FalconEye/
├── Aerie/                    # Server component (Vapor app)
│   ├── Sources/Aerie/
│   │   ├── entrypoint.swift
│   │   ├── configure.swift
│   │   └── routes.swift
│   ├── Package.swift
│   └── README.md
├── HawkSight/                # iOS client component
│   ├── HawkSight/
│   │   ├── HawkSightApp.swift
│   │   └── ContentView.swift
│   ├── HawkSight.xcodeproj/
│   └── README.md
├── README.md                 # This file
└── PROJECT_OVERVIEW.md       # Detailed phase roadmap
```

## Key Technologies

### Coordinate Systems
- **Camera Frame**: 2D image coordinates + depth from LiDAR
- **Device Frame**: ARKit world tracking (4x4 transform matrix)
- **World Frame**: Shared coordinate system across all devices
- Transform pipeline: Camera → Device → World

### Pose Detection
- 13 key joints tracked: head, shoulders, elbows, wrists, hips, knees, ankles
- MediaPipe pose detection running on M4 Max
- Multi-person detection support (3+ people)
- Confidence scores per joint (0-1 range)

### Rendering
- Real-time skeleton overlay on live camera feed
- Joints: 6-8px radius circles (green)
- Limbs: cyan lines connecting joints
- Rendered at camera frame rate (30 FPS)

## Common Issues

### HawkSight won't connect to Aerie
1. Verify both devices on same WiFi network
2. Update serverURL to Aerie's local IP (not localhost)
3. Check firewall settings on MacBook
4. Verify Aerie is running: `swift run` in Aerie directory

### Build errors in Aerie
```bash
# Ensure Swift 6.0+ installed
swift --version

# Clean and rebuild
rm -rf .build
swift package resolve
swift build
```

### HawkSight requires physical device
- Simulator doesn't support ARKit LiDAR features
- Must use iPhone 12 Pro or later with LiDAR sensor
- Enable Developer Mode in iOS Settings

## Testing

### Unit Tests
```bash
# Aerie
cd Aerie && swift test

# HawkSight
# Run tests in Xcode: Product → Test (⌘U)
```

### Integration Testing
1. Single client connection test
2. Multi-client (2-3 devices) simultaneous operation
3. Sustained load test (5+ minutes at 30 FPS)
4. Disconnect/reconnect handling
5. Invalid authentication rejection

## Docker Support

Run Aerie in Docker:
```bash
cd Aerie
docker-compose up
```

## Contributing

This is a research/experimental project. Development follows the phased roadmap in PROJECT_OVERVIEW.md.

## Security

- Token-based authentication with HMAC signatures
- Session management with 1-hour timeout
- Secure WebSocket (WSS) recommended for production
- No persistent storage of camera frames
- Privacy-focused: pose detection only (no face detection)

## License

[License information to be added]

## References

- [ARKit Documentation](https://developer.apple.com/arkit/)
- [Vapor Web Framework](https://docs.vapor.codes/)
- [MediaPipe](https://mediapipe.dev/)
- [WebSocket Protocol](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)

## Support

For issues, questions, or feature requests, please open an issue in the respective component repository.
