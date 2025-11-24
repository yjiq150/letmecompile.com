# VirtualTablet - Bluelink Development Task List

## Phase 1: Foundation (Week 1-2)

### 1.1 Project Setup
- [x] Initialize Flutter project with proper package structure
- [x] Configure go_router for navigation management
- [x] Set up Riverpod for state management
- [x] Create Pigeon schema structure for type-safe platform channels
- [x] Set up logger package with file output capability
- [x] Implement storage service using shared_preferences
- [x] Configure Firebase project for Analytics and AdMob
- [x] Set up development environment for Android (Kotlin)
- [x] Set up development environment for iOS (Swift)
- [ ] Configure build configurations for debug/release modes

### 1.2 UI Framework
- [x] Create base theme with #F16078 brand color
- [x] Design color palette and typography system matching VirtualTablet aesthetics
- [x] Implement responsive layout utilities for different screen sizes
- [x] Set up orientation handling (portrait/landscape)
- [x] Configure Flutter l10n with app_en.arb file
- [x] Create common UI components (buttons, dialogs, cards)
- [x] Implement loading states and error handling UI patterns
- [x] Set up navigation structure with go_router routes
- [ ] Create app icon and launch screen designs

## Phase 2: Android Bluetooth Implementation (Week 3-4)

### 2.1 Native Bluetooth Layer
- [x] Extract Digitizer HID descriptor from Android POC project
- [x] Extract Mouse HID descriptor from Android POC project
- [x] Copy ClassicHidService.kt functionality for Bluetooth Classic
- [x] Copy necessary permissions and service configurations of AndroidManifest.xml from POC project
- [x] Create device search functionality with name filtering
- [x] Implement device pairing logic
- [x] Implement connection establishment and management
- [x] Create HID report generator for digitizer (stylus mode)
  - [x] Implement X/Y absolute positioning
  - [x] Add pressure sensitivity support
  - [x] Add tilt X/Y support
  - [x] Add twist (barrel rotation) support
  - [x] Implement barrel button handling
  - [x] Add eraser tip detection
- [x] Create HID report generator for mouse (finger touch mode)
  - [x] Implement X/Y relative positioning
  - [x] Add left/right click support
  - [x] Implement double-tap as double-click
- [x] Set up Pigeon interfaces for Bluetooth methods
- [x] Implement platform channel communication handlers

### 2.2 Connection Management
- [x] Implement auto-reconnection logic
- [x] Create connection state machine (Disconnected → Pairing → Paired → Connecting → Connected)
- [x] Add connection status monitoring
- [x] Implement error handling and recovery mechanisms
- [x] Add connection timeout handling
- [x] Create connection persistence for app restart
- [x] Implement graceful disconnection handling

## Phase 3: Core Pages - Android (Week 5-6)

### 3.1 IntroPage
- [x] Design and implement onboarding flow UI
- [x] Create feature explanation screens
- [x] Add visual guide on how to use the app
- [x] Implement 'Getting Started' button with PermissionDialog trigger
- [x] Implement navigation to ConnectPage after permissions granted (a user can skip it and allow permissions later)

### 3.2 PermissionDialog
- [x] Create centralized permission management dialog UI
- [x] Implement permission status checking for all required permissions
- [x] Add visual indicators for allowed/disallowed permissions
- [x] Implement permission request flow with user guidance
- [x] Add support for Android API level-specific permissions
- [x] Implement POST_NOTIFICATIONS permission (API 33+)
- [x] Implement BLUETOOTH_SCAN permission with neverForLocation flag (API 31+)
- [x] Implement BLUETOOTH_CONNECT permission (API 31+)
- [x] Implement BLUETOOTH_ADVERTISE permission (API 31+)
- [x] Implement ACCESS_FINE_LOCATION permission (≤API 30)
- [x] Add cross-page usage support (IntroPage, ConnectPage, TabletPage)
- [x] Create permission explanation text for each permission type
- [x] Implement permission retry logic for denied permissions
- [x] Add navigation to system settings for permanently denied permissions

