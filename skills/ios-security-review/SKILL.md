---
name: ios-security-review
description: Use this skill when implementing authentication, handling user data, working with Keychain, creating network requests, or implementing sensitive features. Provides comprehensive iOS security checklist and patterns.
---

# iOS Security Review Skill

This skill ensures all iOS code follows security best practices and identifies potential vulnerabilities.

## When to Activate

- Implementing authentication or authorization
- Handling sensitive user data
- Working with Keychain or Data Protection
- Creating network requests
- Implementing biometric authentication
- Storing or transmitting sensitive data
- Integrating third-party SDKs
- Handling payments or financial data

## Security Checklist

### 1. Keychain for Sensitive Data

#### ❌ NEVER Do This
```swift
// Storing secrets in UserDefaults
UserDefaults.standard.set(accessToken, forKey: "accessToken")
UserDefaults.standard.set(password, forKey: "password")

// Storing in plain text files
try password.write(to: fileURL, atomically: true, encoding: .utf8)
```

#### ✅ ALWAYS Do This
```swift
import Security

enum KeychainError: Error {
    case duplicateEntry
    case unknown(OSStatus)
    case itemNotFound
}

final class KeychainManager {
    static let shared = KeychainManager()
    private init() {}

    func save(_ data: Data, service: String, account: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly
        ]

        let status = SecItemAdd(query as CFDictionary, nil)

        if status == errSecDuplicateItem {
            try update(data, service: service, account: account)
        } else if status != errSecSuccess {
            throw KeychainError.unknown(status)
        }
    }

    func read(service: String, account: String) throws -> Data {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecReturnData as String: true
        ]

        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)

        guard status == errSecSuccess, let data = result as? Data else {
            throw KeychainError.itemNotFound
        }

        return data
    }

    func delete(service: String, account: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account
        ]

        let status = SecItemDelete(query as CFDictionary)

        guard status == errSecSuccess || status == errSecItemNotFound else {
            throw KeychainError.unknown(status)
        }
    }

    private func update(_ data: Data, service: String, account: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account
        ]

        let attributes: [String: Any] = [
            kSecValueData as String: data
        ]

        let status = SecItemUpdate(query as CFDictionary, attributes as CFDictionary)

        guard status == errSecSuccess else {
            throw KeychainError.unknown(status)
        }
    }
}

// Usage
try KeychainManager.shared.save(
    token.data(using: .utf8)!,
    service: "com.app.auth",
    account: "accessToken"
)
```

#### Verification Steps
- [ ] All tokens, passwords, and secrets stored in Keychain
- [ ] Appropriate `kSecAttrAccessible` level set
- [ ] No sensitive data in UserDefaults
- [ ] No sensitive data in plain files
- [ ] Keychain items cleaned on logout

### 2. App Transport Security (ATS)

#### ✅ Keep ATS Enabled
```xml
<!-- Info.plist - Default (recommended) -->
<!-- No NSAppTransportSecurity key needed = ATS fully enabled -->

<!-- Only if exceptions truly necessary -->
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
        </dict>
    </dict>
</dict>
```

#### ❌ NEVER Disable ATS Globally
```xml
<!-- DANGEROUS - Never do this in production -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

#### Verification Steps
- [ ] ATS enabled (no arbitrary loads)
- [ ] TLS 1.2 minimum enforced
- [ ] Only specific exception domains if needed
- [ ] Certificate pinning for sensitive APIs

### 3. Certificate Pinning

```swift
import CryptoKit

final class CertificatePinner: NSObject, URLSessionDelegate {
    private let pinnedCertificates: Set<String>

