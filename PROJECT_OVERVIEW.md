# Spatial Through-Wall Pose Visualization System
## Phase Planning & Development Roadmap

---

## Project Overview

**Goal:** Build a distributed spatial mapping system where multiple iPhones collaboratively detect and visualize human poses through walls in real-time.

**Architecture:** M4 Max MacBook (Aerie server) + Multiple iPhones (HawkSight clients)
**Total Duration:** 8-10 weeks
**Target Latency (MVP):** <50ms end-to-end

### Component Naming
- **FalconEye** - Overall project name
- **Aerie** - MacBook Swift server (central processing hub)
- **HawkSight** - iPhone client app (pose capture and visualization)

---

# PHASE BREAKDOWN

## PHASE 0: Infrastructure & Project Setup
**Duration:** 2-3 days  
**Goal:** ✓ Establish clean project structure with both MacBook server and iPhone client building successfully

### Phase 0.1: Aerie Server Xcode Project
**Goal:** ✓ Create Swift command-line server project that compiles and runs
- Set up Swift Package Manager project for Aerie
- Add Vapor and WebSocketKit dependencies
- Verify clean build without errors
- Aerie executable can start and listen on port 8000

### Phase 0.2: HawkSight Client Xcode Project
**Goal:** ✓ Create iOS app project with proper architecture that builds for LiDAR-capable devices
- Create HawkSight app targeting iPhone 12 Pro+ (or later)
- Set up SwiftUI project structure with proper folders (Models, Views, Networking, etc.)
- Verify build succeeds for physical device
- App launches without crashes

### Phase 0.3: Repository & Version Control
**Goal:** ✓ Clean git repo with proper structure, both projects checked in
- Initialize git repo with meaningful .gitignore
- Create directory structure matching project design
- Both Aerie and HawkSight projects in repo
- Initial commit with README documenting setup

**Phase 0 Checklist:**
- [ ] Aerie server: `swift build` succeeds
- [ ] HawkSight app: builds and runs in Simulator/on device
- [ ] Git repo initialized with both projects
- [ ] No build warnings or errors

---

## PHASE 1: Network Foundation & Authentication
**Duration:** 3-4 days
**Goal:** ✓ Establish secure WebSocket connection with token-based authentication between HawkSight and Aerie

### Phase 1.1: Define Protocol & Shared Data Models
**Goal:** ✓ Create shared Swift models for all network communication
- Define AuthRequest, AuthResponse structures
- Define DeviceInfo, Message wrapper structures
- Define CameraFrame, Joint, PoseMatrix structures
- All models conform to Codable for JSON serialization
- Models serialized/deserialized correctly in tests

### Phase 1.2: Aerie WebSocket Server - Basic Setup
**Goal:** ✓ Aerie server accepts WebSocket connections and handles device registration
- Vapor app listens on ws://0.0.0.0:8000
- WebSocket endpoint at /ws/connect/{deviceId}
- Server accepts incoming connections
- Logs connection/disconnection events
- Server remains stable with continuous activity

### Phase 1.3: Authentication System (Aerie)
**Goal:** ✓ Aerie validates device tokens and manages session lifecycle
- Generate valid auth tokens for devices
- Validate incoming auth tokens (reject invalid/stale)
- Create sessions on successful auth
- Validate sessions for subsequent messages
- Sessions expire after timeout (1 hour)
- Revoke sessions on disconnect

### Phase 1.4: HawkSight WebSocket Client - Connection & Auth
**Goal:** ✓ HawkSight app connects to Aerie and successfully authenticates
- HawkSight can connect to WebSocket at given server URL
- Generates valid auth token locally
- Sends auth request to Aerie
- Receives auth response with session ID
- Connection state updates (disconnected → connecting → connected)
- Can disconnect and reconnect without errors

### Phase 1.5: End-to-End Auth Testing
**Goal:** ✓ Verified secure connection pipeline with multiple devices
- Start Aerie server
- Launch 1 HawkSight → connects and authenticates
- Launch 2nd HawkSight → both authenticate separately
- Server logs show all auth events
- Sessions active and tracked
- Invalid tokens rejected
- Sessions persist for 5+ minutes
- Concurrent connections working

