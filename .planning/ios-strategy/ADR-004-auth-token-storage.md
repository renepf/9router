# ADR-004: Auth Token Storage (Keychain Strategy)

## Status: Accepted

## Context
The iOS app stores OAuth tokens (access + refresh) for multiple AI providers. These are sensitive credentials.

## Decision
**iOS Keychain Services** with per-provider items, protected by biometric (FaceID/TouchID).

## Implementation
```swift
// Each provider connection stored as:
Keychain item:
  service: "com.9router.provider"
  account: "{provider}:{connectionId}"
  value: JSON { accessToken, refreshToken, expiresAt }
  accessControl: .biometryCurrentSet  // Requires FaceID/TouchID
  accessible: .whenUnlockedThisDeviceOnly
```

## Rationale
- Keychain is the only Apple-sanctioned secure storage for credentials
- Hardware-backed encryption on devices with Secure Enclave
- Biometric lock prevents unauthorized access if device is stolen
- Per-device storage (not synced to iCloud Keychain) prevents credential leakage

## Alternatives Considered
- **UserDefaults**: Not encrypted, trivially readable
- **Core Data + encryption**: DIY crypto is error-prone
- **iCloud Keychain sync**: Risk of credentials on untrusted devices
- **Swift Data with .transformable**: No hardware-backed encryption