### 3.3 ConnectPage
- [x] Create paired devices list UI
- [x] Implement device search and pair UI using BottomSheet
- [x] Add connection status indicators for each paired device
- [x] Implement settings shortcut button (floating action button)
- [x] Add 'connecting' indicator when connect button on paired clicked
- [x] Implement error messages for connection failures
- [x] Add refresh functionality for device search list
- [x] Integrate PermissionDialog trigger for missing Bluetooth permissions

### 3.4 TabletPage
- [x] Implement touch input capture using GestureDetector
- [x] Add stylus input detection and capture (location, inRange, tipswitch, barrelswitch, eraser, pressure, tiltX, tiltY, twist) for stylus mode
- [x] Add touch input detection and capture (touch location, up/down/move) for finger touch mode
- [x] Make two modes exclusive .
- [x] Convert received input into HID input report
- [x] Implement report transmission over Bluetooth
- [x] Add visual feedback for active drawing area (hovering/touching) should be minimal (not too distract).
- [x] Create distinct visual boundary for drawable area
- [x] Implement orientation support (landscape/portrait)
- [x] Add mode indicator (stylus/finger touch)
- [ ] Integrate banner ad at bottom (Google AdMob)
- [ ] Add TabletConfig button
- [x] Implement connection status indicator
- [ ] Add auto-reconnect with fallback to ConnectPage

## Phase 4: Configuration & Settings (Week 7)

### 4.1 TabletConfig Dialog
- [x] Create dialog UI layout
- [x] Add configuration persistence using SharedPreference. (Base code already implemented in tablet_config_provider.dart)
- [x] Implement mode selection (stylus/finger touch. If stylus H/W available, default value: stylus. If not, default value: finger)
- [x] Add resolution configuration input fields (ratio of this input used to draw table area. Default value: 16:9)
- [x] Add orientation selector (trigger orientation_menu_bottom_sheet.dart when touch and show current orientation, default value: landscape)
- [x] Add scale adjustment slider (50-100%: default value: 100%)
- [x] Add left-hand mode toggle (default value: false)
  - when enabled in landscape mode, move widgets generated by _buildLeftToolbar function to placed on right side of screen)
  - when enabled in portrait mode, internal item orders of widgets in _buildLeftToolbar should be reversed)
- [x] Add pressure sensitivity slider (50-300%: default value: 100%)

### 4.2 SettingPage
- [ ] Create settings page layout
- [ ] Display app version information
- [ ] Implement log viewer functionality
- [ ] Add website link (sunnysidesoft.com)
- [ ] Create OSS licenses page
- [ ] Implement About section
- [ ] Add TabletConfig shortcut
- [ ] Create IAP purchase button (for premium upgrade)
- [ ] Add purchase restoration option

## Phase 5: iOS Implementation (Week 8-10)

### 5.1 Native iOS Layer
- [ ] Extract HID descriptors from iOS POC project
- [ ] Port BluetoothManager.swift functionality
- [ ] Implement HOGP (HID over GATT Profile) support
- [ ] Create device discovery and advertising
- [ ] Implement connection establishment for iOS
- [ ] Port HID report generation for digitizer
- [ ] Port HID report generation for mouse
- [ ] Implement Pigeon interface for iOS
- [ ] Add platform channel handlers in Swift
- [ ] Optimize for Apple Pencil input

### 5.2 iOS-specific UI Adjustments
- [ ] Adapt connection flow for iOS (discoverable mode)
- [ ] Implement iOS-specific permission requests
- [ ] Ensure compliance with iOS design guidelines
- [ ] Optimize UI for iPad and iPhone form factors
- [ ] Test and optimize Apple Pencil responsiveness
- [ ] Implement iOS-specific connection status handling
- [ ] Add iOS-specific error messages

## Phase 6: Features & Monetization (Week 11)

### 6.1 Shortcuts & Hotkeys
- [ ] Research and finalize list of popular shortcuts for graphic artists
- [ ] Design customizable button mapping UI
- [ ] Implement Undo shortcut (Ctrl+Z)
- [ ] Implement Redo shortcut (Ctrl+Y/Ctrl+Shift+Z)
- [ ] Add brush size controls ([/])
- [ ] Implement zoom functionality
- [ ] Add pan/hand tool support
- [ ] Create tool shortcuts (Brush-B, Eraser-E, Eyedropper-I)
- [ ] Implement shortcut persistence
- [ ] Add default shortcut presets