    init(pinnedCertificates: Set<String>) {
        self.pinnedCertificates = pinnedCertificates
        super.init()
    }

    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
              let serverTrust = challenge.protectionSpace.serverTrust else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }

        if let serverCertificate = SecTrustCopyCertificateChain(serverTrust) as? [SecCertificate],
           let certificate = serverCertificate.first {
            let serverCertData = SecCertificateCopyData(certificate) as Data
            let serverCertHash = SHA256.hash(data: serverCertData)
            let serverCertHashString = serverCertHash.compactMap { String(format: "%02x", $0) }.joined()

            if pinnedCertificates.contains(serverCertHashString) {
                completionHandler(.useCredential, URLCredential(trust: serverTrust))
                return
            }
        }

        completionHandler(.cancelAuthenticationChallenge, nil)
    }
}
```

### 4. Biometric Authentication

```swift
import LocalAuthentication

final class BiometricAuthManager {
    enum BiometricError: Error {
        case notAvailable
        case notEnrolled
        case authenticationFailed
        case userCancelled
        case unknown(Error)
    }

    func authenticate(reason: String) async throws -> Bool {
        let context = LAContext()
        var error: NSError?

        guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
            if let error {
                switch error.code {
                case LAError.biometryNotAvailable.rawValue:
                    throw BiometricError.notAvailable
                case LAError.biometryNotEnrolled.rawValue:
                    throw BiometricError.notEnrolled
                default:
                    throw BiometricError.unknown(error)
                }
            }
            throw BiometricError.notAvailable
        }

        do {
            return try await context.evaluatePolicy(
                .deviceOwnerAuthenticationWithBiometrics,
                localizedReason: reason
            )
        } catch let error as LAError {
            switch error.code {
            case .userCancel, .userFallback:
                throw BiometricError.userCancelled
            case .authenticationFailed:
                throw BiometricError.authenticationFailed
            default:
                throw BiometricError.unknown(error)
            }
        }
    }

    var biometryType: LABiometryType {
        let context = LAContext()
        _ = context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: nil)
        return context.biometryType
    }
}

// Usage
let authManager = BiometricAuthManager()
do {
    let success = try await authManager.authenticate(
        reason: "Authenticate to access your account"
    )
    if success {
        // Proceed with sensitive operation
    }
} catch {
    // Handle error
}
```

#### Verification Steps
- [ ] Fallback to passcode available
- [ ] Biometric changes detected (invalidation)
- [ ] Reason string is clear and localized
- [ ] Not using biometrics for initial login only

### 5. Data Protection

```swift
// Set file protection level
func saveSecureFile(_ data: Data, to url: URL) throws {
    try data.write(to: url, options: .completeFileProtection)
}

// Check file protection
func verifyFileProtection(at url: URL) -> Bool {
    do {
        let attributes = try FileManager.default.attributesOfItem(atPath: url.path)
        let protection = attributes[.protectionKey] as? FileProtectionType
        return protection == .complete || protection == .completeUnlessOpen
    } catch {
        return false
    }
}

// Set Core Data store protection
let container = NSPersistentContainer(name: "Model")
let description = container.persistentStoreDescriptions.first
description?.setOption(
    FileProtectionType.complete as NSObject,
    forKey: NSPersistentStoreFileProtectionKey
)
```

#### Protection Levels
| Level | When Available |
|-------|----------------|
| `.complete` | Only when device unlocked |
| `.completeUnlessOpen` | After first unlock, existing handles OK |
| `.completeUntilFirstUserAuthentication` | After first unlock (default) |
| `.none` | Always (not recommended) |

### 6. Input Validation

```swift
// ✅ GOOD: Validate all user input
struct UserInputValidator {
    enum ValidationError: Error {
        case invalidEmail
        case passwordTooWeak
        case invalidPhoneNumber
        case inputTooLong
        case containsInvalidCharacters
    }

    static func validateEmail(_ email: String) throws {
        let emailRegex = #"^[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$"#
        guard email.range(of: emailRegex, options: .regularExpression) != nil else {
            throw ValidationError.invalidEmail
        }
    }

    static func validatePassword(_ password: String) throws {
        guard password.count >= 8 else {
            throw ValidationError.passwordTooWeak
        }

        let hasUppercase = password.contains(where: { $0.isUppercase })
        let hasLowercase = password.contains(where: { $0.isLowercase })
        let hasNumber = password.contains(where: { $0.isNumber })

        guard hasUppercase && hasLowercase && hasNumber else {
            throw ValidationError.passwordTooWeak
        }
    }

