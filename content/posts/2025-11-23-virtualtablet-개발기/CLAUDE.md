# Coding Rules

## Rule of thumbs

- When you are not sure about the implementation related to Bluetooth communication channel and its configurations,
  refer to poc project code because it is already tested and verified that it is working.
- When working with `docs/tasks.md`, you MUST refer to `docs/requirements.md` and `docs/plan.md` as well for overall project understanding.
- When handling strings which are used in UI, make sure define new key in `app_en.arb` file. And use l10n code.
- When the code include pigeon bridge to communicate between dart and native code, make sure to review the both sides of code.
- Reuse code whenever possible. If you need the UI component or utility function, scan codebase first. Write new code if there is no reusable code. 

## Dart Code Style

- Wrap the future that do not need to be awaited with `unawaited`.

### Snackbar

Use `LetterboxSnackBar`. instead of using `ScaffoldMessenger.of(context).showSnackBar()`


### file / class management rules

- Put Dialog, BottomSheet code in separate files. Do not mix them in 'page' files.
- Even if a widget is only used in one file, extract it to a separate file if its code has more than 50 lines.


### localization (l10n)

Don't use `L` class. instead use the following pattern.

```dart
import 'package:vt_bluelink/l10n/vt_app_localizations.dart';

S.current.{key_name}
```

**Exception: Developer Menu**
- Developer menu items and their related dialogs use hardcoded English strings instead of localization.
- Do NOT add developer menu related strings to any `.arb` files
- Use plain English strings directly in the code for all developer options.
- This includes: menu items, descriptions, success messages, and confirmation dialogs.

### pigeon

- Do not create multiple instances of the same @HostApi() or @FlutterApi() annotated classes. Make them singleton.


### riverpod

- DO NOT access `ref` in overridden `dispose` function of a class that extends `ConsumerState`.
  - ref to notifier should be stored in a member variable of the class and used inside `dispose` function 
- See `riverpod_rules.md` for other details

#### mount check requirement

- Async operation? → Always check mounted. (But you can skip check only if it's a normal notifier without invalidation)
- Purely synchronous state changes? → No need.


## iOS Code Style
- When writing swift code, use 2-space for indent.


## Android Code Style
- When writing kotlin code, use 2-space for indent and follow the standard kotlin style.
