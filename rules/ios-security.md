# iOS Security Guidelines

> iOS 앱 보안을 위한 필수 체크리스트 및 가이드라인
> Based on OWASP Mobile Top 10 and Apple Security Best Practices

---

## Mandatory Security Checks (Before Commit)

### Data Security
- [ ] No hardcoded secrets (API keys, tokens, credentials)
- [ ] Keychain for all sensitive data (with appropriate accessibility level)
- [ ] Data Protection enabled for sensitive files
- [ ] In-memory sensitive data cleared after use
- [ ] No secrets in UserDefaults or Info.plist

### Network Security
- [ ] ATS enabled (no arbitrary loads without justification)
- [ ] SSL Pinning for critical APIs
- [ ] Ephemeral sessions for sensitive requests
- [ ] No credentials in logs (print, os_log, NSLog)

### Authentication Security
- [ ] Biometric auth with domain state validation (iOS 16+)
- [ ] Secure token storage in Keychain
- [ ] Re-authentication for sensitive operations

### Privacy & Compliance
- [ ] Privacy Manifest up to date (iOS 17+)
- [ ] Required Reason APIs documented
- [ ] User Tracking Transparency implemented (if tracking)

### UI Security
- [ ] Screenshot/recording protection for sensitive screens
- [ ] Background snapshot blur/hide
- [ ] Clipboard expiration for sensitive data

### Integrity (Financial/Enterprise Apps)
- [ ] Jailbreak detection
- [ ] Debugger detection
- [ ] App Attest integration (iOS 14+)

---

## Secret Management

### Build-time Secrets (xcconfig)

```swift
// Config.xcconfig (Git에서 제외)
API_KEY = your_api_key_here
API_SECRET = your_secret_here

// Info.plist
// <key>APIKey</key><string>$(API_KEY)</string>

// 코드에서 사용
guard let apiKey = Bundle.main.object(forInfoDictionaryKey: "APIKey") as? String else {
    fatalError("APIKey not configured")
}
```

### Runtime Secrets (Keychain)

```swift
import Security

// NEVER
let apiKey = "sk-proj-xxxxx"  // 하드코딩 금지
UserDefaults.standard.set(token, forKey: "authToken")  // UserDefaults 금지

// ALWAYS: Keychain 사용
enum KeychainAccessibility {
    /// 일반 토큰
    static let standard = kSecAttrAccessibleWhenUnlocked
    /// 민감한 토큰 (권장)
    static let sensitive = kSecAttrAccessibleWhenUnlockedThisDeviceOnly
    /// 백그라운드 접근 필요 시
    static let background = kSecAttrAccessibleAfterFirstUnlock
    /// 최고 보안 (Passcode 필수)
    static let maximum = kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly
}

func saveToKeychain(
    key: String,
    data: Data,
    accessibility: CFString = KeychainAccessibility.sensitive
) throws {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecValueData as String: data,
        kSecAttrAccessible as String: accessibility
    ]

    // 기존 항목 삭제 후 추가
    SecItemDelete(query as CFDictionary)

    let status = SecItemAdd(query as CFDictionary, nil)
    guard status == errSecSuccess else {
        throw KeychainError.saveFailed(status)
    }
}

func loadFromKeychain(key: String) throws -> Data {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecReturnData as String: true,
        kSecMatchLimit as String: kSecMatchLimitOne
    ]

    var result: AnyObject?
    let status = SecItemCopyMatching(query as CFDictionary, &result)

    guard status == errSecSuccess, let data = result as? Data else {
        throw KeychainError.loadFailed(status)
    }
    return data
}

func deleteFromKeychain(key: String) {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key
    ]
    SecItemDelete(query as CFDictionary)
}
```

---

## Data Storage Security

### Data Protection API

```swift
// 파일 생성 시 보호 수준 설정
try data.write(
    to: fileURL,
    options: [.atomic, .completeFileProtection]
)

// 또는 FileManager로 설정
try FileManager.default.setAttributes(
    [.protectionKey: FileProtectionType.complete],
    ofItemAtPath: filePath
)
```

### In-Memory Data Clearing