    static func sanitizeInput(_ input: String, maxLength: Int = 1000) throws -> String {
        guard input.count <= maxLength else {
            throw ValidationError.inputTooLong
        }

        // Remove potentially dangerous characters
        let sanitized = input
            .replacingOccurrences(of: "<", with: "")
            .replacingOccurrences(of: ">", with: "")
            .replacingOccurrences(of: "\"", with: "")

        return sanitized
    }
}
```

### 7. Network Security

```swift
// ✅ GOOD: Secure network requests
final class SecureNetworkClient {
    private let session: URLSession
    private let baseURL: URL

    init(baseURL: URL, pinnedCertificates: Set<String>? = nil) {
        self.baseURL = baseURL

        let configuration = URLSessionConfiguration.default
        configuration.tlsMinimumSupportedProtocolVersion = .TLSv12
        configuration.urlCache = nil  // Disable caching for sensitive requests

        if let pins = pinnedCertificates {
            let pinner = CertificatePinner(pinnedCertificates: pins)
            self.session = URLSession(configuration: configuration, delegate: pinner, delegateQueue: nil)
        } else {
            self.session = URLSession(configuration: configuration)
        }
    }

    func request<T: Decodable>(
        endpoint: String,
        method: String = "GET",
        body: Data? = nil,
        token: String? = nil
    ) async throws -> T {
        var request = URLRequest(url: baseURL.appendingPathComponent(endpoint))
        request.httpMethod = method
        request.httpBody = body
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")

        if let token {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }

        let (data, response) = try await session.data(for: request)

        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }

        guard (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.serverError(httpResponse.statusCode)
        }

        return try JSONDecoder().decode(T.self, from: data)
    }
}
```

### 8. Jailbreak Detection

```swift
struct JailbreakDetector {
    static func isJailbroken() -> Bool {
        #if targetEnvironment(simulator)
        return false
        #else
        // Check for common jailbreak files
        let jailbreakPaths = [
            "/Applications/Cydia.app",
            "/Library/MobileSubstrate/MobileSubstrate.dylib",
            "/bin/bash",
            "/usr/sbin/sshd",
            "/etc/apt",
            "/private/var/lib/apt/"
        ]

        for path in jailbreakPaths {
            if FileManager.default.fileExists(atPath: path) {
                return true
            }
        }

        // Check if app can write outside sandbox
        let testPath = "/private/jailbreak_test.txt"
        do {
            try "test".write(toFile: testPath, atomically: true, encoding: .utf8)
            try FileManager.default.removeItem(atPath: testPath)
            return true
        } catch {
            // Expected behavior on non-jailbroken devices
        }

        // Check if app can open Cydia URL
        if let url = URL(string: "cydia://package/com.example.package"),
           UIApplication.shared.canOpenURL(url) {
            return true
        }

        return false
        #endif
    }
}

// Usage
if JailbreakDetector.isJailbroken() {
    // Show warning or restrict functionality
    showSecurityWarning()
}
```

### 9. Logging & Debugging

```swift
// ✅ GOOD: Safe logging
enum SecureLogger {
    static func log(_ message: String, level: LogLevel = .info) {
        #if DEBUG
        print("[\(level.rawValue)] \(message)")
        #endif
    }

    static func logSensitive(_ message: String) {
        #if DEBUG
        print("[SENSITIVE] \(message.prefix(3))***")
        #endif
    }

    enum LogLevel: String {
        case info = "INFO"
        case warning = "WARNING"
        case error = "ERROR"
    }
}