**Phase 1 Deliverables:**
- [ ] Secure WebSocket connection established
- [ ] Token-based authentication working
- [ ] Session management (create, validate, expire)
- [ ] Multiple devices supported
- [ ] Connection state properly tracked on client
- [ ] 0 authentication failures in stable network

---

## PHASE 2: ARKit Integration & Device Pose Transmission
**Duration:** 3-4 days
**Goal:** ✓ HawkSight captures and transmits device pose + camera intrinsics + camera frames to Aerie

### Phase 2.1: ARKit Session Management (HawkSight)
**Goal:** ✓ HawkSight reliably captures ARKit world tracking data
- ARWorldTracking configuration initializes
- Device pose (4x4 transform matrix) captured each frame
- Camera intrinsics (fx, fy, cx, cy) extracted correctly
- ARKit session runs at 60 FPS
- Pose matrices contain valid values (no NaN/Inf)
- Current pose accessible on demand

### Phase 2.2: Camera Frame Capture (HawkSight)
**Goal:** ✓ HawkSight captures camera frames at configurable rate
- AVCapture session initializes with front camera
- Frames captured at ~30 FPS
- Each frame converted to JPEG (quality: 0.6)
- Frame rate metric tracked and displayed
- Frame sizes reasonable (<500KB per frame)
- No frame drops over sustained capture

### Phase 2.3: Data Serialization & Transmission (HawkSight)
**Goal:** ✓ HawkSight packages and sends ARKit pose + camera frames to Aerie
- Frames buffered before transmission
- Pose + frame + intrinsics serialized to JSON
- Queue manages high throughput without memory leaks
- Transmission rate configurable (e.g., 10-30 FPS)
- Queue depth monitored
- Can pause/resume transmission

### Phase 2.4: Aerie Frame Reception
**Goal:** ✓ Aerie receives, parses, and stores incoming frames
- WebSocket handler receives frame messages
- Image data decoded from base64
- Pose matrix stored with frame
- Camera intrinsics extracted
- Frames stored in memory with timestamp
- Old frames cleaned up automatically
- Server logs show incoming frames

### Phase 2.5: Integration Testing
**Goal:** ✓ Verified frame + pose data flows from HawkSight to Aerie
- Start Aerie server
- Launch HawkSight → captures and transmits
- Aerie logs show frame reception
- Frame rate matches camera (~30 FPS)
- Zero frame drops over 2-minute test
- Pose matrices valid and consistent
- Intrinsics constant across frames
- Handle 2+ HawkSight instances simultaneously

**Phase 2 Deliverables:**
- [ ] ARKit pose captured and serialized
- [ ] Camera frames transmitted at desired rate
- [ ] Aerie receives all frames
- [ ] Frame queue stable (no memory leaks)
- [ ] Latency: capture → receive <500ms
- [ ] Concurrent frame streams from 2+ devices

---

## PHASE 3: Pose Detection Engine (Aerie)
**Duration:** 4-5 days
**Goal:** ✓ Aerie detects human poses in incoming camera frames with confidence scores

### Phase 3.1: MediaPipe Integration
**Goal:** ✓ Pose detection model loaded and running on Aerie (M4 MacBook)
- MediaPipe library installed and configured
- Pose detection model loads without errors
- Can process test image frame
- Returns keypoint positions with confidence
- Supports multiple people in single frame
- Processing time reasonable (<100ms per frame on M4)

### Phase 3.2: Joint Extraction
**Goal:** ✓ Extract 13 key joints from detected poses
- Head detected
- Shoulders (left, right) detected
- Elbows (left, right) detected
- Wrists (left, right) detected
- Hips (left, right) detected
- Knees (left, right) detected
- Ankles (left, right) detected
- Each joint has confidence score (0-1)
- Joint coordinates normalized or in image space

### Phase 3.3: Pose Detection Integration
**Goal:** ✓ Detected poses automatically extracted from every incoming frame
- Frame reception triggers pose detection
- Detection runs asynchronously (non-blocking)
- Results stored with frame metadata
- Can detect 0, 1, 2, 3+ people in frame
- Confidence filtering working (e.g., >0.5)
- Server logs show detection results

