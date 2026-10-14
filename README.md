# Torch Lock

Android app that switches the flashlight off whenever something turns it on.

Built because the Amazon Flex app lights the torch for barcode scanning and does not reliably put
it out again.

**Download:** [`bin/TorchLock-1.0.0.apk`](bin/TorchLock-1.0.0.apk) · Android 8.0 or newer

---

## What it does, and the honest limit

Android provides **no way to prevent** the torch being switched on. There is no veto to register.
What it provides is a callback when torch state changes, and a call to switch it off.

So this is a watchdog, not a lock:

- **An app leaves the torch on after using the camera** → switched off within milliseconds.
  Expect a brief flicker, not darkness.
- **An app is still holding the camera** → that app owns the torch and nothing on the device can
  take it. Torch Lock shows `WAIT` and retries every 0.8 s, so the torch goes out the instant that
  app lets go.

Anyone claiming a true hardware lockout without root is mistaken. The app says which of the two
situations you are in rather than claiming success while your flashlight is visibly lit.

## Using it

Open it once and turn the switch on. It then survives reboots and restarts itself if Android kills
it for memory. A silent, low-priority notification stays in the shade — that is what keeps the
process alive, and it carries a one-tap **Stop**.

## It does not track you — and here is how to check

The claim is not "we promise". It is a property of the build, and every line below is verifiable
from the APK itself with the Android SDK tools.

### Permissions — the complete list

```
FOREGROUND_SERVICE                 keeps the guard running when the app is not on screen
FOREGROUND_SERVICE_SPECIAL_USE     Android 14+ requires a type for the above
POST_NOTIFICATIONS                 the ongoing notification the platform requires
RECEIVE_BOOT_COMPLETED             re-arms after a restart
```

That is all of them.

### What is deliberately absent

| | |
|---|---|
| **`INTERNET`** | **not declared** — Android blocks every outbound connection. The app *cannot* send data anywhere, whatever its code says. |
| `CAMERA` | not declared. Switching the torch needs no camera permission; that permission guards reading image data, which this never does. |
| Analytics / ads | no Firebase, Crashlytics, AdMob, Segment, AppsFlyer or any HTTP library in the build. |
| Account / sign-in | none. |
| Advertising ID | never read. |

### Dependencies

AndroidX and Kotlin only — `androidx.activity`, `androidx.compose`, `androidx.core`,
`androidx.datastore`, `androidx.lifecycle`, `org.jetbrains.kotlin`. No third-party SDKs.

### Components

| Component | Exported | Why |
|---|---|---|
| `MainActivity` | yes | it is the launcher icon |
| `BootReceiver` | yes | only responds to `BOOT_COMPLETED` and friends, which only the system can send |
| `GuardService` | no | internal |

### Data stored

One boolean (on/off) and a counter of how many times the torch was switched off, in the app's own
private storage. Nothing else. Uninstalling removes it.

### Verify it yourself

```sh
aapt2 dump permissions TorchLock-1.0.0.apk      # the four above, and no INTERNET
apksigner verify --print-certs TorchLock-1.0.0.apk
sha256sum -c SHA256SUMS.txt
```

**APK SHA-256**
`de40d370ef332c7d55691c9351f919201d11cab6cdc770f4f64f608d4bd62e3b`

**Signing certificate SHA-256**
`3848ac95d961e2b11d3ea274ebc735d2c4287fb1918719716f60bc470715fe37`

If either differs from what you downloaded, it is not this build.

## Installing

Not on Google Play, so Android will ask you to allow installation from this source — expected for
any app distributed as an APK. Download it, open it, allow when prompted.

## Battery

Negligible. The guard sleeps on a system callback and does nothing until the torch state changes.
While waiting for another app to release the camera it makes one call every 0.8 s, with no wakelock
and no CPU spin.

---

© Reid Phillips 2026/2027. Provided as-is, without warranty of any kind. See [NOTICE](NOTICE).
