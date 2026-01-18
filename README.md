# BundlBe SDK

[![CocoaPods](https://img.shields.io/cocoapods/v/BundlBe)](https://cocoapods.org/pods/BundlBe)
![SPM](https://img.shields.io/badge/SPM-compatible-brightgreen)
![Platform](https://img.shields.io/badge/platform-iOS-lightgrey)
![Swift](https://img.shields.io/badge/Swift-5.9-orange)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

`BundlBe` is a lightweight iOS SDK for subscription activation and paywall management.

It provides three main features:

1. **Login** — authenticate and verify subscription (cached for 24h)
2. **Logout** — clear session and reset state
3. **Paywall Suppressor** — check whether paywall should be hidden

---

## Installation

### Swift Package Manager

Add the package in your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/BundlBe/BundlBe-SDK", from: "1.1.0")
]
```

Or in Xcode:
`File` → `Add Packages...` → paste repository URL → `Add Package`.


### CocoaPods

Add to your `Podfile`:

```ruby
pod 'BundlBe', '~> 1.1.0'
```

Then run:

```bash
pod install
```

---

## Usage

To use `BundlBe` in your app, you need to handle **two simple scenarios**:

- **Manual activation** - when the user enters an activation code  
- **Automatic validation** - when the app launches  

The SDK handles caching and network requests automatically.

**❗Apple may consider the display of any third‑party purchasing platforms as an attempt to redirect users, so we strongly recommend avoiding the mention of BundlBe platform name in UI.**

---

### 1. Login

The `login` method is used to verify the activation code and subscription status.

You should call `login` in two cases:

- when the user **enters an activation code manually**
- when the app **launches**, if the code was saved previously

#### Logic

- If the last successful verification was **less than 24 hours ago**,  
  the cached result is returned immediately (no network request)
- If more than 24 hours have passed, a `/login` request is sent to the backend

---

### 1.1 Manual activation (user enters a code)

You must add your own UI (a “Redeem code” button)  
where the user can enter an activation code  
(e.g. Activation screen, Settings, Onboarding).

After the user enters the code, you should:

1. save the code (for example, in `UserDefaults`)
2. call `BundlBe.login`

**Example:**

```swift
import BundlBe

// Code entered by the user
let userCode = "USER_CODE"

// Save the code for future automatic checks
UserDefaults.standard.set(userCode, forKey: "USER_CODE")

// Verify subscription
BundlBe.login(
    code: userCode,
    appID: "APP_ID",
    deviceID: UIDevice.current.identifierForVendor?.uuidString ?? ""
) { result in
    print("Login result:", result)
}
```

> The SDK does not manage UI - it only verifies the code and subscription status.

---

### 1.2 Automatic check on app launch (AppDelegate)

If the user has already activated the app before,  
they should not be asked to enter the code again.

To achieve this, on app launch you should:

- check whether an activation code is saved
- if it exists, call `login`

It is recommended to do this in `AppDelegate`.

**Example:**

```swift
import BundlBe

func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
) -> Bool {

    if let userCode = UserDefaults.standard.string(forKey: "USER_CODE") {
        BundlBe.login(
            code: userCode,
            appID: "APP_ID",
            deviceID: UIDevice.current.identifierForVendor?.uuidString ?? ""
        ) { result in
            print("Auto login result:", result)
        }
    }

    return true
}
```

This ensures that the subscription status is automatically validated  
**no more than once every 24 hours**.

---

### 2. Logout

The `logout` method clears the current session  
and resets the paywall suppression state.

**Example:**

```swift
BundlBe.logout(
    code: "USER_CODE",
    appID: "APP_ID",
    deviceID: "DEVICE_ID"
) { result in
    switch result {
    case .success:
        print("Logged out")
    case .failure(let error):
        print("Logout error:", error.localizedDescription)
    }
}
```

---

### 3. Paywall Suppressor

After calling `login`, the SDK automatically updates the paywall state.

Use `isPaywallSuppressed` to decide  
whether the paywall should be shown to the user.

**Example:**

```swift
if BundlBe.isPaywallSuppressed {
    // show content without paywall
} else {
    // show paywall
}
```

---