### Phase 3.4: Multi-Person Handling
**Goal:** ✓ Correctly detect and track multiple people simultaneously
- 2 people in frame → both detected
- 3+ people → all detected
- No person IDs mixed up
- Each person has separate joint set
- Confidence scores independent per person

### Phase 3.5: Pose Detection Testing
**Goal:** ✓ Verified reliable pose detection in real conditions
- Single person: detection accuracy >90%
- Multiple people (2-3): all detected
- Varies body positions: detection robust
- Low-light conditions: graceful degradation
- No false positives in empty frames
- Processing latency tracked

**Phase 3 Deliverables:**
- [ ] MediaPipe running on Aerie
- [ ] 13 key joints extracted per person
- [ ] Multiple people detected in single frame
- [ ] Confidence scores reasonable (>0.5 typical)
- [ ] Detection latency <100ms
- [ ] Zero crashes during sustained operation

---

## PHASE 4: Coordinate Transforms & World Frame Alignment
**Duration:** 3-4 days  
**Goal:** ✓ Transform detected poses from camera frame to shared world coordinate system

### Phase 4.1: Coordinate Transform Math
**Goal:** ✓ Implement camera→device→world transformation pipeline
- Image 2D + depth → Camera 3D space
- Camera 3D → Device frame (ARKit orientation)
- Device frame → World frame (shared across all devices)
- Can transform single 3D point through all frames
- Inverse transforms working (world → device → camera → image)
- Matrix operations stable (determinant checks, etc.)

### Phase 4.2: Device Pose Registration
**Goal:** ✓ Store and manage world poses for each connected device
- Each device's world pose stored on Aerie
- Poses updated with each frame reception
- Can retrieve device pose by device ID
- Device poses consistent over time
- No pose drift detected

### Phase 4.3: Pose Transformation to World
**Goal:** ✓ All detected joints transformed to world coordinates
- Estimated depth used for each joint (from LiDAR or default)
- Camera intrinsics used for back-projection
- Device pose used to transform to world
- Transformed joints stored with person ID
- Zero NaN/Inf in transformed coordinates

### Phase 4.4: Manual Calibration
**Goal:** ✓ Two devices can be manually aligned to same world frame
- Place visual markers in overlap region
- Record marker positions from both devices
- Compute relative transformation between devices
- Apply transformation to align coordinate frames
- Verify alignment (marker positions converge)

### Phase 4.5: World Pose Storage
**Goal:** ✓ Poses stored in world frame, indexed by person and device
- World poses stored with metadata (person ID, source device, timestamp)
- Can retrieve latest poses for broadcasting
- Can query poses within time window
- Old poses cleaned up (not indefinite growth)
- Storage stable with sustained pose stream

### Phase 4.6: Coordinate Transform Testing
**Goal:** ✓ Verified correct transformation between all frames
- Manual test: know real-world position, verify transformed matches
- Inverse transform: project back to camera, verify alignment
- Two-device test: calibrate and verify convergence
- No drift over sustained operation
- Transformation math validated

**Phase 4 Deliverables:**
- [ ] Camera→Device→World transforms implemented
- [ ] Device poses registered and tracked
- [ ] All joints transformed to world frame
- [ ] Manual calibration working
- [ ] World poses stored correctly
- [ ] Zero transformation errors (NaN/Inf checks)

---

## PHASE 5: HawkSight Pose Reception & Rendering
**Duration:** 3-4 days
**Goal:** ✓ HawkSight receives world-frame poses from Aerie and renders skeletons on camera overlay

### Phase 5.1: Pose Message Reception (HawkSight)
**Goal:** ✓ HawkSight receives and parses pose broadcasts from Aerie
- WebSocket receives pose messages
- JSON payload parsed correctly
- Poses decoded with all joint data
- Multiple poses in single message handled
- Timestamp verified
- Can retrieve latest poses on demand

### Phase 5.2: World to Image Projection
**Goal:** ✓ World-frame poses projected to HawkSight camera image coordinates
- World point → Device frame (via inverse device pose)
- Device frame → Camera frame (identity or calibrated)
- Camera 3D → Image 2D (perspective projection)
- Points behind camera filtered out
- Projected coordinates within image bounds (0-screen size)
- Confidence scores preserved

