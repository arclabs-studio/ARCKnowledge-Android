# Android Security Reference

**Comprehensive security guidance for ARC Labs Android projects.** Covers OWASP Mobile Top 10 with Kotlin 2.0 + Compose + Hilt + Room examples. Companion to `Quality/code-review.md` domain 8.

> For automated scanning, use `/arc-android-security-audit` skill or `arc-kotlin-security-auditor` agent.

---

## Security Architecture Principles

1. **Defense in depth** — no single security control; layer checks at network, storage, and UI layers
2. **Fail secure** — deny by default; only allow explicitly permitted operations
3. **Least privilege** — request only permissions required; never over-request
4. **Sensitive data minimization** — don't store what you don't need; purge when done
5. **Trust no input** — validate all data from Intents, deep links, ContentProviders, and APIs

---

## OWASP Mobile Top 10

### M1 — Improper Platform Usage

Misuse of Android platform features: exported components, Intents, ContentProviders, PendingIntents.

#### Exported Components

Every `Activity`, `Service`, `BroadcastReceiver`, and `ContentProvider` with `exported="true"` is accessible to any installed app unless protected by a permission.

```xml
<!-- ❌ Exported without permission — any app can launch -->
<activity android:name=".DeepLinkActivity" android:exported="true" />

<!-- ✅ Protected by signature-level permission -->
<activity
    android:name=".DeepLinkActivity"
    android:exported="true"
    android:permission="com.myapp.permission.OPEN_LINK" />
```

#### PendingIntent

```kotlin
// ❌ Missing FLAG_IMMUTABLE — pending intent hijacking possible on API 31+
val pi = PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT)

// ✅
val pi = PendingIntent.getActivity(
    context,
    0,
    intent,
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
)
```

#### Deep Link Validation

```kotlin
// ❌ Trusting deep link parameters without validation
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val url = intent?.data?.getQueryParameter("redirect") ?: return
    openWebView(url)  // Open redirect / SSRF
}

// ✅ Allowlist validation
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val url = intent?.data?.getQueryParameter("redirect") ?: return
    val allowedHosts = setOf("myapp.com", "www.myapp.com")
    val uri = Uri.parse(url)
    require(uri.scheme == "https" && uri.host in allowedHosts) { "Rejected URL: $url" }
    openWebView(url)
}
```

---

### M2 — Insecure Data Storage

#### DataStore (Recommended Pattern)

```kotlin
// ✅ DataStore with EncryptedFile for sensitive preferences
@Singleton
class SecurePreferencesDataSource @Inject constructor(
    @ApplicationContext private val context: Context,
) {
    private val masterKey = MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build()

    private val encryptedPrefs = EncryptedSharedPreferences.create(
        context,
        "secure_prefs",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )

    fun saveToken(token: String) {
        encryptedPrefs.edit().putString("auth_token", token).apply()
    }

    fun getToken(): String? = encryptedPrefs.getString("auth_token", null)

    fun clearToken() {
        encryptedPrefs.edit().remove("auth_token").apply()
    }
}
```

#### Room (Sensitive Entity Pattern)

```kotlin
// ✅ Store only necessary fields; no cleartext credentials in entities
@Entity(tableName = "user_sessions")
data class UserSessionEntity(
    @PrimaryKey val sessionId: String,
    val userId: String,
    val expiresAt: Long,
    // ❌ NEVER store raw tokens in Room entities — use Keystore-backed storage instead
    // val token: String  // Don't do this
)
```

#### File Storage

```kotlin
// ❌ External storage — accessible to other apps
val file = File(Environment.getExternalStorageDirectory(), "user_data.json")

// ✅ Internal storage with file-based encryption
val file = File(context.filesDir, "user_data.enc")
// Or use FileProtectionLevel via Android Keystore:
val outputStream = context.openFileOutput("secure_data", Context.MODE_PRIVATE)
```

---

### M3 — Insecure Communication

#### Network Security Config

