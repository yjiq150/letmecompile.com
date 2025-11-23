# vt_bluelink spec

I am planning to develop an app that works as a virtual graphic tablet called 'VirtualTablet - Bluelink'
By receiving stylus pen input from devices like Samsung Galaxy Tab, Galaxy Note, VirtualTablet transfers this data wirelessly to your desktop or laptop, perfectly simulating your stylus movements.
It uses Bluetooth as communication channel.
(FYI: desktop/latop with Windows, there' no need to install additional software, but for Mac, it additional software installation is required)

## techical requirments
- flutter with android/ios support (flutter project template is already configured in bluelink/vt_bluelink)
- iOS min version requirement 15.x. Need to support both iPad/iPhone
- Android min SDK requirement 28
- All UI layers should be developed by flutter, but bluetooth connection/sending/receiving data will be handled and written in native code in each platform.
  - all necessary method for those features should be defined in bluelink/vt_bluelink/pigeon/schema.dart.
  - no need to complete this schema at initial stage. add mehtods step by step with the each tasks layer.
- always store logs for troubleshooting later
- planning to release Android first then move on to iOS. so make overall code structure in a way that it can fit for both Android and iOS, but focus on Android first.


## user scenario by Pages

### IntroPage

- briefly explain to user how the app works

### ConnectPage

- connect to Windows/Mac using Bluetooth
- allow user to pair and connect as smooth as possible.
- once successfully connect to a device, keep that device for auto connection on app launch.
- iOS and Android use different way of making connection. (different UIs are necessary for making a connection)

- iOS
  - Make iOS device discoverable then connect from OS's bluetooth setting menu in PC.
  - Search available bluetooth devices list then a user initiate connection by selecting it.
  - once connection is successful, store the way it connect last time and automatically try connecting to the the device on the next app launch.
  - refer ViewController for connection process
  - BluetoothManager.swift for bluetooth channel in in 'bluelink-poc/ios/BluetoothMouse'
  - you should write native swift code to handle bluetooth channel and create bridging interface with pigeon

- Android
  - Provide custom Bluetooth search/pairing UI or just let user open Android Settings > Bluetooth menu for search/pairing.
    - Follow design and reuse native code in DevicesDialogFragment in bluelink/poc/android
  - List paired devices and a user can itinitate connection by selecting it
  - once connection is successful, auto connect to the recently connected device on the next app launch.
  - refer BluetoothConnectActivity.kt in 'client/anrdoid' for pairing and connection process
  - refer ClassicHidService.kt for bluetooth channel in 'bluelink-poc/android'
  - you should write kotlin code to handle bluetooth channel and create bridging interface with pigeon

- a floating button to SettingPage

### TabletPage

It shows tablet view to receive user inputs from stylus pen and finger touches.
Received events are converted into HID input report and sent through Bluetooth channel that opened in ConnectPage

- This page supports all orientations (landscape, portrait)
- drawable area (tablet view) should be visually distinguished
- banner ad will take the bottom space of the page
- have a button to open TabletConfig Dialog
- auto reconnect indicator (if it fails to auto reconnect, pop back to Connect Page)
- when stylus pen mode is enable, all touches will be ignored. and vice-versa for finger touch mode.
- finger touch mode provides left and right buttons to send click event. double tap gesture will be considered same effect of double click of left button.


### TabletConfig Dialog

- select finger touch mode or stylus mode
- lefthand mode (all the menu or buttons)
- user can change display resolution [width] x [height]. the tablet view should be rendered to follow this width/height ratio. (default width/height ratio is 16:9)
- enable fixed orientation (ignore orientation changes by sensor. choose the orientation manually)
- tabletView scale slider 50~100% (100% is the size of maximum tabletView that fit in the screen)


### SettingPage
- app version
- tablet config (launch TabletConfig Dialog)
- sunnysidesoft.com website link
- See logs
- OSS licenses
- about