### Phase 5.3: Skeleton Rendering
**Goal:** ✓ Skeletons drawn on camera feed
- Joint positions drawn as circles (6-8px radius)
- Color: green for visible joints
- Limbs drawn as lines connecting joints
- Color: cyan for limbs
- All 13 joints rendered when visible
- Limbs: shoulders-elbows-wrists, hips-knees-ankles, shoulders-hips
- Rendering at camera frame rate (30 FPS)

### Phase 5.4: Real-Time Overlay
**Goal:** ✓ Skeletons overlay on live camera feed without latency
- Camera feed displayed full-screen
- Skeleton overlays render on top
- No visible lag between skeleton and camera
- Multiple skeletons rendered simultaneously
- Smooth animation (no jittering)

### Phase 5.5: Connection Status Display
**Goal:** ✓ User can see connection state and transmission rate
- Status indicator shows connected/connecting/disconnected
- Frame rate displayed (e.g., "30 FPS")
- Latency estimated and displayed
- Button to manually connect/disconnect
- Clear error messages if auth fails

### Phase 5.6: Through-Wall Testing
**Goal:** ✓ Verified end-to-end through-wall skeleton visualization
- HawkSight A behind wall (person visible only to A)
- HawkSight B in front of wall (wall blocks direct view)
- HawkSight A streams pose of person
- HawkSight B displays skeleton outline of same person
- Skeleton position aligns with wall location
- Works with 2+ people simultaneously

**Phase 5 Deliverables:**
- [ ] Pose messages received and parsed
- [ ] World→Image projection working
- [ ] Skeletons render on camera overlay
- [ ] Real-time at camera frame rate
- [ ] Status display working
- [ ] End-to-end through-wall visualization verified

---

## PHASE 6: Latency Optimization & Real-Time <50ms
**Duration:** 2-3 weeks  
**Goal:** ✓ Reduce end-to-end latency from ~200ms to <50ms

### Phase 6.1: Baseline Latency Measurement
**Goal:** ✓ Identify current latency bottlenecks
- Measure: Frame capture → transmission latency
- Measure: Aerie reception → pose detection latency
- Measure: Pose transform latency
- Measure: Broadcast → HawkSight render latency
- Measure: Total end-to-end latency
- Create latency breakdown chart
- Identify top 3 bottlenecks

### Phase 6.2: Frame Encoding Optimization
**Goal:** ✓ Faster frame transmission with less data
- Implement H.264 hardware video stream (vs. JPEG)
- Reduce JPEG quality for lower-motion frames
- Implement adaptive frame rate (drop low-motion frames)
- Measure transmission latency improvement
- Target: <50ms transmission time

### Phase 6.3: GPU Acceleration
**Goal:** ✓ Move pose detection to GPU for faster inference
- Implement Metal acceleration for pose model on Aerie (M4 MacBook)
- Enable batch processing (multiple frames in parallel)
- Implement parallel processing on available cores
- Measure inference time improvement
- Target: <30ms pose detection

### Phase 6.4: Network Optimization
**Goal:** ✓ Reduce network round-trip time
- Implement frame pipelining (send while receiving)
- Remove unnecessary serialization overhead
- Use binary protocol option (vs. JSON)
- Reduce message size where possible
- Target: <50ms round-trip

### Phase 6.5: HawkSight Rendering Optimization
**Goal:** ✓ Faster skeleton rendering and display
- Implement view caching to avoid redraws
- Use Core Animation for smooth joint animation
- Reduce draw call count
- Implement predictive rendering (extrapolate between updates)
- Target: <10ms render time

### Phase 6.6: Latency Target Verification
**Goal:** ✓ End-to-end latency consistently <50ms
- Measure across 10+ cycles
- Sustained performance over 5+ minute session
- Works with 2+ HawkSight instances
- Latency jitter <10ms
- Create performance profiling report

**Phase 6 Deliverables:**
- [ ] Baseline latency measured and documented
- [ ] H.264 streaming implemented
- [ ] GPU acceleration active
- [ ] Network optimized
- [ ] HawkSight rendering optimized
- [ ] End-to-end latency <50ms verified

---

## PHASE 7: Multi-Person Tracking & Persistence
**Duration:** 1-2 weeks  
**Goal:** ✓ Track individual people across frames with persistent IDs

