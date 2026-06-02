# **Enterprise Flutter Security: Architectural Design, Secure Development Lifecycle, and Advanced Implementation Techniques**

The rapid adoption of cross-platform mobile frameworks has fundamentally reshaped the mobile application threat landscape. With the release of the updated OWASP Mobile Top 10, the paradigm of mobile security has shifted from isolated client-side validations to a comprehensive model addressing supply chain vulnerabilities, runtime tamper resilience, and secure cryptographic lifecycles.1 Flutter, utilizing the Dart language, compiles code Ahead-of-Time (AOT) into native ARM machine code, which natively limits runtime reflection compared to interpretation-heavy frameworks.3 However, native compilation does not yield security by obscurity. Instead, it alters the attack vectors, prompting sophisticated adversaries to employ reverse engineering tools, dynamic binary instrumentation, and transport-layer redirection.3 This blueprint defines the architectural controls and high-level development skills required to engineer resilient Flutter applications.

## **The Flutter Security Paradigm and the Secure Software Development Lifecycle**

Traditional mobile security tools rely on stable, predictable compilation models where code structure is retained, enabling static application security testing (SAST) tools to trace control flows and dynamic application security testing (DAST) tools to instrument runtime behaviors.4 Flutter completely breaks this traditional testing paradigm.4

### **Why Traditional Security Testing Fails for Flutter**

Traditional SAST scanners analyze the source code or intermediate bytecode (such as JVM bytecode on Android or Objective-C runtime metadata on iOS) to trace data flows and identify vulnerabilities.4 When Flutter compiles an application for production, the Dart source code is compiled Ahead-of-Time (AOT) into a flattened, native machine code binary.4 Meaningful structures, variable names, and logical relationships are stripped or heavily optimized.4 Consequently, traditional SAST scanners suffer from a loss of visibility, failing to trace data flows or compile accurate dependency graphs.4  
Similarly, traditional DAST tools depend on runtime instrumentation hooks to monitor and manipulate application states during execution.4 Because Flutter compiles to native AOT code, these standard runtime hooks are removed, which limits the ability of traditional dynamic tools to observe execution paths.4  
Furthermore, traditional automated UI testing tools navigate applications by scanning the operating system's native UI component tree.4 Flutter does not use native platform UI elements.4 Instead, the framework renders its interface directly onto a flat canvas using a custom rendering engine (Skia or Impeller).4 Because automated tools cannot inspect the native view hierarchies, UI-driven security testing coverage is severely reduced, which often creates a false sense of security.4

| Testing Dimension | Traditional AppSec Tools | Modern Flutter-First AppSec Tools |
| :---- | :---- | :---- |
| **Primary Analysis Artifact** | Source code or intermediate bytecode 4 | Production-ready native compiled binary (APK/IPA) 4 |
| **Static Code Coverage** | Highly readable function mapping and class tracing 4 | Reconstructed code patterns and binary-first static analysis 4 |
| **Dynamic Execution Inspection** | Standard runtime instrumentation hooks (JVM/Objective-C runtime) 4 | Native-level dynamic analysis and network traffic monitoring 4 |
| **UI Path Automation** | Inspections of platform-specific view hierarchies (Android Views / iOS UIViews) 4 | Custom rendering engine coordinate mapping and automated API-level fuzzing 4 |

### **The "Shift-Left" Philosophy and the Impartial Gatekeeper**

To address these visibility gaps, security must be integrated directly into the software development lifecycle (SDLC).10 Shifting security to the earliest possible stages of development ensures that vulnerabilities are caught before they reach compiled production binaries.10  
The foundation of a secure development workflow is the configuration of the Dart analyzer as an impartial gatekeeper in the Continuous Integration (CI) pipeline.12 By writing a strict configuration in the analysis\_options.yaml file located at the project root, developers can enforce type safety and catch logical flaws at compile time 12:

YAML  
include: package:flutter\_lints/flutter.yaml

analyzer:  
  language:  
    strict-casts: true  
    strict-inference: true  
    strict-raw-types: true

Enforcing these strict language flags addresses several common programming vulnerabilities 12:

* **strict-casts: true**: Prevents runtime errors caused by type coercion failure.12 For example, if a generic API payload returns a List\<Object\>, casting it implicitly to a List\<String\> raises a compile-time issue, forcing explicit validation before processing.12  
* **strict-inference: true**: Forces the type inference engine to be more conservative, flagging issues to ensure types are explicitly understood and safely handled rather than loosely inferred.12  
* **strict-raw-types: true**: Requires all generic classes to include explicit type arguments, ensuring that generic constructs like Map must be declared with explicit parameters (e.g., Map\<String, dynamic\>) to maintain code maintainability.12

For enterprise quality gates, static analysis should be extended with advanced tools like Dart Code Metrics (DCM).12 DCM scans the codebase for complex nested controls, memory leaks, and dead code.12 It also enables dependency audits to detect unused or over-privileged libraries, which helps minimize the application's attack surface.12

## **Identity, Access Control, and Permission Architectures**

Modern mobile applications must treat the client device as untrusted.13 A fundamental security axiom is that the mobile client is merely the *requestor*, whereas the backend remains the sole *enforcer* of authorization checks.13

### **Enterprise Authentication Integration**

To implement robust authentication, applications should leverage industry-standard protocols like OAuth 2.0 or OpenID Connect rather than rolling proprietary authentication logic.13 Popular packages such as google\_sign\_in, firebase\_auth, or flutter\_appauth allow developers to integrate federated identity providers securely.13  
Once authentication is complete, subsequent network requests must be authorized using token-based authentication (such as JSON Web Tokens).14 These tokens must be transmitted via secure HTTPS headers.13 High-value operations must also require Multi-Factor Authentication (MFA), enforcing the collection of an out-of-band One-Time Password (OTP) or physical security key.13

### **Platform Permissions and Privilege Minimization**

To minimize the application's attack surface, the development team must adhere strictly to the principle of least privilege.13 Requesting excessive permissions not only increases the risk of unauthorized data access if the device is compromised, but also degrades user trust.13  
Platform-level permission checks should be handled gracefully using the permission\_handler package.13 This ensures the application requests only the bare minimum platform permissions necessary for core operations and degrades functionality gracefully when permissions are denied 13:

