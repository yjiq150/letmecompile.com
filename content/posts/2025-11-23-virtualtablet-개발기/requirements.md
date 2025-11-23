# VirtualTablet - Bluelink Requirements Document

## 1. Product Overview

### 1.1 Product Description
VirtualTablet - Bluelink is a mobile application that transforms Android and iOS devices into wireless graphics tablets for desktop computers using Bluetooth HID protocol. The app emphasizes **Bluetooth connectivity**, **low latency**, and **easy configuration** compared to network-based alternatives.

### 1.2 Key Value Propositions
- **Bluetooth HID Protocol**: Direct hardware-level communication for ultra-low latency
- **No Network Configuration**: Simple Bluetooth pairing, no IP addresses or network setup
- **Cross-Platform Support**: Works with Windows (no drivers needed) and macOS (with companion software)
- **Professional Stylus Support**: Full pressure sensitivity and tilt for digital art

### 1.3 Target Users
- Digital artists and illustrators
- Graphic designers
- Note-takers and students
- Anyone needing a portable graphics tablet solution

## 2. Technical Requirements

### 2.1 Platform Requirements
- **Android**
  - Minimum SDK: 28
  - Target devices: Both phones and tablets (equal priority)
  - Stylus support: Samsung S-Pen, other active styluses
  
- **iOS**
  - Minimum version: 15.x
  - Target devices: iPad (with Apple Pencil support) and iPhone
  - Universal app supporting both form factors

### 2.2 Technology Stack
- **UI Framework**: Flutter (Dart)
  - Navigation: go_router
  - State Management: Riverpod
  - Localization: Built-in Flutter l10n (`S.current.{KEY}`)
  
- **Native Layer**
  - Android: Kotlin
  - iOS: Swift
  - Bridge: Pigeon for type-safe platform channels
  
- **Bluetooth Protocol**
  - HID (Human Interface Device) protocol
  - Reuse exact HID descriptors from POC projects
  - Support for:
    - Digitizer HID (stylus mode) - Priority 1
    - Mouse HID (finger touch mode) - Priority 2
    - Keyboard HID (future release) - Priority 3

### 2.3 Third-Party Services
- **Analytics**: Firebase Analytics
- **Monetization**: 
  - Google AdMob (banner ads)
  - In-App Purchase (IAP) for premium upgrade
- **Logging**: File-based logging for troubleshooting

### 2.4 Performance Requirements
- **Input Latency**: < 16ms processing time
- **Transmission Rate**: Minimum 60Hz (60 reports per second)
- **Battery Optimization**: Smart idle states when inactive

## 3. Functional Requirements

### 3.1 Core Features

#### 3.1.1 Connection Management
- **Android Connection Flow**:
  - Search discoverable Bluetooth devices (filter by name)
  - Display paired Bluetooth devices
  - Custom pairing UI (reuse DevicesDialogFragment from POC)
  - Option to open system Bluetooth settings
  - Auto-connect to last device on app launch
  - Single device connection only (no multi-device support)
  
- **iOS Connection Flow**:
  - Make device discoverable for PC-initiated connection
  - Search and connect to available devices
  - Store connection method for auto-reconnection
  - Auto-connect on app launch

#### 3.1.2 Input Modes
- **Stylus Mode** (Primary Priority):
  - Absolute positioning
  - Pressure sensitivity (per HID specification)
  - Tilt support (X/Y angles)
  - Twist support (barrel rotation)
  - Barrel button support (stylus side buttons)
  - Eraser tip support
  - Ignore all finger touches when active
  - No palm rejection needed
  - No pressure curve adjustment
  
- **Finger Touch Mode** (Secondary):
  - Relative positioning (mouse emulation)
  - Left and right click buttons
  - Double-tap gesture = double-click
  - No multi-touch support (simple mouse only)
  - Ignore stylus input when active

#### 3.1.3 Tablet Configuration
- Display resolution setting (width x height)
- Default aspect ratio: 16:9
- Tablet view scale: 50-100% of screen
- Orientation support: Portrait and Landscape
- Fixed orientation option (manual selection)
- Left-hand mode (mirror UI elements)

#### 3.1.4 Shortcuts & Hotkeys
Popular shortcuts for graphic artists:
- **Essential**: Undo (Ctrl+Z), Redo (Ctrl+Y/Shift+Z)
- **Brush**: Increase/Decrease size ([/])
- **View**: Zoom, Pan, Rotate canvas
- **Tools**: Brush (B), Eraser (E), Eyedropper (I)
- **Customizable mapping** for user preferences

### 3.2 User Interface Requirements

#### 3.2.1 Pages Structure

**IntroPage**
- App functionality explanation
- Visual guide on how to use
- When 'Getting Started' button clicked, show PermissionDialog before move on to ConnectPage

**PermissionDialog**
- Show PermissionDialog if all required permissions are not ready yet.
- this dialog can be shown in several pages where permissions are required (Intro, ConnectPage, TabletPage). 
  only show it to user when all required permissions are not allowed yet.
- Show the status of permissions that are allowed/disallowed and guide a user to allow it
- List of required permissions
  - android.permission.POST_NOTIFICATIONS 
    - Runtime from Android 13 (API 33) onward. User sees the notification permission dialog.
  - android.permission.BLUETOOTH_SCAN 
    - Runtime from Android 12 (API 31) onward. Required to discover/scan for nearby Bluetooth devices. With neverForLocation, it avoids location prompt (but still runtime).
  - android.permission.BLUETOOTH_CONNECT
    - Runtime from Android 12 (API 31) onward. Needed to pair and connect with devices.
  - android.permission.BLUETOOTH_ADVERTISE
    - Runtime from Android 12 (API 31) onward. Required if your app advertises itself as a Bluetooth peripheral.
  - android.permission.ACCESS_FINE_LOCATION (only on API ≤30)
    - Needed to scan for nearby BT devices pre-Android 12. Runtime prompt (the “Allow location access” dialog).