### Phase 7.1: Person Identity Assignment
**Goal:** ✓ Assign consistent IDs to same person across time
- Generate unique person IDs on first detection
- Track person across consecutive frames
- Handle person entering/leaving view
- Handle 2-3 people tracked simultaneously
- No ID swaps between people

### Phase 7.2: Temporal Tracking
**Goal:** ✓ Maintain person pose history over time
- Store last 30 frames of pose data per person
- Track person velocity and acceleration
- Predict future position based on history
- Identify when person disappears (lost for >5 frames)

### Phase 7.3: Occlusion Handling
**Goal:** ✓ Predict and render occluded joints
- Detect which joints are hidden
- Interpolate occluded joint positions
- Show confidence indicator for predicted joints
- Graceful degradation when partially visible

### Phase 7.4: Multi-Device Tracking
**Goal:** ✓ Track same person across multiple device views
- Correlate poses from different devices
- Maintain identity when moving between views
- Handle simultaneous views of same person

### Phase 7.5: Tracking Testing
**Goal:** ✓ Verified robust multi-person tracking
- Track 2 people moving independently
- Track 3 people in overlapping space
- No identity swaps over 5-minute session
- Smooth tracking (no jumps/jitter)
- Occlusion handling works

**Phase 7 Deliverables:**
- [ ] Person IDs assigned and persistent
- [ ] Temporal tracking stable
- [ ] Occluded joints predicted
- [ ] Multi-person tracking verified
- [ ] No ID swaps or lost tracking
- [ ] Smooth motion over time

---

## PHASE 8: Spatial Mesh Integration
**Duration:** 2-3 weeks  
**Goal:** ✓ Integrate LiDAR mesh; show walls and geometry through-wall

### Phase 8.1: LiDAR Mesh Extraction (HawkSight)
**Goal:** ✓ Extract and transmit LiDAR mesh from HawkSight
- LiDAR mesh captured from ARKit at keyframes
- Mesh vertices and triangles serialized
- Mesh transmitted to Aerie (or streamed via video)
- Mesh size reasonable (<50MB for room-scale)
- Selective transmission (only when significant change)

### Phase 8.2: Mesh Reception & Storage (Aerie)
**Goal:** ✓ Receive and store meshes from multiple HawkSight instances
- Meshes received and decoded
- Stored per device
- Multiple device meshes stored simultaneously
- Mesh metadata tracked (device, timestamp)

### Phase 8.3: Mesh Merging & Alignment
**Goal:** ✓ Merge meshes from multiple devices into unified map
- Transform each device mesh to world frame
- Merge meshes without duplication
- Handle overlapping geometry
- Unified mesh broadcasted to clients
- Mesh size remains manageable

### Phase 8.4: Mesh Rendering (HawkSight)
**Goal:** ✓ Render unified mesh as semi-transparent overlay
- Receive merged mesh from Aerie
- Render as wireframe or transparent geometry
- Show walls, floors, furniture
- Overlay with skeleton data
- Real-time rendering

### Phase 8.5: Occlusion Culling
**Goal:** ✓ Use mesh to properly occlude skeleton joints
- Determine which joints are behind walls
- Fade or hide occluded joints
- Use ray-casting or similar technique
- Proper depth ordering

### Phase 8.6: Mesh Integration Testing
**Goal:** ✓ Verified walls and geometry visible through-wall
- Scan room with HawkSight A, B
- Meshes merge correctly
- Walls visible in merged mesh
- Skeleton joints properly occluded by walls
- Smooth mesh rendering at 30 FPS

**Phase 8 Deliverables:**
- [ ] LiDAR mesh extracted and transmitted
- [ ] Meshes merged from multiple devices
- [ ] Unified mesh rendered on clients
- [ ] Walls and geometry visible
- [ ] Skeleton occlusion working
- [ ] Mesh bandwidth optimized

---

## PHASE 9: Aerie Dashboard & Monitoring
**Duration:** 1-2 weeks
**Goal:** ✓ Central dashboard for monitoring and control

### Phase 9.1: Connection Status Dashboard
**Goal:** ✓ Real-time view of all connected devices
- List of connected HawkSight instances with status
- Connection time and session ID
- Real-time connection/disconnection events
- Display device location/role (behind wall, in front, etc.)