```xml
<!-- res/xml/network_security_config.xml -->
<!-- ✅ Production: no cleartext, no user CAs -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>

    <!-- Development override only — NEVER in production manifest -->
    <debug-overrides>
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

#### Certificate Pinning

```kotlin
// ✅ OkHttp certificate pinning for critical APIs
@Provides
@Singleton
fun provideOkHttpClient(): OkHttpClient {
    val certificatePinner = CertificatePinner.Builder()
        // Always pin at least 2 keys: primary + backup
        .add("api.myapp.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
        .add("api.myapp.com", "sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=")
        .build()

    return OkHttpClient.Builder()
        .certificatePinner(certificatePinner)
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .build()
}
```

> **Note:** Certificate pinning can cause outages on certificate rotation. Always include a backup pin and have a server-side kill switch.

---

### M4 — Insecure Authentication

#### Biometric Authentication

```kotlin
// ✅ Biometric with Keystore-backed CryptoObject (cannot be bypassed on rooted devices)
class BiometricAuthManager @Inject constructor(
    @ApplicationContext private val context: Context,
) {
    private val keyStore = KeyStore.getInstance("AndroidKeyStore").apply { load(null) }
    private val keyAlias = "biometric_auth_key"

    fun setupKey() {
        if (keyStore.containsAlias(keyAlias)) return
        val keyGenerator = KeyGenerator.getInstance(
            KeyProperties.KEY_ALGORITHM_AES,
            "AndroidKeyStore"
        )
        keyGenerator.init(
            KeyGenParameterSpec.Builder(
                keyAlias,
                KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
            )
                .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
                .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
                .setUserAuthenticationRequired(true)
                .setUserAuthenticationParameters(0, KeyProperties.AUTH_BIOMETRIC_STRONG)
                .build()
        )
        keyGenerator.generateKey()
    }

    fun buildCryptoObject(): BiometricPrompt.CryptoObject {
        val secretKey = keyStore.getKey(keyAlias, null) as SecretKey
        val cipher = Cipher.getInstance("AES/GCM/NoPadding")
        cipher.init(Cipher.ENCRYPT_MODE, secretKey)
        return BiometricPrompt.CryptoObject(cipher)
    }
}
```

#### Session Management

```kotlin
// ✅ Session expiry enforced at every navigation
@HiltViewModel
class SessionViewModel @Inject constructor(
    private val sessionRepository: SessionRepository,
) : ViewModel() {

    val isSessionValid: StateFlow<Boolean> = sessionRepository
        .observeSession()
        .map { session -> session?.expiresAt?.let { it > System.currentTimeMillis() } ?: false }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), false)
}
```

---

### M5 — Insufficient Cryptography

#### Android Keystore for Key Generation

```kotlin
// ✅ AES-256-GCM via Android Keystore — hardware-backed on supported devices
object KeystoreUtils {
    private const val KEY_ALIAS = "app_encryption_key"

    fun getOrCreateKey(): SecretKey {
        val keyStore = KeyStore.getInstance("AndroidKeyStore").apply { load(null) }
        if (keyStore.containsAlias(KEY_ALIAS)) {
            return keyStore.getKey(KEY_ALIAS, null) as SecretKey
        }

        return KeyGenerator.getInstance(
            KeyProperties.KEY_ALGORITHM_AES,
            "AndroidKeyStore"
        ).apply {
            init(
                KeyGenParameterSpec.Builder(
                    KEY_ALIAS,
                    KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
                )
                    .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
                    .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
                    .setKeySize(256)
                    .build()
            )
        }.generateKey()
    }

    fun encrypt(data: ByteArray): Pair<ByteArray, ByteArray> {
        val cipher = Cipher.getInstance("AES/GCM/NoPadding")
        cipher.init(Cipher.ENCRYPT_MODE, getOrCreateKey())
        return cipher.doFinal(data) to cipher.iv  // ciphertext to IV
    }

