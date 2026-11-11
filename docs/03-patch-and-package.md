# 3. Choose a patch strategy and package it

## The two workflows

| Approach | Output | Why choose it |
| --- | --- | --- |
| Direct binary patch | Existing executable with a few verified instructions changed | A small, understood behavior in compiled native code. |
| Injected framework | Original app plus an additional framework loaded at startup | Custom behavior that benefits from code you can edit and rebuild, as in CatTok. |

Both end with signing and installing **one modified app**. A separate app does not
simply launch an arbitrary IPA as if it were a document.

## Bryce's framework approach

In the [CatTok video](https://www.youtube.com/watch?v=YW3jL2gI9IE), Bryce builds an iOS
framework in Xcode around 19:36 and uses Sideloadly's injection options around 20:18.
The example uses Objective-C method replacement. The author's
[companion page](https://bryce.co/cattok/) links the original video.

For that style of project:

1. Identify the target behavior and how its methods are dispatched.
2. Create an iOS framework target in Xcode and implement the startup/hook logic.
3. Build for the physical device architecture, not the iOS Simulator.
4. Add the original IPA and the built framework in Sideloadly's advanced injection
   options. Check any hook-library dependencies.
5. Re-sign the combined app, install it, and test the hook on the device.

Objective-C method replacement does not automatically apply to arbitrary Swift or
IL2CPP native functions. Choose a technique that matches the actual target.
Sideloadly documents support for [framework and dylib injection](https://sideloadly.io/).

## A direct patch

A same-size instruction replacement is the smallest native change. For example,
a countdown decrement can change from:

```asm
subs w8, w8, #1
```

to:

```asm
subs w8, w8, #0
```

The surrounding store, callback path, and elapsed-time accounting remain. Only
the one update changes. An already-expired state is not repaired by this kind
of patch.

When skipping an operation, preserve its completion flow. Returning immediately
from an async state-machine function can leave the caller waiting forever. Route
the operation to an existing path whose meaning matches the intended outcome.

## Make the edit reproducible

Write a builder script that:

1. Checks the input IPA and target binary hashes.
2. Matches each original instruction before replacing it.
3. Applies non-overlapping replacements.
4. Writes a separate output IPA and refuses to overwrite existing files.
5. Checks ZIP integrity and verifies that every other member is unchanged.

Reusing the original archive entries preserves package paths and file attributes.
If repackaging manually, `Payload/` must be at the archive root; do not zip an extra
parent folder around it. A valid ZIP is still not a validly signed iOS app. Changing
a binary invalidates the old signature, and the app must be re-signed.

Before installing, check the edited instructions independently. A
[Unicorn](https://www.unicorn-engine.org/) script can execute selected
original/patched ARM64 bytes with controlled state. This provides evidence for the
local code changes; it does not run UIKit, SDKs, server calls, or the full app.