```swift
// 민감한 데이터 사용 후 메모리에서 제거
var sensitiveData = Data(/* ... */)
defer {
    sensitiveData.resetBytes(in: 0..<sensitiveData.count)
}

// 사용 로직
processSensitiveData(sensitiveData)
// defer로 자동 클리어
```

### Core Data / SwiftData Encryption

```swift
// Core Data: SQLite 암호화 (SQLCipher 또는 Encrypted Core Data)
let description = NSPersistentStoreDescription()
description.setOption(
    FileProtectionType.complete as NSObject,
    forKey: NSPersistentStoreFileProtectionKey
)

// SwiftData (iOS 17+): 자동으로 Data Protection 적용
@Model
class SensitiveRecord {
    var encryptedPayload: Data  // 추가 암호화 권장
}
```

---

## Network Security

### App Transport Security (ATS)

```xml
<!-- Info.plist - ATS 기본 활성화 (권장) -->
<!-- 예외가 필요한 경우에만 명시 -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>legacy-api.example.com</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.2</string>
            <!-- 예외 사유 문서화 필수 -->
        </dict>
    </dict>
</dict>
```

### SSL Pinning

```swift
class SSLPinningDelegate: NSObject, URLSessionDelegate {
    private let pinnedCertificateHash: String  // SHA256 해시

    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge
    ) async -> (URLSession.AuthChallengeDisposition, URLCredential?) {

        guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
              let serverTrust = challenge.protectionSpace.serverTrust,
              let certificate = SecTrustCopyCertificateChain(serverTrust)?.first else {
            return (.cancelAuthenticationChallenge, nil)
        }

        // 인증서 해시 비교
        let serverCertData = SecCertificateCopyData(certificate) as Data
        let serverHash = SHA256.hash(data: serverCertData)
            .map { String(format: "%02x", $0) }
            .joined()

        guard serverHash == pinnedCertificateHash else {
            return (.cancelAuthenticationChallenge, nil)
        }

        return (.useCredential, URLCredential(trust: serverTrust))
    }
}
```

### Ephemeral Sessions (Sensitive APIs)

```swift
// 민감한 요청용 - 캐시/쿠키 비활성화
let ephemeralConfig = URLSessionConfiguration.ephemeral
ephemeralConfig.urlCache = nil
ephemeralConfig.httpCookieStorage = nil
ephemeralConfig.requestCachePolicy = .reloadIgnoringLocalCacheData

let secureSession = URLSession(
    configuration: ephemeralConfig,
    delegate: sslPinningDelegate,
    delegateQueue: nil
)
```

---

## Authentication Security

### Biometric Authentication

```swift
import LocalAuthentication

actor BiometricAuthenticator {
    private var savedDomainState: Data?

    func authenticate() async throws -> Bool {
        let context = LAContext()
        var error: NSError?

        // 생체인증 가능 여부 확인
        guard context.canEvaluatePolicy(
            .deviceOwnerAuthenticationWithBiometrics,
            error: &error
        ) else {
            throw AuthError.biometricNotAvailable(error)
        }

        // 보안 강화 설정
        context.touchIDAuthenticationAllowableReuseDuration = 0  // 매번 재인증

        // iOS 16+: 생체인증 정보 변경 감지
        if let currentState = context.evaluatedPolicyDomainState,
           let saved = savedDomainState,
           currentState != saved {
            throw AuthError.biometricChanged
        }

        let success = try await context.evaluatePolicy(
            .deviceOwnerAuthenticationWithBiometrics,
            localizedReason: "계정 접근을 위해 인증이 필요합니다"
        )

        // 도메인 상태 저장
        savedDomainState = context.evaluatedPolicyDomainState

        return success
    }
}
```

### OAuth with ASWebAuthenticationSession