    fun decrypt(data: ByteArray, iv: ByteArray): ByteArray {
        val cipher = Cipher.getInstance("AES/GCM/NoPadding")
        val spec = GCMParameterSpec(128, iv)
        cipher.init(Cipher.DECRYPT_MODE, getOrCreateKey(), spec)
        return cipher.doFinal(data)
    }
}
```

#### Prohibited Algorithms

| ❌ Prohibited | ✅ Replacement |
|-------------|--------------|
| `DES`, `3DES` | `AES-256-GCM` |
| `RC4` | `AES-256-GCM` |
| `MD5` (for security) | `SHA-256` minimum |
| `SHA-1` (for security) | `SHA-256` minimum |
| `AES/ECB` mode | `AES/GCM/NoPadding` |
| `java.util.Random` | `SecureRandom` |

---

### M1 — WebView Security

```kotlin
// ✅ Secure WebView configuration
@Composable
fun SecureWebView(
    url: String,
    allowedHosts: Set<String>,
    modifier: Modifier = Modifier,
) {
    AndroidView(
        factory = { context ->
            WebView(context).apply {
                settings.apply {
                    javaScriptEnabled = true  // Only if truly required
                    allowFileAccess = false
                    allowContentAccess = false
                    allowUniversalAccessFromFileURLs = false
                    allowFileAccessFromFileURLs = false
                    setSupportZoom(false)
                    saveFormData = false
                }
                webViewClient = object : WebViewClient() {
                    override fun shouldOverrideUrlLoading(
                        view: WebView,
                        request: WebResourceRequest,
                    ): Boolean {
                        val host = request.url.host ?: return true
                        return host !in allowedHosts  // Block non-allowlisted hosts
                    }
                }
            }
        },
        update = { webView ->
            val parsedUri = Uri.parse(url)
            require(parsedUri.scheme == "https" && parsedUri.host in allowedHosts)
            webView.loadUrl(url)
        },
        modifier = modifier,
    )
}
```

---

### M9 — Reverse Engineering / Logging

#### Timber Configuration

```kotlin
// ✅ ARC Labs Timber setup — release tree strips all logs
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        if (BuildConfig.DEBUG) {
            Timber.plant(Timber.DebugTree())
        } else {
            Timber.plant(CrashReportingTree())
        }
    }
}

private class CrashReportingTree : Timber.Tree() {
    override fun log(priority: Int, tag: String?, message: String, t: Throwable?) {
        if (priority < Log.WARN) return  // Only WARN and above go to crash reporter
        // Ensure no sensitive data in message before reporting
        FirebaseCrashlytics.getInstance().log("$tag: $message")
        if (t != null) FirebaseCrashlytics.getInstance().recordException(t)
    }
}
```

#### ProGuard/R8 for Release

```proguard
# Release build — ensure minifyEnabled = true in build.gradle.kts

# Keep crash reporting symbols
-keepattributes SourceFile,LineNumberTable
-keep public class * extends java.lang.Exception

# Rename source file attribute for obfuscation
-renamesourcefileattribute SourceFile
```

---

### M8 — Code Tampering Protections

#### Play Integrity API

```kotlin
// ✅ Server-side integrity check for high-value operations
@HiltViewModel
class CheckoutViewModel @Inject constructor(
    private val integrityManager: IntegrityManager,
    private val paymentRepository: PaymentRepository,
) : ViewModel() {

    fun initiatePayment(amount: BigDecimal) {
        viewModelScope.launch {
            try {
                // Request integrity token
                val nonce = generateNonce()  // Server-generated nonce
                val tokenRequest = StandardIntegrityManager.StandardIntegrityTokenRequest
                    .builder()
                    .setRequestHash(nonce)
                    .build()

                val tokenResponse = integrityManager
                    .requestAndPrepareIntegrityToken(
                        PrepareIntegrityTokenRequest.builder()
                            .setCloudProjectNumber(CLOUD_PROJECT_NUMBER)
                            .build()
                    ).await()
                    .request(tokenRequest)
                    .await()

                // Send token to backend for server-side validation
                paymentRepository.initiatePayment(
                    amount = amount,
                    integrityToken = tokenResponse.token(),
                )
            } catch (e: IntegrityServiceException) {
                Timber.e(e, "Integrity check failed")
                // Handle gracefully — don't block legitimate users unnecessarily
            }
        }
    }
}
```

#### Release Build Configuration

```kotlin
// build.gradle.kts
android {
    buildTypes {
        release {
            isMinifyEnabled = true          // Enable R8
            isShrinkResources = true        // Remove unused resources
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            // Ensure debuggable is false (default, but be explicit)
            isDebuggable = false
        }
    }
}
```

---

### M2 — Sensitive Data in UI

```kotlin
// ✅ FLAG_SECURE for sensitive screens (payment, credentials)
@Composable
fun SecureScreen(content: @Composable () -> Unit) {
    val activity = LocalContext.current as Activity

    DisposableEffect(Unit) {
        activity.window.setFlags(
            WindowManager.LayoutParams.FLAG_SECURE,
            WindowManager.LayoutParams.FLAG_SECURE
        )
        onDispose {
            activity.window.clearFlags(WindowManager.LayoutParams.FLAG_SECURE)
        }
    }

    content()
}