### Phase 9.2: Pose Visualization
**Goal:** ✓ Visualize detected poses on Aerie dashboard
- Display last frame from each device
- Overlay detected skeletons
- Show confidence scores
- Update in real-time

### Phase 9.3: Performance Metrics
**Goal:** ✓ Monitor latency and throughput
- Frame rate per device
- Pose detection latency
- Network latency
- End-to-end latency
- Memory usage (Aerie and HawkSight instances)
- CPU usage

### Phase 9.4: Recording & Playback
**Goal:** ✓ Record and replay captured data
- Record frames + poses + meshes
- Playback with timeline control
- Export recorded data
- Useful for analysis and debugging

### Phase 9.5: Configuration UI
**Goal:** ✓ Control server from dashboard
- Start/stop pose detection
- Calibrate devices
- Adjust frame rate and quality
- Toggle features on/off

**Phase 9 Deliverables:**
- [ ] Dashboard shows all connected devices
- [ ] Real-time pose visualization
- [ ] Performance metrics displayed
- [ ] Recording and playback working
- [ ] Configuration controls functional

---

## PHASE 10: Quest 3 Integration (Parallel to Phase 6+)
**Duration:** 2-3 weeks
**Goal:** ✓ Stream spatial data to Meta Quest 3; render in mixed reality

### Phase 10.1: Quest WebSocket Client
**Goal:** ✓ Quest 3 connects to Aerie server
- Quest app sends auth request
- Receives and stores session ID
- Maintains connection
- Handles disconnection/reconnection

### Phase 10.2: 3D Skeleton Rendering in MR
**Goal:** ✓ Render skeletons in Quest 3's 3D space
- Receive world-frame poses from server
- Render skeleton in 3D (not 2D overlay)
- Use Quest's spatial tracking for camera matrix
- Render at 90 FPS (Quest refresh rate)

### Phase 10.3: Passthrough Camera Integration
**Goal:** ✓ Show real environment while viewing poses
- Enable Quest passthrough mode
- Show real walls/geometry
- Overlay skeleton on passthrough
- Proper depth ordering

### Phase 10.4: Multi-Person Visualization
**Goal:** ✓ Show multiple people tracked across devices
- Each person rendered with different color
- Persistence of person IDs across views
- Show which device detected each person

### Phase 10.5: MR Interaction & UI
**Goal:** ✓ Allow user interaction with spatial data
- Tap skeleton to inspect person
- UI menus for settings
- Record/playback in MR
- Draw annotations on map

### Phase 10.6: Quest Integration Testing
**Goal:** ✓ Verified end-to-end with Quest 3
- Connect Quest to Aerie
- See skeletons in MR view
- Multi-device poses rendered correctly
- Smooth rendering at 90 FPS

**Phase 10 Deliverables:**
- [ ] Quest 3 connects to server
- [ ] Skeletons rendered in 3D space
- [ ] Passthrough camera working
- [ ] Multi-person visualization
- [ ] MR interaction working
- [ ] End-to-end verified

---

## SUCCESS CRITERIA & FINAL CHECKLIST

### MVP Success (End of Phase 5)
- [ ] Two HawkSight instances connected simultaneously
- [ ] One HawkSight behind wall, one in front
- [ ] Person visible only to HawkSight behind wall
- [ ] HawkSight in front displays skeleton overlay on wall view
- [ ] Skeleton position aligns with actual location
- [ ] Latency <200ms
- [ ] Stable connection for 5+ minutes
- [ ] 0 crashes

### Phase 6 Success (<50ms Latency)
- [ ] End-to-end latency measured consistently <50ms
- [ ] 2+ people tracked simultaneously
- [ ] Smooth rendering without jitter
- [ ] Sustained performance over 30-minute session

### Phase 8 Success (Geometry Visible)
- [ ] Walls rendered through-wall view
- [ ] Skeleton joints properly occluded by walls
- [ ] Mesh from multiple devices merged correctly
- [ ] Real geometry matches actual layout

### Phase 10 Success (Quest Integration)
- [ ] Quest 3 shows same skeletal data as HawkSight instances
- [ ] Rendering smooth at 90 FPS
- [ ] Multi-person tracking works across devices
- [ ] MR passthrough shows environment + skeletons