```swift
import AuthenticationServices

func signInWithOAuth() async throws -> String {
    let authURL = URL(string: "https://provider.com/oauth/authorize?...")!
    let callbackScheme = "myapp"

    return try await withCheckedThrowingContinuation { continuation in
        let session = ASWebAuthenticationSession(
            url: authURL,
            callbackURLScheme: callbackScheme
        ) { callbackURL, error in
            if let error = error {
                continuation.resume(throwing: error)
                return
            }

            guard let url = callbackURL,
                  let token = URLComponents(url: url, resolvingAgainstBaseURL: false)?
                    .queryItems?.first(where: { $0.name == "token" })?.value else {
                continuation.resume(throwing: AuthError.invalidCallback)
                return
            }

            continuation.resume(returning: token)
        }

        session.presentationContextProvider = self
        session.prefersEphemeralWebBrowserSession = true  // 세션 공유 방지
        session.start()
    }
}
```

---

## Privacy Compliance

### Privacy Manifest (iOS 17+)

```xml
<!-- PrivacyInfo.xcprivacy -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSPrivacyTracking</key>
    <false/>

    <key>NSPrivacyTrackingDomains</key>
    <array/>

    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeName</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <false/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>
    </array>

    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <!-- UserDefaults -->
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string>
            </array>
        </dict>
        <!-- File Timestamp -->
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>C617.1</string>
            </array>
        </dict>
    </array>
</dict>
</plist>
```

### App Tracking Transparency

```swift
import AppTrackingTransparency

func requestTrackingPermission() async -> Bool {
    guard ATTrackingManager.trackingAuthorizationStatus == .notDetermined else {
        return ATTrackingManager.trackingAuthorizationStatus == .authorized
    }

    let status = await ATTrackingManager.requestTrackingAuthorization()
    return status == .authorized
}
```

---

## UI Security

### Screenshot Protection

```swift
// 민감한 화면 보호
class SecureViewController: UIViewController {
    private var secureField: UITextField?

    override func viewDidLoad() {
        super.viewDidLoad()
        setupScreenshotProtection()
    }

    private func setupScreenshotProtection() {
        // isSecureTextEntry를 이용한 보호 (스크린샷에 표시 안 됨)
        let field = UITextField()
        field.isSecureTextEntry = true
        view.addSubview(field)
        field.centerYAnchor.constraint(equalTo: view.centerYAnchor).isActive = true
        field.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true

        // 보호할 뷰를 field의 서브뷰로 추가
        view.layer.superlayer?.addSublayer(field.layer)
        field.layer.sublayers?.first?.addSublayer(sensitiveView.layer)

        secureField = field
    }
}
```

### Background Snapshot Protection

```swift
// SceneDelegate 또는 AppDelegate
func sceneWillResignActive(_ scene: UIScene) {
    // 백그라운드 전환 시 민감 정보 숨김
    showPrivacyOverlay()
}

func sceneDidBecomeActive(_ scene: UIScene) {
    hidePrivacyOverlay()
}

private func showPrivacyOverlay() {
    let blurEffect = UIBlurEffect(style: .light)
    let blurView = UIVisualEffectView(effect: blurEffect)
    blurView.tag = 999
    blurView.frame = window?.bounds ?? .zero
    window?.addSubview(blurView)
}

private func hidePrivacyOverlay() {
    window?.viewWithTag(999)?.removeFromSuperview()
}
```

### Clipboard Security

```swift
// 민감한 데이터 복사 시 만료 시간 설정
UIPasteboard.general.setItems(
    [[UIPasteboard.typeAutomatic: sensitiveText]],
    options: [
        .expirationDate: Date().addingTimeInterval(60),  // 60초 후 만료
        .localOnly: true  // 다른 기기와 공유 안 함
    ]
)
```

---

## App Integrity (Optional)

### Jailbreak Detection

