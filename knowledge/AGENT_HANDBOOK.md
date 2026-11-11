# Agent handbook: investigate, modify, and verify an iOS app

This is the working reference for agents in this repository. It contains the
procedures and conclusions needed to begin work without watching videos, opening
external articles, or downloading the local media library. Source links in the
technique notes provide attribution and optional deeper investigation.

The guidance combines a Unity IL2CPP app investigation, engineering deductions from
that work, and techniques demonstrated in the ten Bryce Bostwick videos collected
on 2026-11-02. A demonstrated technique is not automatically verified for your app,
its operating-system version, or your phone.

## 1. Establish what is being changed

Translate the request into an observable outcome. “Freeze the timer” is ambiguous;
“the player can remain in an active level beyond its normal duration without a
timeout loss” is testable. “Skip this step” needs a choice about its result:
reporting the step unavailable and reporting it completed are different behaviors.

Record the original IPA hash, app bundle ID, version/build, architecture, executable
hash, engine, and relevant metadata version. Keep the original input immutable and
write modified artifacts to a separate directory. The same app name or build label
does not establish that two binary files have the same instructions at an address.

A decrypted IPA contains inspectable compiled code. It is neither an Xcode project
nor recovered C# source. An IPA is already a ZIP package, so renaming it is optional.
An app's small launch executable may delegate the important work to a framework.

This iPhone workflow uses IPA files; APK files are for Android. An already-patched
IPA can go directly to SideStore for signing and installation.

## 2. Pick an investigation route

| Observation | Start here | Evidence needed before modifying |
| --- | --- | --- |
| Unity framework plus IL2CPP metadata | Map types/method tokens to native code, then disassemble selected functions | Exact binary/metadata pairing and executable-address checks |
| Objective-C classes and selectors are visible | Trace the relevant method and its callers; consider method replacement | Correct receiver, declaring class, signature, original implementation, and load timing |
| Desired content arrives as structured network data | Observe requests, parsing, and caches | A request that produces the content and proof the proposed interceptor sees it |
| A visible value appears wrong | Trace its state owner and consumers | The write that changes the value and the decision that uses it |
| A screen seems frozen | Inspect execution and the UI's input state | Reproduction, thread state, and view/gesture hierarchy |
| Injected code crashes at launch | Inspect the first crash and loaded images | Exception, faulting image, relative address, and startup sequence |
| System behavior differs between OS versions | Inspect the matching framework or firmware artifact | OS/build/architecture and a minimal reproducible example |

Do not assume all iOS code supports Objective-C swizzling. Swift functions may or
may not use Objective-C dispatch; Unity IL2CPP methods are native functions with
separate metadata. Confirm the actual dispatch mechanism.

## 3. Locate the code that owns the behavior

Use the strongest available lead: a class name, unique diagnostic string, metadata
method name, observed request, crash address, or visible UI label. Follow references
and callers until the state owner and its consumer are connected. A text search
locates a candidate; it does not prove that candidate controls the requested behavior.

For a stripped Unity build, pair the native framework with its matching metadata.
Validate table boundaries, method token indexes, pointer alignment, and executable
section membership. A metadata name is useful evidence, but its associated native
instructions determine what the compiled build does.

Hopper is optional. Agents can usually work from metadata mapping, `otool`,
`llvm-objdump`, and scripts. Use Hopper only when raw disassembly is not enough.
In Hopper, open the correct native executable and architecture, navigate to a
known entry address, define a procedure if needed, and inspect assembly alongside
pseudocode. Save the database and record which procedures were actually analyzed.
If full analysis stalls, a focused disassembly can still answer a narrow question.
Do not label an entire app “decompiled” because a single function exported successfully.

## 4. Separate state, presentation, and outcomes

Draw the small dependency chain relevant to the request:

```text
state update -> stored value -> decision predicate -> consequence
                         \-> UI representation
```

Changing only the UI representation can hide the countdown while the loss check
continues to run. Changing a state update can affect both the UI and the decision,
but only if both read the same state. Inspect the read/write addresses or fields to
prove that connection. Also check copied collections, snapshots, direct field
access, and getters that may expose different views of the same data.

Example: a Unity level timer stores remaining seconds at a fixed object offset.
An update method decrements it, an `IsTimeUp` method checks whether it is below 1,
and the win/loss handler delegates its active-timer check to that method. Changing
the decrement to subtract zero targets the state used by the normal timeout
decision, not a text label. Other timers with their own state remain unaffected.

## 5. Make the smallest change that preserves the contract

