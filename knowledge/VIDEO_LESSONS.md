# Distilled lessons from all ten collected videos

This file states the reusable information in text. The linked videos are sources,
not required reading. Detailed implementation procedures are in `techniques/` and
the agent handbook. These notes summarize caption/metadata research; the examples
are not all techniques reproduced on a device.

## V01 — This Filename Breaks iOS, And That's Boring Now

A particular asset name failed because an internal filename transformation reached
a special hash value. The visible failure looked like a missing image, but the
important input was its name rather than its pixels. This illustrates a useful
debugging order: isolate the smallest failing input, compare it with a passing
input, inspect the transformation between the input and its consumer, and test
special values. A practical fix should follow the established mechanism rather
than a guess that the entire asset pipeline is broken.

For agents: propose hypotheses and discriminating observations before editing. AI
can help explore candidates, but retain the exact input, binary evidence, and test
that supports the conclusion. A game analogue is proving which stored time value
the loss rule reads instead of assuming a method called “Timer” is the right target.

Detailed procedure: [state and hypotheses](techniques/01-state-and-hypotheses.md).
Source: [video](https://www.youtube.com/watch?v=Dt4o7hz0B60).

## V02 — How Apple Replaces Entire Frameworks

Objective-C dispatch can be redirected while preserving the original implementation.
Doing this for many methods requires more than sharing a wrapper: the wrapper needs
the original method's identity and calling context. Current receiver class and
selector alone can lose information in inheritance/superclass cases. The explored
design associates a small trampoline with a saved descriptor and preserves argument
state while shared observation logic runs.

For agents: record the exact original implementation and signature per hook. Test
inheritance, return values, and argument preservation. Broad instrumentation has
startup and locking costs, so measure before scaling it. The later remapping work
is an advanced implementation strategy; it is not needed for a fixed one-instruction patch.

Detailed procedure: [hooks and frameworks](techniques/02-hooks-and-frameworks.md).
Source: [context and trampolines](https://www.youtube.com/watch?v=SuQGQ1vh9k0&t=329).

## V03 — Apple's Widget Backdoor

A widget is not a continuously running app view. The host receives a serialized
presentation and controls its display schedule, so normal in-process animation
assumptions can fail. The investigation examines a system clock-related primitive,
then composes timer text, custom glyphs/ligatures, masks, and shifted schedules to
produce richer visible behavior. Studying system artifacts explains which process
and serialization boundary impose the restriction.

For agents: write down the host's execution model before attempting a workaround.
List supported primitives and test compositions of them before assuming a private
interface is necessary. Preserve OS/build-specific evidence; historical accessibility
of an interface does not prove it is stable today. A Unity game timer is different:
the game owns a native remaining-time object rather than a widget presentation.

Detailed procedure: [system frameworks](techniques/06-system-frameworks.md).
Source: [execution model](https://www.youtube.com/watch?v=NdJ_y1c_j_I&t=104).

## V04 — Modding the Gulf of Mexico Back

The same visible label can arrive through multiple representations. Some information
is available as JSON; other labels are decoded from binary data and drawn on a
canvas, so neither DOM text replacement nor searching raw responses necessarily
finds the useful boundary. Following the decoder can expose an accessible function
that produces the meaningful value. A hook must execute in the environment that
owns that function, not an unrelated isolated scope.

For agents: identify representation and execution scope before choosing a hook.
Generated/minified names are unstable. Match a known structural expectation, verify
that the match is unique, and stop applying the transformation if it changes. Our
native equivalent is matching both the binary hash and original instruction bytes.

Detailed procedures: [data boundaries](techniques/03-data-and-caches.md) and
[patch guards](techniques/05-patches-and-completion.md).
Source: [decoder and hook investigation](https://www.youtube.com/watch?v=F5m2JxplnXk&t=265).

## V05 — Debugging An Undebuggable App

Debugger attachment failure, a process/device-UI failure, and an injected-build
crash can have different causes. Each needs its own first observation. The runtime
image containing a fault and its load location connect a stack/crash address to the
corresponding static instructions. That mapping permits useful analysis even when
the next observation comes from a crash report rather than an interactive debugger.

For agents: classify the failure stage, preserve the first error, confirm the image
identity, and convert the address correctly before inspecting it. Do not call every
injection crash “anti-tamper.” Some demonstrations use a jailbroken device; a stock
iPhone with a sideloaded app does not automatically provide the same debug access.

Detailed procedure: [debugging and addresses](techniques/04-debugging-and-addresses.md).
Source: [failure investigation](https://www.youtube.com/watch?v=ih6gWZDuNME&t=538).

## V06 — NSURLProtocol: How I Stole an App For My Wedding

When content is supplied as structured data, a narrow loading-system interceptor
can transform that content while reusing the app's interface. First proving that
the interceptor observes the request separates installation problems from
transformation problems. Application caches can suppress a fetch even when HTTP
interception works. Related identifiers in a listing response and detail responses
must remain coherent. Native and embedded web content can also use different
networking paths.

For agents: log pass-through behavior before replacing responses, match requests
narrowly, preserve response/completion contracts, and inspect cache ownership.
A controlled fresh identifier can test a database-cache hypothesis, but randomizing
production identifiers indiscriminately can break persistence. A native interceptor
does not imply coverage of every web view or custom transport.

Detailed procedure: [data and caches](techniques/03-data-and-caches.md).
Source: [networking boundary](https://www.youtube.com/watch?v=vohvFpjNBz8&t=363).

## V07 — Tricking iOS into Animating Icons

A public API may be a wrapper around checks, asynchronous blocks, private methods,
and service objects. Following that chain distinguishes a visible notification
from the work that changes the icon and from restrictions encountered in the
background. Compiler-generated block functions may hold the error or completion
logic that explains an otherwise confusing decompilation.

For agents: follow the actual work and inspect deferred branches before bypassing
a wrapper. Test foreground and background behavior separately. Record which OS
artifact supplied the implementation; private interfaces discovered there are
version-sensitive. The corresponding app lesson is to retain an operation's
completion flow while skipping its work, rather than returning from an arbitrary
wrapper.

Detailed procedure: [system frameworks](techniques/06-system-frameworks.md).
Source: [framework analysis](https://www.youtube.com/watch?v=KDVibKGtSVI&t=221).

## V08 — Hacking Disneyland's App to fix a Freeze

An apparently frozen screen can be executing normally while its view hierarchy
rejects user input. Reproducing the gesture, inspecting the stack, and examining
the interaction state reveal different evidence than assuming a main-thread hang.
Temporary changes to gesture decisions help distinguish an early rejection from a
later lifecycle callback that disrupts an active transition.

For agents: design a change that separates competing explanations and record the
prediction before running it. A workaround that suppresses the trigger is not yet
a root-cause explanation. Keep the final intervention scoped to the established
problem and retest adjacent interactions. Our analogous verification connects the
actual remaining-time field to the loss predicate rather than trusting appearance.

Detailed procedure: [state and hypotheses](techniques/01-state-and-hypotheses.md).
Source: [diagnosis](https://www.youtube.com/watch?v=SpHl5_0n3Ps&t=63).

## V09 — Modding Plugins back into Xcode

A distinctive string can lead through references to a useful function when direct
symbol browsing gives little direction. A loader can also run before the classes
it needs exist: making a declaration compile does not make the target image load
earlier. Runtime lookup, prerequisite checks, load observation, and a one-time
completion rule solve a different problem from the hook's own implementation.

For agents: distinguish linking, loading, target availability, and successful
invocation. Recheck what is already loaded as well as future load events so an
initialization race does not leave the hook permanently absent. The example concerns
a modified macOS developer tool; its packaging/signing steps are not automatically
the right procedure for an iPhone app.

Detailed procedures: [hooks and frameworks](techniques/02-hooks-and-frameworks.md)
and [runtime diagnosis](techniques/04-debugging-and-addresses.md).
Source: [Hopper string references](https://www.youtube.com/watch?v=nwWhpS1q9Ek&t=800).

## V10 — Modding TikTok to only show Cat Videos

The end-to-end modification embeds an iOS framework into an existing IPA, signs
the combined app, and installs it as one app. A startup-only log is a useful first
test of this packaging path. The later filtering refinement illustrates that
changing a getter can leave related backing collections or direct field consumers
inconsistent. A shared intervention point can keep those consumers aligned.

For agents: separately prove “framework loaded,” “hook installed,” and “desired
behavior with coherent state.” Our project reused the artifact-analysis/signing
workflow, but used a direct native patch because the target was Unity IL2CPP code.
Do not describe our result as an injected launcher or a rebuilt Unity project.

Detailed procedures: [hooks and frameworks](techniques/02-hooks-and-frameworks.md)
and [state ownership](techniques/01-state-and-hypotheses.md).
Source: [framework integration](https://www.youtube.com/watch?v=YW3jL2gI9IE&t=1170).

## Retrieval terms

Search this file and `techniques/` for: swizzling, IMP, selector, inheritance,
trampoline, callback, async, completion, cache, URLProtocol, web view, IL2CPP,
remaining time, timeout, UI freeze, gesture, image base, file offset, signature,
load order, widget, serialization, private API, version guard, or byte patch.