// ✅ Password fields — prevent autofill leakage
@Composable
fun PasswordField(
    value: String,
    onValueChange: (String) -> Unit,
) {
    OutlinedTextField(
        value = value,
        onValueChange = onValueChange,
        visualTransformation = PasswordVisualTransformation(),
        keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Password),
        modifier = Modifier.semantics {
            // Exclude from autofill and accessibility tree
            contentDescription = "Password field"
        },
    )
}
```

---

### M10 — Supply Chain / Third-Party SDKs

#### Dependency Audit

```bash
# Check for known vulnerabilities in dependencies
./gradlew dependencyUpdates

# List all transitive dependencies
./gradlew dependencies --configuration releaseRuntimeClasspath

# Check for duplicate dependencies
./gradlew app:dependencies | grep "(*)"
```

#### Third-Party SDK Trust Evaluation

Before adding a new SDK, verify:
- [ ] SDK requires only permissions your app already has
- [ ] SDK does not access storage, camera, microphone without your code path
- [ ] Privacy policy covers data collection by the SDK
- [ ] SDK is not on Google Play's restricted SDK list
- [ ] SDK source is available or binary has been security reviewed

---

## Sensitive Data Classification

| Classification | Examples | Storage | Logging |
|----------------|---------|---------|---------|
| **Critical** | Passwords, PINs, biometric data, encryption keys | Android Keystore only | Never |
| **Sensitive** | Auth tokens, session IDs, payment card tokens | `EncryptedSharedPreferences` | Never |
| **Personal** | Name, email, phone, address | Encrypted Room / DataStore | Never full value |
| **Internal** | User preferences, app state | `DataStore` / Room | Identifiers only |
| **Public** | App version, theme, language | `DataStore` / Room | OK |

---

## Security Checklist (Pre-Release)

### Manifest
- [ ] `android:debuggable` NOT set to `true` in release
- [ ] `android:allowBackup="false"` or `android:dataExtractionRules` configured
- [ ] All exported components have explicit `android:permission` or are unexported
- [ ] Network security config referenced and enforces HTTPS

### Storage
- [ ] No plaintext sensitive data in SharedPreferences
- [ ] No PII in Room entities (use encrypted fields or separate secure storage)
- [ ] No secrets in `res/values/strings.xml` or bundled assets
- [ ] `EncryptedSharedPreferences` or Keystore used for tokens

### Network
- [ ] HTTPS enforced for all production endpoints
- [ ] Certificate pinning on financial/health/authentication APIs
- [ ] No `checkServerTrusted` override accepting all certificates
- [ ] `network_security_config.xml` has `cleartextTrafficPermitted="false"`

### Cryptography
- [ ] `AES/GCM/NoPadding` (no ECB, no DES, no RC4)
- [ ] Encryption keys in Android Keystore (not hardcoded)
- [ ] `SecureRandom` (not `java.util.Random`) for security operations
- [ ] `SHA-256` minimum for hashing

### Code
- [ ] No sensitive data in `Log.d()` / `Log.v()` (use Timber with release no-op tree)
- [ ] `minifyEnabled = true` in release buildType
- [ ] No debug endpoints or backdoor code in production build
- [ ] `FLAG_SECURE` on payment/credential screens

### Dependencies
- [ ] All third-party SDKs reviewed for permission and data collection
- [ ] No dependencies with known critical CVEs
- [ ] `libs.versions.toml` pinned to specific versions (no `latest.release`)

---

## References

- [Android Security Overview](https://developer.android.com/privacy-and-security/security-overview)
- [OWASP Mobile Top 10 (2024)](https://owasp.org/www-project-mobile-top-10/)
- [Android Keystore System](https://developer.android.com/privacy-and-security/keystore)
- [Network Security Configuration](https://developer.android.com/privacy-and-security/network-security-config)
- [EncryptedSharedPreferences](https://developer.android.com/reference/androidx/security/crypto/EncryptedSharedPreferences)
- [Play Integrity API](https://developer.android.com/google/play/integrity)
- `Quality/code-review.md` — Security domain (domain 8, line 755)
- `.claude/skills/arc-android-security-audit/SKILL.md` — Automated scanning skill
