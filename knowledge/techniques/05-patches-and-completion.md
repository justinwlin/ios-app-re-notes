# Version-aware patches and preserved completion paths

## Choosing a modification method

| Method | What changes | Useful for | Main constraint |
| --- | --- | --- | --- |
| Direct binary patch | Existing machine instructions in the executable or a framework | Small changes to a count update, condition, or result | Addresses and original bytes belong to a particular build |
| Injected framework or dynamic library | Added compiled code loaded inside the app, with hooks into selected behavior | Custom logic, diagnostics, a settings overlay, or several related changes | Loading, hook signatures, dispatch mechanism, and initialization timing must be correct |
| Asset or configuration edit | Bundled images, text, data, or actual configuration values | Appearance and rules that the app really reads from those files | A hardcoded or server-controlled rule cannot be changed just by adding a setting |
| Runtime instrumentation | Observations or temporary interventions in a running process | Finding the relevant state and prototyping a change | Debug access and permitted mechanisms depend on the device and app setup; it is not automatically a distributable patch |
| Rebuild from source | A new compiled app or component | Larger changes when usable source exists | Decompiled pseudocode is not the original buildable project |

Direct binary patching does not require reconstructing the original Unity project.
A timer patch can replace a compiled subtraction instruction with another
instruction of the same size. More complex edits need carefully arranged control
flow and space for added code; arbitrary insertion can invalidate addresses and
Mach-O layout. Hopper helps interpret the code, while a guarded script writes
and packages the final bytes.

An injected framework runs inside the resulting app. It is not necessarily a
separate launcher. Sideloadly supports embedding frameworks and dynamic libraries,
but this does not make every target method hookable by the same technique.
Objective-C method replacement and Unity IL2CPP native hooks have different
requirements. See [hooks and frameworks](02-hooks-and-frameworks.md).

## How signing fits the workflow

Changing executable bytes invalidates the old signature. Modifying signed resources
or embedding a new framework also requires the resulting app to be signed again.
The signature establishes the signer and the integrity of the signed content; it
does not make the content permanently uneditable.

```text
original decrypted IPA -> extract -> modify -> package -> re-sign -> install
```

Sideloadly handles signing for its Apple ID installation workflow. The installed
copy uses that signing identity and its permitted provisioning, rather than keeping
the original publisher's valid signature. Nested executable code must be signed
appropriately as part of signing the complete bundle. Editing it again after
signing requires another signing pass. A valid signature is necessary but does not
prove that the modification works, or that publisher-specific services and
entitlements will remain available under a different signing identity.

SideStore can perform that re-sign on the phone after the IPA is built. The
procedure, free-account limits, and refresh steps are in
[the SideStore guide](../../docs/sidestore.md).

Sources: Apple's [application code signing explanation](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/AppSigning.html),
[current signature-format and nested-code guidance](https://developer.apple.com/documentation/Xcode/using-the-latest-code-signature-format),
and [Sideloadly's signing and injection features](https://sideloadly.io/).

## Identify and guard the artifact

Store both the original artifact identity and the assumptions under which a change
is valid. A native patch should identify the binary hash, architecture, file offset,
expected original bytes, replacement bytes, and why that location controls the
behavior. Refuse a mismatch. Checking only the displayed version or filename does
not protect against a different build or previously modified executable.

For a JavaScript hook, minified names can change even when the apparent product
version does not. The map-label video matched expected function structure to locate
a candidate instead of hardcoding a generated name. This also has limits: formatting,
implementation, scope, and behavior can change. A safe adaptation is to require a
unique expected match and leave the original behavior intact when it is absent or
ambiguous. A failed guard is evidence to re-analyze, not an invitation to remove it.

## Patch construction checklist

1. Read the original artifact and verify its identity.
2. Translate the chosen virtual address to the correct file offset.
3. Match the original instruction bytes and instruction boundaries.
4. Validate replacement length, branch destination, encoding, and overlap rules.
5. Write a separate modified file; do not make the baseline your working copy.
6. Compare bytes and archive members to prove the scope of changes.
7. Inspect or emulate the resulting instructions independently of the patch writer.
8. Re-sign, install, and test the complete app for the intended scenario.

Do not mistake a `.hop` database save for an exported patched executable. A saved
analysis database preserves labels and analysis state; the build pipeline must
explicitly produce and package the intended native bytes.

## Synchronous returns versus async operations

A synchronous function's caller receives its result when the function returns.
An asynchronous operation can return before its work has completed; a separate
callback, promise/task, event, or state machine reports the eventual result.
Replacing an async body with a bare return can prevent that completion signal.

Before skipping work, trace both its entry and its successful/unavailable/error
completion paths. Prefer an existing path whose semantics match the intended
outcome. “Unavailable” is not the same as “completed successfully.”

Conceptual contract:

```text
request -> pending -> complete(result)
                   -> fail(error)
                   -> cancel()
```

Every supported outcome must settle the operation appropriately. The exact
completion mechanism is framework-specific. This diagram does not mean all APIs
expose all three outcomes or that an arbitrary early return is a cancellation.

## Why a state patch can be smaller than a policy patch

Freezing the underlying positive remaining-time value keeps a timer's ordinary
expiry predicate false. Replacing every loss handler would be a broader change that
could affect other reasons for ending the level. A state patch is useful only after
the writer-to-consumer connection is established; otherwise it can be as misleading
as editing the label.

Changing subtraction by one into subtraction by zero preserves the surrounding
tick and elapsed-time updates. An already-zero state remains expired, so test with
a fresh positive countdown. Other timers with separate state remain unchanged.

## Validation record

Retain input/output hashes, the before/after bytes, changed-member list, emulation
inputs and outputs, and device observations. Label proof scopes precisely. A
10,000-tick test is useful for the modeled countdown, but it does not establish that
every timed mechanic or server-side rule has been disabled.

Sources: [matching unstable functions, 21:36](https://www.youtube.com/watch?v=F5m2JxplnXk&t=1296).
The completion-path guidance is grounded primarily in a Unity app investigation.