```swift
func isDeviceCompromised() -> Bool {
    #if targetEnvironment(simulator)
    return false
    #else

    // 1. 의심 파일 존재 확인
    let suspiciousPaths = [
        "/Applications/Cydia.app",
        "/Library/MobileSubstrate/MobileSubstrate.dylib",
        "/bin/bash",
        "/usr/sbin/sshd",
        "/etc/apt",
        "/private/var/lib/apt/"
    ]

    for path in suspiciousPaths {
        if FileManager.default.fileExists(atPath: path) {
            return true
        }
    }

    // 2. 샌드박스 외부 쓰기 권한 확인
    let testPath = "/private/jailbreak_test_\(UUID().uuidString)"
    do {
        try "test".write(toFile: testPath, atomically: true, encoding: .utf8)
        try FileManager.default.removeItem(atPath: testPath)
        return true  // 쓰기 성공 = 탈옥
    } catch {
        // 쓰기 실패 = 정상
    }

    // 3. URL scheme 확인
    if let url = URL(string: "cydia://package/com.example"),
       UIApplication.shared.canOpenURL(url) {
        return true
    }

    return false
    #endif
}
```

### App Attest (iOS 14+)

```swift
import DeviceCheck

actor AppAttestService {
    private var keyId: String?

    func generateKey() async throws -> String {
        let service = DCAppAttestService.shared

        guard service.isSupported else {
            throw AppAttestError.notSupported
        }

        let keyId = try await service.generateKey()
        self.keyId = keyId
        return keyId
    }

    func attestKey(challenge: Data) async throws -> Data {
        guard let keyId = keyId else {
            throw AppAttestError.keyNotGenerated
        }

        let service = DCAppAttestService.shared
        let attestation = try await service.attestKey(keyId, clientDataHash: challenge)

        // 서버로 attestation 전송하여 검증
        return attestation
    }

    func generateAssertion(for request: Data) async throws -> Data {
        guard let keyId = keyId else {
            throw AppAttestError.keyNotGenerated
        }

        let service = DCAppAttestService.shared
        let hash = Data(SHA256.hash(data: request))

        return try await service.generateAssertion(keyId, clientDataHash: hash)
    }
}
```

---

## Security Response Protocol

### Issue Discovery Flow

```
Security Issue Found
        │
        ▼
   ┌────────────┐
   │   STOP     │ ← 즉시 작업 중단
   └────────────┘
        │
        ▼
   ┌────────────────────┐
   │ Assess Severity    │
   │ (CRITICAL/HIGH/    │
   │  MEDIUM/LOW)       │
   └────────────────────┘
        │
        ├─── CRITICAL ──▶ Fix before ANY commit
        │                 (Data exposure, credential leak)
        │
        ├─── HIGH ──────▶ Fix before release
        │                 (Missing encryption, insecure storage)
        │
        ├─── MEDIUM ────▶ Fix in next sprint
        │                 (Missing pinning, weak validation)
        │
        └─── LOW ───────▶ Add to backlog
                          (Best practice improvements)
```

### If Credentials Exposed

1. **Rotate immediately** - API keys, tokens, certificates
2. **Revoke sessions** - Force logout affected users
3. **Audit logs** - Check for unauthorized access
4. **Notify team** - Security incident report
5. **Review codebase** - Search for similar issues
6. **Update secrets** - All environments (dev, staging, prod)

### Use ios-security-reviewer Agent

```
보안 이슈 발견 시:
1. ios-security-reviewer 에이전트 호출
2. OWASP Mobile Top 10 기준 분석
3. 심각도별 우선순위 결정
4. 수정 계획 수립
```

---

## Quick Reference

### Severity Levels

| Level | Examples | Action |
|-------|----------|--------|
| CRITICAL | Hardcoded secrets, unencrypted credentials | Immediate fix |
| HIGH | Missing SSL pinning, insecure storage | Fix before release |
| MEDIUM | Missing rate limiting, weak validation | Next sprint |
| LOW | Missing obfuscation, verbose logging | Backlog |

### iOS Version Requirements

| Feature | Minimum iOS |
|---------|-------------|
| Privacy Manifest | iOS 17.0 |
| evaluatedPolicyDomainState | iOS 16.0 |
| App Attest | iOS 14.0 |
| ASWebAuthenticationSession | iOS 12.0 |
| Biometric (Face ID/Touch ID) | iOS 8.0 |

---

## Related Resources

- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
- [Apple Security Documentation](https://developer.apple.com/documentation/security)
- [App Store Review Guidelines - Safety](https://developer.apple.com/app-store/review/guidelines/#safety)
- [Privacy Manifest Requirements](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files)