---

## Key Metrics to Track

| Metric | Target | Phase |
|--------|--------|-------|
| End-to-end latency | <50ms | 6 |
| Frame rate | 30 FPS | 1 |
| Multi-person support | 3+ simultaneous | 7 |
| Devices supported | 3+ | 1 |
| Aerie memory | <1GB | All |
| HawkSight memory | <500MB | All |
| Network bandwidth | <10 Mbps | 2 |
| Pose detection accuracy | >90% | 3 |
| Connection stability | >99% uptime | 1 |

---

## Development Notes

### Architecture Overview
- **Aerie**: Central Swift server (Vapor) handling auth, pose detection, coordinate transforms, and broadcasting
- **HawkSight**: Swift clients capturing frames + poses, authenticating, and rendering skeletons
- **Network**: Local WiFi with WebSocket protocol (JSON messages)
- **Pose Engine**: MediaPipe running on Aerie for each incoming frame
- **Rendering**: SwiftUI + Core Graphics on HawkSight for skeleton overlay

### Tech Stack
- **Aerie**: Swift, Vapor, MediaPipe (via subprocess or native library), SIMD for matrix math
- **HawkSight**: Swift, SwiftUI, ARKit, AVFoundation, CryptoKit for auth
- **Quest 3** (Phase 10): Unity + C# or native XR SDK

### Build & Run
```bash
# Aerie
cd Aerie
swift build
swift run

# HawkSight
Open HawkSight/HawkSight.xcodeproj in Xcode
Update serverURL in config to Aerie's local IP
Build and run on device (iPhone 12 Pro+ or later with LiDAR)
```

### Testing Environment
- Two iPhone 13 Pro/14 Pro/15 Pro (must have LiDAR)
- M4 Max MacBook Pro on same WiFi network
- Real walls for through-wall testing
- Optional: Meta Quest 3 for Phase 10

---

## Risk Mitigation

| Risk | Mitigation |
|------|-----------|
| MediaPipe integration complex | Start with Python subprocess, migrate to Swift wrapper later |
| Coordinate transforms incorrect | Implement unit tests with known inputs/outputs; manual verification |
| Network latency high | Profile early; optimize encoding if needed |
| Multiple device sync issues | Implement strict time synchronization and pose validation |
| Memory leaks in sustained ops | Profile regularly with Xcode Instruments; test 1-hour sessions |
| Quest 3 integration late | Design server protocol to be device-agnostic from Phase 1 |

---

## Timeline Summary

```
Week 1:   Phase 0-1   (Infrastructure, Auth)
Week 2:   Phase 2     (ARKit, Frame transmission)
Week 3:   Phase 3     (Pose detection)
Week 4:   Phase 4     (Coordinate transforms)
Week 5:   Phase 5     (iPhone rendering, MVP complete)
Week 6-8: Phase 6     (Latency optimization to <50ms)
Week 9:   Phase 7     (Multi-person tracking)
Week 10:  Phase 8     (Mesh integration)
Parallel: Phase 9-10  (Dashboard, Quest 3)
```

---

## Resources

### Documentation
- ARKit: https://developer.apple.com/arkit/
- Vapor: https://docs.vapor.codes/
- MediaPipe: https://mediapipe.dev/
- WebSocket: https://tools.ietf.org/html/rfc6455

### Dependencies
- Vapor 4.89+
- WebSocketKit 2.12+
- MediaPipe (via pip or native Swift if available)
- SIMD (built into Swift)
- CryptoKit (built into Swift 5.9+)

---

## Next Steps

1. **Complete Phase 0** by end of Day 1
   - Create both Xcode projects
   - Verify clean builds

2. **Complete Phase 1** by end of Week 1
   - Auth pipeline working
   - Multiple devices connecting

3. **Demo Phase 5 MVP** by end of Week 5
   - Through-wall skeleton visualization
   - Show to team, get feedback

4. **Optimize to <50ms** in Week 6-8
   - Profile and fix bottlenecks
   - Ensure smooth real-time performance

5. **Extend features** in Week 9-10
   - Multi-person tracking
   - Geometry mesh
   - Quest 3 support

Good luck! This is an ambitious project. Take it one phase at a time. 🚀
