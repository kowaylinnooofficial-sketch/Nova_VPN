# NOVA VPN V2

Premium Material 3 Android VPN client project with two separate tunnel engines:

- WireGuard: official `com.wireguard.android:tunnel` embeddable library; import a provider `.conf` file.
- VMess/Xray: XTLS/libXray Android AAR built from source in CI; import legacy `vmess://BASE64(JSON)` links or Xray JSON configuration files. HEV SOCKS5 TUN is built from source and connects Android's TUN descriptor to Xray's local SOCKS inbound.

## Build requirements
Android Studio, JDK 17, Android SDK 35, Gradle 8.13. CI builds native Xray and HEV dependencies automatically. WireGuard runs in its own Android process, separate from libXray, because the two native Go runtimes must not be loaded into the same process.

## Build locally
Native prerequisites are Go, Python 3, Android NDK, and the HEV submodule. Run:

```bash
git submodule update --init --recursive
export NDK_HOME="$ANDROID_NDK_HOME"
git clone --depth 1 https://github.com/XTLS/libXray.git native/libXray
python3 native/libXray/build/main.py android
mkdir -p app/libs
cp native/libXray/libXray.aar app/libs/
./scripts/build-hev.sh
# Build with Android Studio or Gradle 8.13:
gradle :app:assembleDebug
```

APK: `app/build/outputs/apk/debug/app-debug.apk`.

## GitHub Actions
Push to `main` or `master`, or run **Actions → Android APK**. Download the `NovaVPN-debug-apk` artifact from the successful run. CI builds libXray and HEV native libraries for arm64-v8a, armeabi-v7a, x86 and x86_64 before Gradle assembles the APK.

## Usage
1. Install APK and open NOVA VPN.
2. Choose WireGuard or VMess.
3. WireGuard: import a `.conf` file. VMess: paste a `vmess://` link or import a full Xray JSON file.
4. Tap Connect and approve Android's VPN permission.
5. Only one VPN mode can be active at a time.

## Important limitations and security
- This app does not include VPN servers, free keys, or subscriptions. Supply a valid configuration from a trusted provider or your own server.
- VMess legacy QR links (`vmess://` followed by Base64-encoded JSON) and Xray JSON with VMess outbounds are supported by the Kotlin converter. Other share-link protocols are not claimed as supported.
- Configs contain credentials/private keys. They are kept in app-private storage/preferences and must never be committed to GitHub. Do not share your APK with private server keys embedded in it.
- Android's VPN permission is required. A different always-on VPN or another active VPN app may prevent connection.
- Debug APK is for testing. A signed release build needs a signing key configured by the owner.
- Source generation is complete, but this environment cannot run an Android SDK/NDK build or a device-level VPN test. Treat the first GitHub Actions run as the compile validation; test both protocols on a physical device before relying on it.

## Native components and licenses
- WireGuard Android tunnel library: Apache-2.0.
- XTLS/libXray: MIT wrapper; Xray-core: MPL-2.0. The CI build fetches the upstream source at build time.
- HEV SOCKS5 tunnel: upstream project license applies. The source is fetched as a git submodule; include its license when redistributing.
