# Flutter share_plus Crash on iOS 26: The sharePositionOrigin Fix

If your Flutter app's share button suddenly started crashing on iOS 26, you're not alone. Here's what happened and how to fix it.

## The Problem

When using the `share_plus` package to share files or text on iOS, you may encounter this crash:

```
PlatformException(error, sharePositionOrigin: argument must be set,
{{0, 0}, {0, 0}} must be non-zero and within coordinate space of source view:
{{0, 0}, {440, 956}}, null, null)
```

This occurs when calling `Share.shareXFiles()` or `SharePlus.instance.share()` without providing the `sharePositionOrigin` parameter.

## Why This Worked Before

The `share_plus` package is the standard way to invoke the native share sheet in Flutter apps. On iOS, the share sheet is a `UIActivityViewController` that presents differently depending on the device:

- **iPhone**: Modal sheet sliding up from the bottom
- **iPad**: Popover anchored to a specific point on screen

Because iPads use a popover presentation, iOS needs to know where to anchor the popover arrow. The `sharePositionOrigin` parameter provides this—a `Rect` defining the source location.

Historically, this parameter was only required on iPad. iPhone apps could omit it because the modal presentation doesn't need an anchor point.

## What Changed in iOS 26

**Apple now validates `sharePositionOrigin` on all iOS devices, including iPhones.**

If you don't provide a valid non-zero rect, the share sheet crashes with a `PlatformException`. This was reported to the `share_plus` team in August 2025 ([GitHub issue #3645](https://github.com/fluttercommunity/plus_plugins/issues/3645)) and fixed in version 12.0.1.

## Why Testing Didn't Catch It

I had both HayTracker and PropaneTracker in the App Store and Google Play with working export functionality. I tested before release:

- AirDropped exports from both apps to my MacBook
- Saved exports directly on my development iPhone
- Verified exports on Android devices

Everything worked. The crash only appeared after updating my test iPhone to iOS 26.

The issue is timing: I tested on an older iOS version before Apple introduced the stricter validation. Once iOS 26 rolled out, the missing `sharePositionOrigin` caused crashes for users who had updated.

## The Fix

Always provide a `sharePositionOrigin` parameter:

```dart
import 'dart:ui';
import 'package:share_plus/share_plus.dart';

await Share.shareXFiles(
  files,
  subject: 'My Export',
  sharePositionOrigin: const Rect.fromLTWH(0, 0, 100, 100),
);
```

The rect values don't need to be precise for iPhone since it uses modal presentation anyway. They just need to be non-zero. `Rect.fromLTWH(0, 0, 100, 100)` works fine.

**For better UX on iPad**, anchor the popover to the actual button that triggered the share:

```dart
final box = context.findRenderObject() as RenderBox?;
final sharePositionOrigin = box!.localToGlobal(Offset.zero) & box.size;

await Share.shareXFiles(
  files,
  sharePositionOrigin: sharePositionOrigin,
);
```

## Alternative: Update share_plus

If you update to `share_plus` version 12.0.1 or later, the package handles this internally and won't crash on iPhones even without the parameter. However, you should still provide `sharePositionOrigin` for proper iPad support—it has always been required there.

## Complete Example

**Before (crashes on iOS 26):**
```dart
await Share.shareXFiles(
  files,
  subject: 'Export Data',
);
```

**After (works on all iOS versions):**
```dart
import 'dart:ui';

await Share.shareXFiles(
  files,
  subject: 'Export Data',
  sharePositionOrigin: const Rect.fromLTWH(0, 0, 100, 100),
);
```

## Key Takeaways

1. **Always provide `sharePositionOrigin`** when using share_plus on iOS, even for iPhone-only apps
2. **iOS 26 introduced stricter validation** that breaks previously working code
3. **Test on beta iOS versions** when possible, especially before major releases
4. **The fix is minimal**—just add a non-zero `Rect` parameter

---

## References

- [GitHub Issue #3645: iOS 26 Unhandled Exception: PlatformException](https://github.com/fluttercommunity/plus_plugins/issues/3645)
- [GitHub Issue #3685: SharePlus.instance.share fails in iOS 26](https://github.com/fluttercommunity/plus_plugins/issues/3685)
- [share_plus package on pub.dev](https://pub.dev/packages/share_plus)

---

*This issue affected both [HayTracker](https://apps.apple.com/us/app/haytracker/id6757104455) and [PropaneTracker](https://apps.apple.com/us/app/propanetracker/id6757357644), two Flutter apps I built using [Planning-Driven Development](https://www.nathanfox.net/p/planning-driven-development). The fix was a one-line change in each app.*