**ConnectPage**
- Platform-specific connection UI
- Device list with connection status
- Auto-reconnect indicator
- Settings button (floating action button)

**TabletPage**
- Drawable area (visually distinguished)
- Banner ad at bottom (free version)
- TabletConfig button
- Mode indicator (stylus/finger)
- Connection status indicator
- Auto-reconnect with fallback to ConnectPage

**TabletConfig Dialog**
- Input mode selection (stylus/finger)
- Left-hand mode toggle
- Resolution settings
- Orientation lock settings
- Scale slider (50-100%)

**SettingPage**
- App version
- Tablet configuration (opens dialog)
- Website link (sunnysidesoft.com)
- View logs
- OSS licenses
- About section

#### 3.2.2 Visual Design
- **Brand Color**: #F16078 (pink/coral)
- **Design Language**: Match existing VirtualTablet app aesthetics
- **Responsive**: Support all screen sizes and orientations
- **Accessibility**: Follow platform accessibility guidelines

#### 3.2.3 Localization
- **Primary Language**: English (app_en.arb)
- **Implementation**: Use `S.current.{TRANSLATION_KEY}`
- **Other Languages**: To be added post-development
- **Focus**: English-only during development phase

### 3.3 Business Requirements

#### 3.3.1 Monetization Model
- **Free Version**:
  - Full functionality
  - Banner ads (bottom of TabletPage)
  - Firebase Analytics tracking
  
- **Premium Version** (via IAP):
  - Ad removal only
  - Price: ~$5 USD (subject to change)
  - Same binary/app (feature flag based)
  - Purchase restoration support
  - Platform-specific purchases (Android and iOS separate)
  - No trial period

#### 3.3.2 Analytics Events
Track key user actions:
- App launch
- Connection success/failure
- Mode switches (stylus/finger)
- IAP conversion
- Feature usage statistics

## 4. Non-Functional Requirements

### 4.1 Development Approach
- **Platform Priority**: Android first, then iOS
- **Architecture**: Clean separation between Flutter UI and native Bluetooth
- **Code Reuse**: Maximum code sharing between platforms via Pigeon
- **Logging**: Comprehensive logging for remote troubleshooting

### 4.2 Security Requirements
- No sensitive data storage
- Secure Bluetooth pairing
- No network permissions required (Bluetooth only)
- IAP validation

### 4.3 Compatibility
- **Desktop OS Support**:
  - Windows 10/11 (no additional software)
  - macOS 12+ (requires tablet-event-injector from POC)
  - Linux (future consideration)

### 4.4 Release Strategy
1. Android release first
2. iOS release after Android stability
3. Keyboard HID support in future update

## 5. Constraints & Assumptions

### 5.1 Technical Constraints
- Must use exact HID descriptors from POC projects (extract from code)
- Bluetooth Classic for Android (not BLE, refer POC android project)
- iOS Bluetooth uses HOGP (HID over GATT Profile, refer POC iOS project)
- 60Hz minimum transmission rate
- No additional Bluetooth profiles beyond HID required

### 5.2 Design Constraints
- Visual consistency with existing VirtualTablet app
- Banner ad must not interfere with drawing area
- All platform-specific code isolated in native layer

### 5.3 Assumptions
- Users have Bluetooth-capable computers
- Stylus users have compatible active stylus (S-Pen, Apple Pencil)
- Users understand basic Bluetooth pairing process

## 6. Success Criteria

### 6.1 Performance Metrics
- Input latency < 20ms end-to-end
- 60+ fps input capture
- < 5% battery drain per hour of active use
- 99% connection stability

### 6.2 User Experience Metrics
- < 30 seconds from app launch to drawing
- < 3 taps to connect to known device
- Successful auto-reconnection > 95%

### 6.3 Business Metrics
- IAP conversion rate target: 5-10%
- User retention: 30% after 30 days
- Average session length: 15+ minutes

## 7. Resolved Clarifications

All initial questions have been addressed and incorporated into the requirements above:
- ✅ macOS companion software exists in POC (tablet-event-injector)
- ✅ HID descriptors to be extracted from POC code
- ✅ No additional Bluetooth profiles needed
- ✅ No palm rejection required
- ✅ No pressure curve adjustment
- ✅ Single device connection only
- ✅ IAP price ~$5 USD
- ✅ No trial period
- ✅ Platform-specific purchases

## 8. Development Priorities

### Phase 1 - Core Foundation
1. Flutter project structure with go_router and Riverpod
2. Pigeon schema definition
3. Logging infrastructure
4. Basic navigation flow

### Phase 2 - Android Bluetooth
1. Port HID descriptors from POC
2. Bluetooth connection management
3. Stylus input capture and HID reports
4. Auto-reconnection logic

### Phase 3 - User Interface
1. All pages with English localization
2. Brand color theming (#F16078)
3. Responsive layouts
4. Settings persistence

### Phase 4 - Features
1. Finger touch mode
2. Configuration options
3. Shortcuts implementation
4. IAP integration

### Phase 5 - iOS Port
1. Swift Bluetooth implementation
2. iOS-specific UI adjustments
3. Apple Pencil support

### Phase 6 - Polish
1. Performance optimization
2. Analytics integration
3. Ad integration
4. Final testing

---

*Version 1.1 - Updated with all clarifications resolved*
