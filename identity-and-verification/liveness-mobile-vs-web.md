---
title: "Liveness Detection: Mobile vs Web Implementation"
category: identity-and-verification
summary: "Platform-specific implementation differences for liveness detection — native SDK vs WebRTC, camera access, performance, and security trade-offs."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Liveness Detection: Mobile vs Web Implementation

> Platform-specific implementation differences for liveness detection — native SDK vs WebRTC, camera access, performance, and security trade-offs.

### 9.1 Mobile Native SDK (iOS / Android)

#### Architecture
```
Mobile Native Liveness SDK:
├── Hardware Access:
│   ├── AVFoundation (iOS) / Camera2 API (Android)
│   ├── Direct sensor access: RAW frames, hardware timestamps
│   ├── Depth sensor API (iOS ARKit TrueDepth, Android Depth API)
│   ├── Accelerometer + gyroscope for device motion context
│   └── Biometric hardware: Face ID, Touch ID, fingerprint sensor
├── Processing:
│   ├── On-device ML inference (CoreML / TFLite / ONNX Runtime)
│   ├── Native C/C++ via JNI (Android) or Objective-C/Swift (iOS)
│   └── GPU/NPU acceleration (Apple Neural Engine, Qualcomm AI Engine)
├── Security:
│   ├── Certificate pinning on API communication
│   ├── App Attest (iOS) / Play Integrity API (Android)
│   ├── Secure Enclave storage for biometric templates (iOS)
│   └── Anti-tampering checks at SDK initialization
└── Output:
    ├── Encrypted biometric payload (not raw image)
    └── Device attestation certificate
```

#### iOS-Specific
- **Face ID**: Structured light depth map + IR image processed entirely on-chip by the Secure Enclave — face template never leaves the device
- **ARKit**: Provides 52 facial blend shapes and full face mesh for advanced liveness
- **App Attest**: Cryptographic proof that SDK is running on unmodified Apple hardware
- **Limitation**: Apple restricts third-party access to TrueDepth raw data; most vendors use ARKit's higher-level API

#### Android-Specific
- **Camera2 API**: Raw sensor access, manual exposure control, hardware-level frame timestamps
- **Google Play Integrity API**: Replaces SafetyNet; provides device integrity attestation with three verdict levels
- **Biometric API**: Hardware-backed keystore for biometric credentials
- **Fragmentation challenge**: 24,000+ Android device models with varying camera quality, sensor arrays, and hardware capabilities
- **Root detection**: More complex due to open ecosystem; requires comprehensive check battery

### 9.2 Web Browser Implementation

#### Architecture
```
Web Liveness (WebRTC + WASM):
├── Camera Access:
│   ├── MediaDevices.getUserMedia() → MediaStream
│   ├── WebRTC camera constraints (resolution, frameRate, facingMode)
│   └── No access to: sensor metadata, depth data, hardware timestamps
├── Frame Capture:
│   ├── Canvas 2D: drawImage() from video element → ImageData
│   ├── ImageCapture API (limited browser support)
│   └── Frame rate throttling (browser security policy)
├── ML Inference:
│   ├── TensorFlow.js / ONNX Runtime Web
│   ├── WebAssembly (WASM) for near-native performance
│   ├── WebGL acceleration (GPU compute via shader)
│   └── WebGPU (emerging — Chrome 113+, significant speed boost)
├── Security Limitations:
│   ├── No App Attest equivalent for browsers
│   ├── Virtual camera APIs accessible via browser — harder to detect
│   ├── No hardware-backed key storage
│   └── JavaScript execution environment is inspectable/modifiable
└── Communication:
    └── HTTPS WebSocket or REST API to liveness backend
```

#### Browser-Specific Challenges

| Challenge | Mobile Native | Web Browser |
|---|---|---|
| **Camera metadata access** | Full (manufacturer, model, serial) | None (browser strips EXIF) |
| **Depth data** | Available (ARKit/Android Depth) | Not available |
| **Hardware attestation** | Strong (App Attest, Play Integrity) | None (FIDO WebAuthn is closest) |
| **Virtual camera detection** | Device driver-level | Limited (MediaDevices label inspection) |
| **Anti-tampering** | Native code obfuscation | JavaScript — easily inspectable |
| **ML inference speed** | Native/NPU — fastest | WASM/WebGL — ~3–5x slower |
| **Battery impact** | Moderate | Higher (JS+WebGL rendering) |
| **CORS/CSP restrictions** | N/A | Significant constraints |

#### Browser Compatibility Matrix

| Feature | Chrome | Firefox | Safari | Edge |
|---|---|---|---|---|
| getUserMedia | ✅ | ✅ | ✅ | ✅ |
| ImageCapture API | ✅ | ⚠️ Partial | ❌ | ✅ |
| WebAssembly | ✅ | ✅ | ✅ | ✅ |
| WebGL | ✅ | ✅ | ✅ | ✅ |
| WebGPU | ✅ | ⚠️ Experimental | ⚠️ Preview | ✅ |
| HTTPS required | ✅ Mandatory | ✅ Mandatory | ✅ Mandatory | ✅ Mandatory |
| MediaDevices in iframe | ✅ + permissions | ✅ + permissions | ❌ Blocked | ✅ + permissions |

#### Recommended Web Architecture
```
Web Liveness Best Practices:
├── Frontend SDK (JavaScript/WASM):
│   ├── Passive liveness: Run on-device for low latency
│   ├── Frame selection: Send only highest-quality frames
│   ├── Browser fingerprint: Canvas, WebGL, AudioContext
│   └── Gyroscope API: Device orientation during capture
├── Secure Transport:
│   ├── TLS 1.3 with certificate pinning where possible
│   ├── Short-lived session tokens (< 60 seconds)
│   └── Encrypted frame payload (avoid raw image transmission)
└── Server-Side (authoritative):
    ├── Full liveness model inference (larger models than edge)
    ├── Injection attack detection (metadata + frame analysis)
    └── Deepfake classification (GPU-accelerated)
```

### 9.3 Key Performance Differences

| Metric | Mobile Native | Web (WASM+WebGL) | Web (Server-side) |
|---|---|---|---|
| **Inference latency** | 50–200ms | 200–800ms | 300–1,500ms (+ network) |
| **Model size** | Up to 50MB | Up to 10MB (download) | Unlimited |
| **Privacy** | On-device processing | On-device processing | Server receives image |
| **Security** | Highest | Medium | Depends on transport |
| **Update mechanism** | App store release | CDN instant update | Server-side instant |

---