### 6.2 IAP Integration
- [ ] Set up IAP products in app stores (~$5 USD)
- [ ] Implement premium upgrade flow UI
- [ ] Add ad removal logic based on purchase status
- [ ] Implement purchase validation
- [ ] Add purchase restoration functionality
- [ ] Create purchase success/failure handling
- [ ] Implement platform-specific purchase logic (no cross-platform)
- [ ] Add Firebase Analytics events for conversion tracking
- [ ] Test sandbox purchases on both platforms

### 6.3 Analytics Integration
- [ ] Configure Firebase Analytics
- [ ] Implement app launch event
- [ ] Add connection success/failure events
- [ ] Track mode switches (stylus/finger)
- [ ] Monitor IAP conversion events
- [ ] Add feature usage statistics
- [ ] Implement session length tracking
- [ ] Add user retention metrics

## Phase 7: Polish & Optimization (Week 12)

### 7.1 Performance Optimization
- [ ] Profile and optimize input latency (target < 16ms)
- [ ] Optimize Bluetooth transmission efficiency
- [ ] Reduce memory usage and garbage collection
- [ ] Implement battery usage optimization
- [ ] Add smart idle states when inactive
- [ ] Optimize UI rendering performance
- [ ] Minimize app startup time

### 7.2 Testing & Bug Fixes
- [ ] Conduct cross-device testing on Samsung Galaxy Tab (S-Pen)
- [ ] Test on Pixel Tablet
- [ ] Test on various Samsung phones
- [ ] Test on other Android phones
- [ ] Test on iPad Pro with Apple Pencil
- [ ] Test on iPad Air
- [ ] Test on various iPhone models
- [ ] Verify Windows 10/11 compatibility
- [ ] Test macOS 12+ with companion software
- [ ] Validate connection stability
- [ ] Test input accuracy across devices
- [ ] Fix identified bugs
- [ ] Perform regression testing

### 7.3 Release Preparation
- [ ] Create app store descriptions
- [ ] Prepare promotional screenshots
- [ ] Write privacy policy
- [ ] Create terms of service
- [ ] Prepare release notes
- [ ] Set up app store listings
- [ ] Configure release build settings
- [ ] Generate signed APK/AAB for Android
- [ ] Create iOS archive for App Store
- [ ] Submit for app store review

## Post-Launch Tasks

### Future Features (Priority 3)
- [ ] Research keyboard HID descriptor requirements
- [ ] Implement keyboard HID support
- [ ] Add keyboard shortcut configuration
- [ ] Create keyboard input UI

### Maintenance
- [ ] Monitor crash reports
- [ ] Track user feedback
- [ ] Plan update schedule
- [ ] Monitor analytics for performance metrics
- [ ] Respond to app store reviews

## Testing Checklist

### Unit Tests
- [ ] Test HID report generation accuracy
- [ ] Test coordinate transformation logic
- [ ] Test settings persistence
- [ ] Test connection state machine
- [ ] Test input filtering logic

### Integration Tests
- [ ] Test complete Bluetooth connection flow
- [ ] Test input capture and transmission pipeline
- [ ] Test auto-reconnection scenarios
- [ ] Test IAP purchase flow
- [ ] Test ad integration

### Performance Tests
- [ ] Measure input latency (must be < 20ms end-to-end)
- [ ] Verify 60+ fps input capture
- [ ] Test battery usage (< 5% per hour)
- [ ] Validate 99% connection stability
- [ ] Test memory usage over extended sessions

### User Experience Tests
- [ ] Verify < 30 seconds from launch to drawing
- [ ] Test < 3 taps to connect known device
- [ ] Validate 95%+ auto-reconnection success
- [ ] Test all UI flows and navigation paths
- [ ] Verify localization correctness

---

*Total Tasks: 200+*  
*Estimated Timeline: 12 weeks*  
*Priority: Android first, then iOS*  
*Focus: Stylus/digitizer mode before finger touch*