// ❌ BAD: Logging sensitive data
print("User token: \(token)")
print("Password: \(password)")
NSLog("API Key: %@", apiKey)
```

#### Verification Steps
- [ ] No sensitive data in logs
- [ ] Debug logging disabled in release
- [ ] No NSLog with sensitive data
- [ ] No print statements in production

### 10. Code Obfuscation & Anti-Tampering

```swift
// Verify app signature
func verifyAppSignature() -> Bool {
    guard let executablePath = Bundle.main.executablePath else {
        return false
    }

    var staticCode: SecStaticCode?
    guard SecStaticCodeCreateWithPath(
        executablePath as CFURL,
        [],
        &staticCode
    ) == errSecSuccess else {
        return false
    }

    guard let code = staticCode else { return false }

    var requirement: SecRequirement?
    guard SecRequirementCreateWithString(
        "anchor apple generic" as CFString,
        [],
        &requirement
    ) == errSecSuccess else {
        return false
    }

    return SecStaticCodeCheckValidity(
        code,
        [],
        requirement
    ) == errSecSuccess
}
```

### 11. Clipboard Security

```swift
// ✅ GOOD: Expire sensitive clipboard data
func copyToClipboard(_ text: String, expiresIn seconds: TimeInterval = 60) {
    UIPasteboard.general.setItems(
        [[UIPasteboard.typeAutomatic: text]],
        options: [
            .expirationDate: Date().addingTimeInterval(seconds),
            .localOnly: true
        ]
    )
}

// Clear sensitive data when app backgrounds
NotificationCenter.default.addObserver(
    forName: UIApplication.willResignActiveNotification,
    object: nil,
    queue: .main
) { _ in
    UIPasteboard.general.items = []
}
```

### 12. WebView Security

```swift
import WebKit

final class SecureWebViewController: UIViewController {
    private lazy var webView: WKWebView = {
        let configuration = WKWebViewConfiguration()

        // Disable JavaScript if not needed
        configuration.defaultWebpagePreferences.allowsContentJavaScript = false

        // Disable data detection
        configuration.dataDetectorTypes = []

        let webView = WKWebView(frame: .zero, configuration: configuration)
        webView.navigationDelegate = self
        return webView
    }()

    func loadURL(_ url: URL) {
        // Only allow HTTPS
        guard url.scheme == "https" else {
            showError("Insecure URL not allowed")
            return
        }

        // Whitelist domains
        let allowedDomains = ["example.com", "api.example.com"]
        guard let host = url.host, allowedDomains.contains(host) else {
            showError("Domain not allowed")
            return
        }

        webView.load(URLRequest(url: url))
    }
}

extension SecureWebViewController: WKNavigationDelegate {
    func webView(
        _ webView: WKWebView,
        decidePolicyFor navigationAction: WKNavigationAction,
        decisionHandler: @escaping (WKNavigationActionPolicy) -> Void
    ) {
        guard let url = navigationAction.request.url,
              url.scheme == "https" else {
            decisionHandler(.cancel)
            return
        }

        decisionHandler(.allow)
    }
}
```

## Pre-Release Security Checklist

Before ANY App Store submission:

- [ ] **Keychain**: All sensitive data in Keychain
- [ ] **ATS**: App Transport Security enabled
- [ ] **Certificate Pinning**: Implemented for critical APIs
- [ ] **Biometrics**: Proper fallback handling
- [ ] **Data Protection**: Complete protection for sensitive files
- [ ] **Input Validation**: All user input validated
- [ ] **Network**: TLS 1.2+ enforced
- [ ] **Logging**: No sensitive data in logs
- [ ] **Jailbreak**: Detection implemented (if required)
- [ ] **Clipboard**: Sensitive data expires
- [ ] **WebView**: Only HTTPS, domain whitelist
- [ ] **Debug**: Disabled in release builds
- [ ] **Third-party SDKs**: Security reviewed
- [ ] **Privacy**: Data usage disclosed in Info.plist

## Resources

- [Apple Security Documentation](https://developer.apple.com/documentation/security)
- [OWASP Mobile Security Guide](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Apple Keychain Services](https://developer.apple.com/documentation/security/keychain_services)
- [App Transport Security](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity)

---

**Remember**: Security is not optional. One vulnerability can compromise user data and trust. When in doubt, err on the side of caution.