Dart  
import 'package:permission\_handler/permission\_handler.dart';

Future\<void\> requestCameraPermission() async {  
  final PermissionStatus status \= await Permission.camera.request();  
    
  if (status.isGranted) {  
    // Initialize secure camera capture interface  
  } else if (status.isDenied) {  
    // Inform user of functional degradation without crashing  
  } else if (status.isPermanentlyDenied) {  
    // Guide user to the native operating system settings panel  
  }  
}

### **Cryptographic Deep Link Validation**

Mobile applications frequently rely on deep linking to navigate users directly to specific resources from external triggers like emails, web pages, or QR codes.1 However, traditional custom URI schemes (such as myapp://profile) are highly insecure.13 Any malicious application installed on the same device can register the same custom scheme.13 If the operating system routes the deep link to the malicious app instead of the legitimate one, sensitive parameters (such as OAuth authorization codes) can be hijacked.13  
To prevent deep link hijacking, developers must implement Android App Links and iOS Universal Links.13 Unlike custom schemes, App Links and Universal Links rely on standard HTTPS URLs.13 The operating system verifies ownership of the domain by performing a cryptographic handshake with the hosting web server.13  
During application installation, the OS downloads a verification file hosted at a designated, secure path on the domain (e.g., https://example.com/.well-known/assetlinks.json on Android, or https://example.com/.well-known/apple-app-site-association on iOS).13 If the cryptographic identifiers (such as the app's team ID or signing certificate hash) in the hosted file match those of the installed application, the OS registers the domain exclusively to that app.13 This ensures that external links are routed only to the verified application.13

## **Hardening Local Configurations and Secrets Management**

Hardcoding static credentials, API keys, or database passwords directly within source files is a common developmental error.3 When an application binary is unpacked, these hardcoded secrets can be easily extracted by decompilers.3

### **The Asset Leak Trap and Compile-Time Injection**

A common architectural vulnerability is the misuse of .env files.13 Many developers add their .env files directly to the assets block in the pubspec.yaml configuration 13:

YAML  
\# Insecure asset bundling pattern  
flutter:  
  assets:  
    \-.env

When an asset is declared in this manner, the Flutter build tool packages the file exactly as-is into the application's internal assets directory.13 Because Android Application Packages (APKs) and iOS App Store Packages (IPAs) are standard ZIP archives, any attacker can unzip the package, navigate to the assets folder, and read the raw, plain-text .env file containing the credentials.3  
To securely manage configuration values, developers should use compile-time injection via the \--dart-define-from-file compilation flag.13 This method passes key-value configurations directly into the compiler, injecting the values as constants without bundling physical, unencrypted files into the asset directory.13  
To implement this pattern, the configuration keys are defined in a local JSON configuration file (e.g., config.json):

JSON  
{  
  "API\_BASE\_URL": "https://api.enterprise.com",  
  "OAUTH\_CLIENT\_ID": "prod\_client\_id\_9921"  
}

This configuration file must be excluded from version control by adding it to the project's .gitignore file.15 During compilation, the JSON parameters are injected into the build using the following command 13:

Bash  
flutter build apk \--dart-define-from-file=config.json

Within the Dart code, these variables are resolved safely using environmental constants 13:

Dart  
class ConfigurationManager {  
  static const String apiBaseUrl \= String.fromEnvironment(  
    'API\_BASE\_URL',  
    defaultValue: 'https://staging.api.enterprise.com',  
  );  
}

### **Production Log Sanitization**

While logging is essential during development, leaving active debug logs in production code poses a severe security risk.13 If an application outputs sensitive data (such as API tokens, credentials, or PII) to the console, these logs are written to system-wide logs (logcat on Android, or syslog on iOS).13 These logs are easily accessible to other applications or debugging tools on the device.13  
To avoid leaking sensitive data, production logs should be sanitized.13 Developers must avoid using standard print() statements, as they output values directly to the native console in both debug and release builds.13  
Instead, developers should use the dart:developer library's log() method, or wrap logging statements inside conditional statements checking the application's build mode 13:

Dart  
import 'package:flutter/foundation.dart';  
import 'dart:developer' as developer;

void secureLog(String message) {  
  // Option A: Utilizing the dedicated developer log mechanism  
  developer.log(message, name: 'com.enterprise.app');

  // Option B: Restricting standard console output strictly to debug mode  
  if (kDebugMode) {  
    print(' Message: $message');  
  }  
}

| Configuration Control | Vulnerable Pattern | Secure Implementation Pattern | Primary Benefit |
| :---- | :---- | :---- | :---- |
| **Secrets Management** | Bundling flat .env files in pubspec.yaml assets 13 | Injecting keys as constants via \--dart-define-from-file 13 | Prevents plain-text credential harvesting from unpacked APK/IPA files 13 |
| **Console Logging** | Using raw, unconditioned print() statements in production 13 | Wrapping logs with kDebugMode or using developer.log() 13 | Prevents leakage of PII, JWTs, and keys into system-wide logs 13 |
| **Deep Link Routing** | Relying on custom URI schemes (myapp://profile) 13 | Enforcing Android App Links and iOS Universal Links 13 | Prevents deep link interception by malicious local applications 13 |

## **Secure Local Storage and Memory Management**

Securing sensitive data at rest is a critical component of mobile application hardening.14 The choice of local storage depends heavily on the volume and structure of the data.17

### **Implementation of flutter\_secure\_storage**

For small key-value pairs (such as JWTs, session tokens, or API keys), the industry-standard package is flutter\_secure\_storage.14 This package delegates data storage to the operating system's native secure vaults—Keychain Services on iOS and the Android Keystore Provider.14  
On Android, developers must explicitly configure the storage instance to use encryptedSharedPreferences: true.17 This forces the system to use the modern EncryptedSharedPreferences interface, which wraps standard SharedPreferences in hardware-backed encryption keys.17 Failing to specify this parameter can cause the application to fall back to older, legacy RSA/AES encryption methods, which can introduce performance regressions and startup delays on legacy devices 17:

Dart  
import 'package:flutter\_secure\_storage/flutter\_secure\_storage.dart';

class SecureStorageManager {  
  // Singleton pattern prevents multiple concurrent file accessor conflicts  
  static final SecureStorageManager \_instance \= SecureStorageManager.\_internal();  
  factory SecureStorageManager() \=\> \_instance;  
  SecureStorageManager.\_internal();

  static const FlutterSecureStorage \_storage \= FlutterSecureStorage(  
    aOptions: AndroidOptions(  
      encryptedSharedPreferences: true,  
    ),  
  );

  // Type-safe enum key management prevents typographical key errors  
  static Future\<void\> write(StorageKey key, String value) async {  
    await \_storage.write(key: key.name, value: value);  
  }

  static Future\<String?\> read(StorageKey key) async {  
    try {  
      return await \_storage.read(key: key.name);  
    } catch (e) {  
      // Defensively handle storage read corruption errors (e.g., key-rotation failures)  
      return null;  
    }  
  }

  static Future\<void\> delete(StorageKey key) async {  
    await \_storage.delete(key: key.name);  
  }  
}

enum StorageKey {  
  accessToken,  
  refreshToken,  
  userProfile,  
}

### **iOS Keychain Persistence and First-Run Handling**

On iOS, data written to the Keychain remains on the device even after the application is uninstalled.17 If a user uninstalls and subsequently reinstalls the application, stale credentials or authentication tokens can still be read by the newly installed app.17 This behavior can lead to inconsistent state transitions, session hijacks, or authentication bugs.17  
To mitigate this, developers should implement a first-run validation check.17 By storing an unencrypted isFirstRun flag in SharedPreferences (which *is* cleared on uninstallation), the application can detect a fresh install.17 If the flag is absent, the application executes a programmatic wipe of the iOS Keychain before updating the flag 17:

Dart  
import 'package:shared\_preferences/shared\_preferences.dart';

Future\<void\> handleIosKeychainCleanLaunch() async {  
  final SharedPreferences prefs \= await SharedPreferences.getInstance();  
  final bool isFirstRun \= prefs.getBool('is\_first\_run\_key')?? true;

  if (isFirstRun) {  
    // Programmatic wipe of the native iOS keychain  
    final SecureStorageManager storage \= SecureStorageManager();  
    await storage.\_storage.deleteAll();  
      
    // Set the first run flag to false  
    await prefs.setBool('is\_first\_run\_key', false);  
  }  
}

### **Structured Databases and Dynamic Key Wrapping**

When an application must store larger datasets, basic key-value secure storage is not performant enough.17 Instead, developers should use encrypted databases like SQLite (configured with SQLCipher) or Drift.14  
To protect these databases, developers should use a dynamic key wrapping pattern 16:

1. **Generate a cryptographically secure key**: Generate a high-entropy database encryption key using a secure random number generator.19  
2. **Store the key securely**: Save the key within flutter\_secure\_storage so it is protected by hardware-backed system vaults.16  
3. **Open the database**: Retrieve the key from secure storage at runtime and pass it as the encryption key when initializing the database.16  
4. **Wipe key bytes from memory**: Immediately clear the plaintext key bytes from Dart memory to prevent exposure.16

### **Deterministic Memory Management and Rust FFI Isolation**

Because the Dart Virtual Machine relies on non-deterministic Garbage Collection (GC), credentials and cryptographic keys can linger in heap memory for an unpredictable duration.20 This creates a vulnerability where an attacker with root access can execute a memory dump and extract the raw keys.20  
To prevent key exposure, applications can isolate cryptographic operations within a native layer using the Foreign Function Interface (FFI) and Rust.5 Using an integration library like flutter\_rust\_bridge, raw cryptographic keys are managed exclusively within native memory buffers allocated by Rust.20  
The Dart layer only receives an opaque handle or pointer to the memory.20 Once the cryptographic operation is complete, the Rust destructor immediately zeroizes the native memory buffer 20:  
![][image1]  
This mathematical guarantee ensures that the plaintext key material ![][image2] is overwritten with zeroes before the memory is released, leaving no key remnants in the garbage-collected Dart heap.20  
To avoid blocking Flutter's rendering thread (which must remain responsive to maintain performance on high-refresh-rate devices), heavy cryptographic FFI operations must be executed asynchronously using Isolate.run() 5:

Dart  
import 'dart:ffi';  
import 'dart:isolate';  
import 'package:ffi/ffi.dart';

Future\<Uint8List\> performSecureDecryption(Uint8List encryptedData, Pointer\<Void\> opaqueKeyHandle) async {  
  // Execute the synchronous native Rust function inside an isolated thread  
  return await Isolate.run(() {  
    return nativeDecryptWithOpaqueKey(encryptedData, opaqueKeyHandle);  
  });  
}

Furthermore, applications handling highly sensitive computations can leverage Fully Homomorphic Encryption (FHE) with the flutter\_concrete package.22 This package brings on-device homomorphic computations via TFHE-rs, allowing client devices to perform operations on encrypted data without ever exposing the private key to the hosting operating system or backend servers.22

### **Preventing Background Data Leakage and Input Sanitization**

To protect user privacy, applications should block screen capture and hide app screenshots in the OS task switcher.13 When an app is sent to the background, the operating system takes a screenshot to show in the multitasking view.13 For high-security applications (like banking or healthcare), this screenshot can leak PII or financial details.13  
To prevent this data leakage, developers should programmatically obscure the screen using packages like secure\_application, screen\_protector, or flutter\_screenshot\_blocker 13:

Dart  
import 'package:screen\_protector/screen\_protector.dart';

void configureScreenProtection() async {  
  // Prevent screen recording and screenshot capture  
  await ScreenProtector.preventScreenshotOn();  
    
  // Programmatically blur the application screen in the task switcher view  
  await ScreenProtector.protectScreenOn();  
}

Additionally, developers must validate and sanitize all dynamic inputs to neutralize injection attacks (such as SQL injection or XSS if rendering HTML in web views) 14:

Dart  
import 'package:ultra\_secure\_flutter\_kit/ultra\_secure\_flutter\_kit.dart';

String sanitizeUserInput(String input) {  
  final securityKit \= UltraSecureFlutterKit();  
  return securityKit.sanitizeInput(input); // Programmatically strips malicious characters  
}

## **Secure Network Layer Architecture and Interception Defenses**

Transport layer security protects data in transit and ensures the application is communicating with a trusted backend.14

### **Custom SecurityContext Certificate Pinning**

By default, mobile applications trust any valid certificate signed by a root Certificate Authority (CA) installed on the user's device.13 If an attacker compromises a trusted CA or installs a malicious certificate on a compromised device, they can intercept and decrypt HTTPS traffic.23  
To prevent this, developers must implement SSL/TLS certificate pinning, ensuring the application accepts only a predetermined, trusted certificate or public key.23 In Flutter, this can be achieved by loading a .pem certificate from the application's assets and initializing a custom SecurityContext with withTrustedRoots: false.23 This disables trust in standard system root CAs, restricting connections only to the server matching the pinned certificate 23:

Dart  
import 'dart:io';  
import 'package:dio/dio.dart';  
import 'package:flutter/services.dart' show rootBundle;

Future\<Dio\> createSecureDioInstance() async {  
  final Dio dio \= Dio();

  // Load the trusted PEM certificate from the app bundle  
  final ByteData certificateData \= await rootBundle.load('assets/certs/server\_cert.pem');

  // Create a SecurityContext and disable default/system trusted roots  
  final SecurityContext context \= SecurityContext(withTrustedRoots: false);  
  context.setTrustedCertificatesBytes(certificateData.buffer.asUint8List());

  // Instantiate an HttpClient configured with the custom SecurityContext  
  final HttpClient httpClient \= HttpClient(context: context);  
    
  // Attach the client to Dio's HTTP adapter  
  dio.httpClientAdapter \= IOHttpClientAdapter(  
    createHttpClient: () \=\> httpClient,  
  );

  return dio;  
}

### **Pinning Lifecycle and Rotation Strategies**

Implementing certificate pinning introduces operational complexities, as server-side certificates must be rotated periodically.26 If a server's certificate is rotated but the application does not have the new pin, all network connections will fail.26  
To manage certificate lifecycles without causing outages, developers can use three primary pinning strategies 26:

* **Static Pinning**: Hardcoding the certificate file or public key hash directly inside the application bundle.26 This method is highly secure but lacks operational flexibility, requiring a full app store release to update pins when certificates expire.26  
* **Backup Pinning**: Registering multiple hashes within the application, including the active leaf certificate fingerprint and pre-calculated public key hashes from backup, intermediate, or root CAs.26 This allows the server to fall back to a backup certificate without breaking older app versions.26  
* **Dynamic Pinning**: Querying a remote configuration server at runtime to fetch the active public key hashes.28 While this provides the flexibility to update pins on the fly, the first request to fetch the pins must itself be secure, requiring its endpoint to be statically pinned to prevent interception.28

### **Defending Against Kernel-Level Redirection Bypasses**

On rooted or jailbroken devices, standard user-space proxies can be bypassed.6 While standard penetration testing involves routing traffic through a user-configured Wi-Fi proxy, advanced attackers utilize kernel-level packet redirection.6 Because the Flutter engine bypasses Android system proxy settings by default, adversaries implement transport-layer interception directly within the kernel via the Network Address Translation (NAT) table of iptables.6  
An attacker with root access executes the following packet filtering command on the device to target outbound TCP traffic originating from the range of user-installed application UIDs (![][image3] to ![][image4] on Android) 6:

Bash  
iptables \-t nat \-A OUTPUT \-p tcp \-m owner \--uid-owner 10000-99999 \-j DNAT \--to-destination \<ProxyIP\>:\<ProxyPort\>

This redirection rewrites the destination IP at the kernel layer before packets leave the network interface.6 Because this occurs below the application layer, client libraries remain unaware that the network path has been altered.6  
To defend against transport-layer manipulation, developers must deploy a multi-layered security model 6:

* **Implement active environment checks**: Programmatically scan the network layer to detect anomalies, such as active network tunnels, proxy environments, or local VPN routes.6  
* **Integrate Mutual TLS (mTLS)**: Enforce mutual authentication where both the client and server must present cryptographic certificates before establishing a connection.6  
* **Enforce robust server-side security**: Ensure the backend server rejects connection requests from clients presenting outdated TLS configurations or showing signs of protocol anomalies.6

## **Dynamic Threat Protection and Anti-Reverse Engineering**

AOT compilation hardens Flutter applications, but they are not immune to reverse engineering.3 Attackers can still analyze compiled binaries or bypass security checks at runtime.3

### **Static Analysis and Assembly Modification**

Using decompilers like Ghidra, an attacker can analyze a compiled shared library (e.g., libtoolChecker.so used by flutter\_jailbreak\_detection) to find security routines like checkForRoot.7  
Once the binary offsets are mapped, the attacker can patch the native library to force the security check to always return 0 (no root detected), bypass the control entirely, and re-export the binary 7:

Code snippet  
; Decompiled checkForRoot function  
checkForRoot:  
 ...  
  BL check\_su\_binary       ; Branch to su check routine  
  CMP W0, \#1               ; If su exists, set register W0 to 1  
  B.NE safe\_exit           ; Exit safely if no root found  
  MOV W0, \#1               ; Prepare return register with 1 (Root Detected)  
  RET  
safe\_exit:  
  MOV W0, \#0               ; Prepare return register with 0 (Safe)  
  RET

; Patched assembly forcing return value 0  
checkForRoot:  
 ...  
  MOV W0, \#0               ; Directly force safe return status  
  RET                      ; Short-circuit execution and return

By pushing this modified library back onto the device, the security check is completely neutralized.7

### **Designing an Active RASP Architecture**

To counter binary patching and dynamic hooking (using tools like Frida), developers must implement an active Runtime Application Self-Protection (RASP) framework.15 Rather than performing simple, isolated boolean checks, a secure RASP architecture should establish a reactive, stream-based configuration to monitor the application's execution environment.29  
By using a state management library like Riverpod to monitor a continuous stream of events from a RASP engine (such as Talsec's freeRASP), the application can respond dynamically to threats detected at startup or during execution 15:

Dart  
import 'package:flutter\_riverpod/flutter\_riverpod.dart';  
import 'package:freerasp/freerasp.dart';

// StreamProvider exposes a continuous flow of security threats  
final securityThreatProvider \= StreamProvider\<SecurityThreat\>((ref) {  
  final TalsecConfig config \= TalsecConfig(  
    androidConfig: AndroidConfig(  
      packageName: 'com.enterprise.app',  
      signingCertHashes:, // Developers signature SHA-256  
    ),  
    iosConfig: IOSConfig(  
      bundleIds: \['com.enterprise.app'\],  
      teamId: 'ENTERPRISE123',  
    ),  
  );

  final ThreatCallback callback \= ThreatCallback(  
    onPrivilegedAccess: () \=\> ref.read(securityStateProvider.notifier).registerThreat(SecurityThreat.rootDetected),  
    onDebug: () \=\> ref.read(securityStateProvider.notifier).registerThreat(SecurityThreat.debuggerAttached),  
    onAppTamper: () \=\> ref.read(securityStateProvider.notifier).registerThreat(SecurityThreat.binaryTampered),  
    onHooks: () \=\> ref.read(securityStateProvider.notifier).registerThreat(SecurityThreat.fridaHooked),  
  );

  Talsec.instance.attachListener(callback);  
  Talsec.instance.start(config);

  ref.onDispose(() {  
    // Gracefully detach the listener during hot restart cycles  
    Talsec.instance.detachListener(callback);  
  });

  return ref.watch(securityStateProvider.notifier).threatStream;  
});

enum SecurityThreat {  
  none,  
  rootDetected,  
  debuggerAttached,  
  binaryTampered,  
  fridaHooked,  
}

By subscribing to this provider, the user interface can react instantly to security events.32 If a threat is detected, the application can invalidate local session tokens, wipe secure storage, and terminate the session.15

## **Secure Deployment, Supply Chain Management, and OTA Patch Security**

Mobile applications often pull in complex networks of third-party dependencies, exposing them to supply chain risks.1 Maintaining security throughout compilation, deployment, and ongoing patching is essential.13

### **Supply Chain Audits and Dependency Isolation**

Developers should integrate automated dependency auditing into their CI/CD pipelines.13 By regularly running flutter pub outdated, teams can scan for outdated or deprecated packages, identify transitive vulnerabilities, and ensure that only audited, actively maintained libraries are included in release builds.13  
Furthermore, developers must audit and limit the requested permissions of third-party packages to ensure dependencies cannot access platform APIs (such as location, camera, or storage) unnecessarily.14

### **Securing Over-the-Air (OTA) Code Push**

To deploy security hotfixes and UI updates without waiting for app store approval cycles, developers can use over-the-air (OTA) update frameworks like Shorebird.31 Shorebird integrates a custom Dart engine that can fetch and run compiled Dart patches dynamically.31  
To protect this update channel, Shorebird enforces several security controls 33:

* **Cryptographic Patch Signing**: When a patch is created via shorebird patch, the update is cryptographically signed using the developer's private key before being sent to the distribution servers.33  
* **Engine-Level Validation**: The client-side Shorebird engine verifies the digital signature against the developer's public key before executing the patch.33 If signature verification fails, the patch is rejected, preventing the execution of intercepted or tampered code.33  
* **Automated Rollback Controls**: The Shorebird engine monitors patch performance; if a patch fails to execute or causes the application to crash during initialization, the system automatically rolls back to the stable release binary.31

Developer Workspace            Shorebird Servers              Client Device  
  (Build Patch)               (Distribute Patch)          (Validate & Execute)  
        |                              |                            |  
  shorebird patch                      |                            |  
              |                            |  
        |------ Cryptographic \-------\> |                            |  
        |        Patch Upload          |                            |  
        |                              |------ Secure TLS \--------\> |  
        |                              |       Download             |  
        |                              |                            |    
        |                              |                            |   Signature Valid?  
        |                              |                            |     /          \\  
        |                              |                            |   Yes           No  
        |                              |                            |   /              \\  
        |                              |                            | Execute      Rollback to  
        |                              |                            | Patch       Stable Release  
        v                              v                            v      v              v

### **Build-Chain Security Tools and Integration Conflicts**

When integrating post-compilation security tools (such as GuardSquare's RASP and obfuscation tools for iOS) with Shorebird, precise build-chain orchestration is required.33  
Post-processing tools must run *during* the compilation process, not after.33 When shorebird release runs, it calls flutter build under the hood and registers a reference copy of the compiled binary.33 This reference is used to calculate the diff when generating subsequent patches.33 If a security tool modifies the binary *after* compilation, the reference copy will diverge from the distributed version, which will prevent Shorebird from generating or applying compatible patches.33  
Furthermore, some native security tools (like GuardSquare on iOS) obfuscate binaries by replacing standard function calls with dynamic, encrypted access to strings and constants in memory.33 This modification can conflict with the way Shorebird updates Dart code dynamically.33 In such scenarios, developers must coordinate with both support teams to configure custom exclusion patterns, ensuring the dynamic patch delivery channel remains secure and functional.33

## **Conclusions and Actionable Architectural Roadmap**

Securing a Flutter application requires a holistic approach that spans local storage, network communications, binary protection, and CI/CD validation.  
To build an enterprise-grade security posture, development teams should implement the following architectural roadmap:

1. **Configure strict compilation gates**: Enforce strict type checks (strict-casts, strict-inference, strict-raw-types) in analysis\_options.yaml to prevent runtime type errors, and run dependency audits in CI/CD pipelines to block vulnerable third-party packages.12  
2. **Harden configurations and local secrets**: Inject configuration variables at compile time using \--dart-define-from-file instead of bundling flat, unencrypted .env files in app assets.13 Use flags like kDebugMode to strip debug prints and prevent leaking PII into system-wide console logs.13  
3. **Isolate keys and leverage hardware-backed storage**: Save sensitive keys and tokens within flutter\_secure\_storage (with encryptedSharedPreferences: true enabled on Android), and implement a first-run validation system to manage iOS Keychain persistence across app reinstalls.17 For high-security keys, isolate cryptographic operations in native Rust memory using FFI to bypass Dart's garbage-collected heap.5  
4. **Implement robust network defenses**: Implement SSL/TLS pinning with custom SecurityContexts to reject system root CAs, and adopt public key pinning with backup pins to accommodate certificate rotations.23 Perform active environment scans to detect local network redirections or active proxies.6  
5. **Establish runtime self-protection**: Enable symbol obfuscation during compilation (--obfuscate), and integrate active RASP solutions to monitor the application's runtime environment.14 Define reactive callbacks to handle threats (like root access, attached debuggers, or memory hooking) by instantly clearing local sessions and terminating the application.29

#### **Works cited**

1. Top 10 Mobile Risks \- OWASP Mobile Top 10 2024 \- Final Release, accessed on June 2, 2026, [https://owasp.org/www-project-mobile-top-10/2023-risks/](https://owasp.org/www-project-mobile-top-10/2023-risks/)  
2. OWASP Mobile Top 10, accessed on June 2, 2026, [https://owasp.org/www-project-mobile-top-10/](https://owasp.org/www-project-mobile-top-10/)  
3. Understanding Mobile App Reverse Engineering: How Attackers Actually Break Your Apps, accessed on June 2, 2026, [https://www.iteratorshq.com/blog/understanding-mobile-app-reverse-engineering-how-attackers-actually-break-your-apps/](https://www.iteratorshq.com/blog/understanding-mobile-app-reverse-engineering-how-attackers-actually-break-your-apps/)  
4. Flutter App Security Testing: Why Most Tools Fail & What Works in ..., accessed on June 2, 2026, [https://www.appknox.com/blog/flutter-app-security-testing-why-most-tools-fail-what-works](https://www.appknox.com/blog/flutter-app-security-testing-why-most-tools-fail-what-works)  
5. Flutter Performance Part 5: Breaking Limits with Rust & FFI | CodeWithVamoSs, accessed on June 2, 2026, [https://codewithvamoss.com/en/blog/flutter-performance-part5-ffi-rust](https://codewithvamoss.com/en/blog/flutter-performance-part5-ffi-rust)  
6. Flutter SSL Pinning: How I Found a New Bypass Technique | by ..., accessed on June 2, 2026, [https://medium.com/@mr.ohmkumar1811/flutter-ssl-pinning-how-i-found-a-new-bypass-technique-48f2c5717331](https://medium.com/@mr.ohmkumar1811/flutter-ssl-pinning-how-i-found-a-new-bypass-technique-48f2c5717331)  
7. Bypass Root Detection — Flutter Jailbreak Detection | by Rayhan ..., accessed on June 2, 2026, [https://medium.com/@rayhanhanaputra/bypass-root-detection-flutter-jailbreak-detection-65f1dbb9cbf3](https://medium.com/@rayhanhanaputra/bypass-root-detection-flutter-jailbreak-detection-65f1dbb9cbf3)  
8. SAST, DAST & IAST | The 'Hows' of Applicaton Security Testing | Imperva, accessed on June 2, 2026, [https://www.imperva.com/learn/application-security/sast-iast-dast/](https://www.imperva.com/learn/application-security/sast-iast-dast/)  
9. Mastering Flutter Secure Storage: Code Optimization and Security Tips | by Kalpesh Shinde, accessed on June 2, 2026, [https://shindekalpesharun.medium.com/mastering-flutter-secure-storage-code-optimization-and-security-tips-7a79b380a124](https://shindekalpesharun.medium.com/mastering-flutter-secure-storage-code-optimization-and-security-tips-7a79b380a124)  
10. SAST vs DAST: What they are and when to use them \- CircleCI, accessed on June 2, 2026, [https://circleci.com/blog/sast-vs-dast-when-to-use-them/](https://circleci.com/blog/sast-vs-dast-when-to-use-them/)  
11. What Is Dynamic Application Security Testing (DAST) ? DAST vs SAST Explained \- Fortinet, accessed on June 2, 2026, [https://www.fortinet.com/resources/cyberglossary/dynamic-application-security-testing](https://www.fortinet.com/resources/cyberglossary/dynamic-application-security-testing)  
12. Getting Started with Flutter Lint and Static Analysis | DCM \- Code ..., accessed on June 2, 2026, [https://dcm.dev/blog/2025/10/21/getting-started-flutter-static-analytics-lints/](https://dcm.dev/blog/2025/10/21/getting-started-flutter-static-analytics-lints/)  
13. Ultimate Guide: How to Secure a Flutter App (OWASP Mobile Top 10 ..., accessed on June 2, 2026, [https://dev.to/kodekhalifah/ultimate-guide-how-to-secure-a-flutter-app-owasp-mobile-top-10-checklist-5d6i](https://dev.to/kodekhalifah/ultimate-guide-how-to-secure-a-flutter-app-owasp-mobile-top-10-checklist-5d6i)  
14. Flutter Security: Top Best Practices \- DEV Community, accessed on June 2, 2026, [https://dev.to/sushan\_dristi\_ab98c07ea8f/flutter-security-top-best-practices-aoe](https://dev.to/sushan_dristi_ab98c07ea8f/flutter-security-top-best-practices-aoe)  
15. OWASP Top 10 For Flutter \- M1: Mastering Credential Security in Flutter | AppSec Articles, accessed on June 2, 2026, [https://docs.talsec.app/appsec-articles/articles/owasp-top-10-for-flutter-m1-mastering-credential-security-in-flutter](https://docs.talsec.app/appsec-articles/articles/owasp-top-10-for-flutter-m1-mastering-credential-security-in-flutter)  
16. Building Secure Mobile App with Flutter | by Tomáš Repčík \- ITNEXT, accessed on June 2, 2026, [https://itnext.io/building-secure-mobile-app-with-flutter-863c7d894f1d](https://itnext.io/building-secure-mobile-app-with-flutter-863c7d894f1d)  
17. What Is Secure Storage in Flutter: Best Practices, Common Mistakes ..., accessed on June 2, 2026, [https://leancode.co/glossary/secure-storage-in-flutter](https://leancode.co/glossary/secure-storage-in-flutter)  
18. Storing Data in Secure Storage in Flutter | Blog \- Digital.ai, accessed on June 2, 2026, [https://digital.ai/catalyst-blog/flutter-secure-storage/](https://digital.ai/catalyst-blog/flutter-secure-storage/)  
19. ultra\_secure\_flutter\_kit | Flutter package \- Pub.dev, accessed on June 2, 2026, [https://pub.dev/packages/ultra\_secure\_flutter\_kit](https://pub.dev/packages/ultra_secure_flutter_kit)  
20. M-Security: high-performance Flutter security SDK powered entirely by Rust (no platform channels, no Dart crypto) : r/FlutterDev \- Reddit, accessed on June 2, 2026, [https://www.reddit.com/r/FlutterDev/comments/1s4fe5n/msecurity\_highperformance\_flutter\_security\_sdk/](https://www.reddit.com/r/FlutterDev/comments/1s4fe5n/msecurity_highperformance_flutter_security_sdk/)  
21. Built a Rust crypto \+ encrypted VFS SDK for Flutter apps, looking for Rust feedback \- Reddit, accessed on June 2, 2026, [https://www.reddit.com/r/rust/comments/1rymyju/built\_a\_rust\_crypto\_encrypted\_vfs\_sdk\_for\_flutter/](https://www.reddit.com/r/rust/comments/1rymyju/built_a_rust_crypto_encrypted_vfs_sdk_for_flutter/)  
22. flutter\_concrete | Flutter package \- Pub.dev, accessed on June 2, 2026, [https://pub.dev/packages/flutter\_concrete](https://pub.dev/packages/flutter_concrete)  
23. How to Implement SSL Pinning in a Flutter App Using Dio — A ..., accessed on June 2, 2026, [https://medium.com/@swatantra109/how-to-implement-ssl-pinning-in-a-flutter-app-using-dio-a-complete-guide-16d4cb13c88a](https://medium.com/@swatantra109/how-to-implement-ssl-pinning-in-a-flutter-app-using-dio-a-complete-guide-16d4cb13c88a)  
24. Flutter App Security : Mastering SSL Certificate Pinning for App Protection \- Web and Mobile App Development Company \- NGD Technolab, accessed on June 2, 2026, [https://ngendevtech.com/blogs/flutter-app-security-mastering-ssl-certificate-pinning-for-app-protection/](https://ngendevtech.com/blogs/flutter-app-security-mastering-ssl-certificate-pinning-for-app-protection/)  
25. Best practice to SSL Pinning in Flutter, fetch the certificate every time? or storing it in assets?, accessed on June 2, 2026, [https://stackoverflow.com/questions/75131536/best-practice-to-ssl-pinning-in-flutter-fetch-the-certificate-every-time-or-st](https://stackoverflow.com/questions/75131536/best-practice-to-ssl-pinning-in-flutter-fetch-the-certificate-every-time-or-st)  
26. SSL Pinning in Flutter: Step-by-Step Implementation Guide (2026) \- Droids On Roids, accessed on June 2, 2026, [https://www.thedroidsonroids.com/blog/ssl-certificate-pinning-in-flutter](https://www.thedroidsonroids.com/blog/ssl-certificate-pinning-in-flutter)  
27. SSL Pinning Guide \- What Is SSL Pinning & How It Works \- DoveRunner, accessed on June 2, 2026, [https://doverunner.com/blogs/importance-of-ssl-pinning/](https://doverunner.com/blogs/importance-of-ssl-pinning/)  
28. How to Implement SSL Pinning in Flutter for Better App Security \- Logique Digital Indonesia, accessed on June 2, 2026, [https://www.logique.co.id/blog/en/2025/06/18/implement-ssl-pinning-dev/](https://www.logique.co.id/blog/en/2025/06/18/implement-ssl-pinning-dev/)  
29. How to Detect Root on Flutter | AppSec Articles \- Docs Portal, accessed on June 2, 2026, [https://docs.talsec.app/appsec-articles/articles/how-to-detect-root-on-flutter](https://docs.talsec.app/appsec-articles/articles/how-to-detect-root-on-flutter)  
30. SSL pinning disable flutter 3.35.1 \- General, accessed on June 2, 2026, [https://forum.itsallwidgets.com/t/ssl-pinning-disable-flutter-3-35-1/3939](https://forum.itsallwidgets.com/t/ssl-pinning-disable-flutter-3-35-1/3939)  
31. How to Push Over-the-Air (OTA) Flutter Updates with Shorebird \- Complete 2026 Guide, accessed on June 2, 2026, [https://dev.to/techwithsam/how-to-push-over-the-air-ota-flutter-updates-with-shorebird-complete-2026-guide-4d35](https://dev.to/techwithsam/how-to-push-over-the-air-ota-flutter-updates-with-shorebird-complete-2026-guide-4d35)  
32. Flutter: Designing Root/Jailbreak Detection and Handling with freerasp and Riverpod \- Zenn, accessed on June 2, 2026, [https://zenn.dev/harx/articles/27219eba6dd46b?locale=en](https://zenn.dev/harx/articles/27219eba6dd46b?locale=en)  
33. Obfuscation and Security Tooling | Shorebird, accessed on June 2, 2026, [https://docs.shorebird.dev/code-push/guides/security-tools/](https://docs.shorebird.dev/code-push/guides/security-tools/)  
34. Shorebird Library: Supercharge App Distribution with Flutter \- Logique Digital Indonesia, accessed on June 2, 2026, [https://www.logique.co.id/blog/en/2025/01/20/shorebird-library/](https://www.logique.co.id/blog/en/2025/01/20/shorebird-library/)  
35. Security \- Shorebird docs, accessed on June 2, 2026, [https://docs.shorebird.dev/system/security/](https://docs.shorebird.dev/system/security/)  
36. Shorebird (Flutter Code Push) — is anyone actually using this in production? \- Reddit, accessed on June 2, 2026, [https://www.reddit.com/r/FlutterDev/comments/1rmky40/shorebird\_flutter\_code\_push\_is\_anyone\_actually/](https://www.reddit.com/r/FlutterDev/comments/1rmky40/shorebird_flutter_code_push_is_anyone_actually/)

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABYAAAAaCAYAAACzdqxAAAABTUlEQVR4Xu2TvytFYRyHv0IpJBkMSNlkMFixsFJ0lfI/GCzKzKBkkMlmkMFisaDcUmaLMmIxsZnkx/Ppe97u+957rk53MJ2nnrrn/bzvue/5nPeYlfw3s3iMj/iCS2ncwI75PHmPh2lcYxQruI8/5n/yF+/m8+5wFefSuBHt5ByvsbsuCwzjjfmN1+qyXLSgipP4imNJ6szgNn7iLfamcT7zeIr9+IFTaWx9eGS+Ae32II2bs4Wb2W8tXIgysY4r2GGeL6dxPl14htPZtRZu1GKbMH/znTiEbzge5U0JNWg3Qjc+wbbsWrsN6KlaqkGo4yr2mHeqfoWe7MJarEHoVDzjCO5F46rhyQrWEI6ZFgUezHtUz7vRuCr7tgLHTC9DX88lDkbj+vK+cDEaG8Ar8/7bo/EG9DjalSYGw0tR3zqz+mMRzwnGp6akpKQIv4PRRHq+fjjcAAAAAElFTkSuQmCC>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADMAAAAZCAYAAACclhZ6AAACaUlEQVR4Xu2WT6hNURTGvxfqCQOeSEovSZF6ShJDSQwoSpSBGRMZkvHLwFRG/iQDKZkoBsrgRhkwpTdAXQOUklIMKHxfa6971973nH2LXknnV1/n7j9rf2vtfdrnAh0dHX/LBLWh7AxofBW1k1pQjEUmqS3U0nKgYBe1jlpUDiSi3xrUPQcoaDN1j+rnQwOmqafUC+oK9YraGifAzM5Qn6ib6XkRo8muoO5Q16n7sHn7sxlG9HsC86wiI+3gNuor9TYfHvCT2hPam2BJXIZtxmpqjnoAOxnnEvUZdlJiH2ytU4MZwErqJbU49J1G7ifk6X5VasWoWCWuxRwdu+YqCSVzkPoFSz5yLvUreSVxA6Mbo/V7GK5fth15ul+VWjHrYf1azHFDxSj2AixpJR85kPr12i2nnmMYE9G45gr5fUDuJ+TZFDtCrRiNtRWjRJWEkqkV08PwNJsSUrzHei5NxbhflVoxCh5XjP+uFeMe44rxmHkpRjfNuGIepd+1YmaoLxhfjPzmrZj/6jVrugCWwe7+b9R2tF8Ah1P/LbRfABNpXHNF2wUgT/erUitGSej10Gvi+C73qbXUcVjS18Ic4Vezngupu6kdP5J+yl6gFx39hDz7ML9WtDP6mP2gPqZ2yXvYN8K/5udhxe9IbcXMFn3T1BvkcVPUM+pxaouj1PfQFvpbFeO0vjx97UZUrXaqVA/5f6vdsEJvU8dgx30EeeFLYAm8o05Sr6mHsL8vEX0M52CnqddTa53NZhjR7ypsXtNG/xFKdi91Iv1uQmYbYYnq2WauHdecQ7DTaiL66dnm2dHR0dHx7/EbYFi4F0XlaloAAAAASUVORK5CYII=>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADMAAAAZCAYAAACclhZ6AAAC9UlEQVR4Xu2WS6hOURTH/zcUeT/yiPIIZaSIUmKCGFCYKMXQRMoAJYNvYmhiKAaUyGOgKMXgRlEMjFAehZQMUIqSwvpZZ33f+vZ37rkDI3X+9e/cvc7+9m+ddfba50qtWrX6F00zrzLPL28kDZlny+eNKe5ljTcvM08qbxQK3rjyRqXM49rE7OqK+Yv5vPmJeWP/7b9aZH5gfiuf98i8PE+Qww6Zv5tvmL+aj2gw2RlyZvBeqp6Zea/kzEZ1zMeL2CfzJfPYasz1t3lXd4a0yfxL/nu0zvzDfN88uYrxhm6Z35sXV7GOfK3MXCFnZh78zEMwO0Wsq+nmx+a9RZxqvFFvyy0x/zSvjwmm1eZv5mH5djopT/Kaekkhqkp8h3o8xpk5T87MvA/q5yGYwxph+8Yi24s4MapOJVAkzjUUMSpKZSNprlkRP60ej3FmRryJF/HgDajpYfK2qls8YvTHGo3+MBc1+sM08SIevFod02DP8FYAcg/V9czBKhaJRQK5Z+aYn1dzhqsYa5Y9s1XOzLyzGuyZYJbF72qq+YV5YRp/lP/oQEySz7lb3Ue35cnHdhwyH5X31u5qfNj8VL4WBwHi96zDejG+LGdmHvlkHmOYefvXipOD1/dODn8tT2BbmrPIfE9+3DKPBuZvvLKaw9FM9WIO1b8gX4tKhziaYQaPtWBmHsq8h+oxgzeiOEZnyivK/h2p0ThJ+MDGtiKZWX0z/KHi2xI9k6seCl70TBMPwazjdTXBvLSI8ZbydwZRNaAh9jNJdqoxSVExjtP4UtM79FD+zsDbrH4mDQ0z86aon4dgdopYn5hwXQ5Ba+V7n+SySPxUivNW6I3QAnnSfB+ocPRG3vcoigATwfssZ2axRTMPFsxGAaJ6Z8xX5RUq//1Az8w35fO47lP/A8cB8MZ8Tr5teJC5aQ6Cd0fODN4JDTJp+MxjHsxRtUHeXDvl+7hOE81bzPura514IP5f21Ndy7cbInGYTTyUefBbtWrVqtX/oz8vhMgPcGyM6AAAAABJRU5ErkJggg==>