Use a direct instruction patch when the behavior is small, local, and supported by
clear control-flow evidence. Use an injected framework when the change needs
maintainable runtime logic, observation, or a supported interception point. Neither
approach is universally preferable.

Before returning early or replacing a result, identify the function's contract:

- What is its return type and calling convention?
- Does the caller expect a completion callback, task result, or event?
- Does it own a lock, temporary state, allocation, or cleanup action?
- Does success imply a reward or some other state transition?
- Can initialization or the target method run more than once?

An async wrapper or state-machine method is not interchangeable with a synchronous
void method. Skipping an operation should complete it through a valid existing
path, rather than leave a task pending. An “unavailable” result is not the same as
a successful completion.

For a framework, first install a version whose only behavior is an identifying
startup log. Prove that the framework loads before debugging the hook itself. Then
add one narrow change and preserve the original behavior for nonmatching inputs.

## 6. Verify at the correct level

| Check | What it supports | What it does not prove |
| --- | --- | --- |
| Input hashes and expected-byte matches | The patch is applied to the intended artifact | Correct behavior after installation |
| Disassembly and control-flow tracing | What the selected instructions and branches do | Every runtime caller or server decision |
| Repacked archive comparison | Which files/bytes changed and whether the ZIP is intact | A valid iOS signature |
| Instruction emulation | Behavior for the explicitly modeled machine state | UIKit, SDK, callback, or network behavior not executed |
| Device test | Observed behavior on that device/build and scenario | Every other device, level, OS, or long-session case |

Record the scope alongside a result. A 10,000-tick emulation that stops at the tick
callback entry does not run the callback body or the complete app. It proves only
that the patched state remains unchanged under those modeled ticks. A separate
static trace explains why that state matters to the decision.

For a new patch, compare original and modified behavior on the same modeled input.
Check boundary cases such as 1 second remaining, an already-expired state, a null
adapter, and an existing adapter. Preserve evidence of both intended changes and
important neighboring behavior, such as elapsed-time accounting or task completion.

## 7. Sign, install, and test on the phone

Changing an executable invalidates its prior signature. Re-sign the complete app
with the sideloading tool. In Sideloadly, select the actual iPhone; the Apple Silicon
entry selects the Mac. The Apple ID field takes the normal Apple Account email,
not a Team ID. Use the modified IPA as input.

Connect and unlock the phone, trust the Mac, and enable Developer Mode for this
installation on iOS 16 or newer. After installation, an Untrusted Developer prompt
is resolved by trusting the signing email under Settings → General → VPN & Device
Management. Developer Mode and developer trust are separate steps.

Sideloadly's refresh daemon runs on the computer. For a later IPA that should
re-sign on the phone, use [the SideStore guide](../docs/sidestore.md). SideStore
still needs the computer once, to install it and to replace an expired pairing
file. Record each sideload in [the per-app template](../templates/sideload-app.md).
Keep Apple Accounts, pairing files, and IPAs out of Git.

SideStore's `-80045` sign-in error was resolved by disabling
on-device Anisette, selecting SideStore (New) with rotation off, fetching and saving
remote client overrides, resetting `adi.pb`, and retrying. The
[complete workaround](../docs/sidestore.md#sign-in-error-80045) includes sources and
evidence limits. Sign-in success does not yet confirm pairing, first refresh, or
app behavior. To transfer an app installed by Sideloadly, import its finished IPA
into SideStore once while leaving the current installation in place. No extra code
or pairing file needs to be embedded in that IPA; SideStore manages later signing.

On the phone, exercise the changed behavior, its neighboring flows, and
background/resume. Record the result; do not infer device success from the fact
that signing or installation completed.

## 8. Detailed technique notes

The following files contain procedures, mechanisms, failure modes, and expected
evidence entirely in text:

- [Locating state and testing hypotheses](techniques/01-state-and-hypotheses.md)
- [Method hooks and injected frameworks](techniques/02-hooks-and-frameworks.md)
- [Data interception, caches, and web views](techniques/03-data-and-caches.md)
- [Runtime debugging and address mapping](techniques/04-debugging-and-addresses.md)
- [Version-aware patches and completion paths](techniques/05-patches-and-completion.md)
- [System frameworks and constrained execution](techniques/06-system-frameworks.md)

Every collected video is mapped to these distilled lessons in
[VIDEO_LESSONS.md](VIDEO_LESSONS.md). The local videos and captions are optional
research inputs, not prerequisites for reading or applying this handbook.
