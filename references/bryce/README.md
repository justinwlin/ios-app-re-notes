# Bryce Bostwick: video library and practical tips

The full distilled text for agents is in
[the agent handbook](../../knowledge/AGENT_HANDBOOK.md),
[the ten-video lessons](../../knowledge/VIDEO_LESSONS.md), and the six procedure
notes under `knowledge/techniques/`. Those files explain how the mechanisms work,
when to use them, and what to verify. This directory retains the shorter overview,
and source/chapter index.

Indexed the public uploads on [the channel's Videos tab](https://www.youtube.com/@brycedotco/videos)
on November 2, 2026: **10 videos, about 5 hours 12 minutes**. English captions were
downloaded for all ten. This scope covers that tab, not Shorts, streams, unlisted,
deleted, or otherwise unavailable videos.

The [chapter index](INDEX.md) links straight to the creator's sections and source
projects. The [machine-readable index](video-index.json) records dates, durations, and
caption provenance. Raw caption files are not included in this repository.

These are our notes from the captions and chapter metadata, with focused passages
reviewed for the tips below. They are not a claim to have visually watched every
minute or reproduced every technique. Automatic captions sometimes misrecognize
API names; use the linked video/source code to confirm exact spelling and syntax.

## Where to start for our IPA work

| Priority | Video | Useful focus |
| --- | --- | --- |
| 1 | [CatTok](https://www.youtube.com/watch?v=YW3jL2gI9IE&t=1170) | Build a framework, inject it, re-sign, and test the changed app. |
| 2 | [Disneyland freeze](https://www.youtube.com/watch?v=SpHl5_0n3Ps&t=63) | Reproduce a symptom and distinguish its actual cause before patching. |
| 3 | [Wedding games / NSURLProtocol](https://www.youtube.com/watch?v=vohvFpjNBz8&t=363) | Change a narrow data boundary instead of reconstructing the whole app. |
| 4 | [Framework replacement](https://www.youtube.com/watch?v=SuQGQ1vh9k0&t=123) | Understand method replacement, original calls, context, and scaling. |
| 5 | [Undebuggable app](https://www.youtube.com/watch?v=ih6gWZDuNME&t=538) | Relate runtime/crash addresses to a static disassembly. |

## Tips from all ten videos

### 1. This Filename Breaks iOS, And That's Boring Now

[Watch](https://www.youtube.com/watch?v=Dt4o7hz0B60).
A narrowly reproducible asset-name failure leads to a hash function returning a
special value. The practical lesson is to reduce a strange symptom to the smallest
input, then inspect the transformation of that input. Bryce also discusses using
AI tools to accelerate binary investigation and translating discoveries into
ordinary app improvements. Our application: keep the evidence and reproducible
checks alongside any AI-assisted explanation; a confident explanation alone does
not establish the binary's behavior.

### 2. How Apple Replaces Entire Frameworks

[Method replacement](https://www.youtube.com/watch?v=SuQGQ1vh9k0&t=123),
[finding the implementation](https://www.youtube.com/watch?v=SuQGQ1vh9k0&t=656),
[trampolines](https://www.youtube.com/watch?v=SuQGQ1vh9k0&t=1004).
Distinctive diagnostics help locate the relevant library. A shared hook needs
enough context to call the correct original implementation, including inheritance
cases. Small trampolines can carry that context. Later sections examine remapping
existing code and the startup cost of instrumenting many methods. Our application:
begin with one well-understood hook, preserve calling conventions and the original
route, then measure overhead before expanding instrumentation.

### 3. Apple's Widget Backdoor

[Execution model](https://www.youtube.com/watch?v=NdJ_y1c_j_I&t=104),
[firmware inspection](https://www.youtube.com/watch?v=NdJ_y1c_j_I&t=341),
[timer-based animation](https://www.youtube.com/watch?v=NdJ_y1c_j_I&t=1270).
Widgets serialize layouts into another process; they do not run like a continuously
active app view. Understanding that boundary explains the restrictions and possible
compositions. Firmware artifacts can expose system implementations without requiring
the investigation to begin on a jailbroken phone. Public timer behavior, font
ligatures, and masking can sometimes replace dependence on a private animation API.
Our application: identify who owns the state and clock before changing a timer.

### 4. Modding the Gulf of Mexico Back

[Data formats](https://www.youtube.com/watch?v=F5m2JxplnXk&t=265),
[hook placement](https://www.youtube.com/watch?v=F5m2JxplnXk&t=788),
[resilience](https://www.youtube.com/watch?v=F5m2JxplnXk&t=1296).
Visible text can come from multiple paths, including decoded binary data rendered
to a canvas. Trace the data transformation rather than assuming a DOM text node
exists. Minified names are unstable; matching expected function structure is more
selective, although it also changes over time. Our application: treat mismatches as
a reason to stop and investigate. Our exact-byte/hash guards serve the same purpose
for an updated native binary.

### 5. Debugging An Undebuggable App

[Attachment behavior](https://www.youtube.com/watch?v=ih6gWZDuNME&t=36),
[address mapping](https://www.youtube.com/watch?v=ih6gWZDuNME&t=538),
[injection crashes](https://www.youtube.com/watch?v=ih6gWZDuNME&t=794).
A debugger failing to attach, a process being killed, and an injected build crashing
can have separate causes. Map a stack frame back to the correct image before
inspecting it. Crash reports can supply useful evidence when an interactive debugger
is unavailable. Parts of this investigation use a jailbroken device; that setup is
different from our stock-iPhone sideload. Our application: capture the first actual
crash and identify its image, offset, and exception before changing more code.

### 6. NSURLProtocol: How I Stole an App For My Wedding

[Networking boundary](https://www.youtube.com/watch?v=vohvFpjNBz8&t=363),
[framework setup](https://www.youtube.com/watch?v=vohvFpjNBz8&t=1020),
[caching](https://www.youtube.com/watch?v=vohvFpjNBz8&t=1652).
Inspect network traffic first to learn whether the desired content is supplied as
data. A small framework can initially just log which requests it observes, then
handle selected requests. A hook may appear ineffective because the application
already has the data cached in a database. Our application: prove the interception
point is reached, understand caching and identifiers, and preserve unrelated
requests. This is a different strategy from directly editing Unity timer code.

### 7. Tricking iOS into Animating Icons

[Framework analysis](https://www.youtube.com/watch?v=KDVibKGtSVI&t=221),
[background behavior](https://www.youtube.com/watch?v=KDVibKGtSVI&t=472),
[service errors](https://www.youtube.com/watch?v=KDVibKGtSVI&t=920).
A public method can wrap lower-level methods, asynchronous blocks, and service
objects. Follow those boundaries to locate where an alert, restriction, or failure
actually originates. Recognizing compiler-generated block functions helps interpret
the control flow. Our application: inspect callbacks and error paths, then test
foreground/background behavior separately. OS internals in this example are
version-sensitive; successful behavior in the video is not a compatibility promise
for a current device.

### 8. Hacking Disneyland's App to fix a Freeze

[Diagnose](https://www.youtube.com/watch?v=SpHl5_0n3Ps&t=63),
[UI hierarchy](https://www.youtube.com/watch?v=SpHl5_0n3Ps&t=149),
[test competing explanations](https://www.youtube.com/watch?v=SpHl5_0n3Ps&t=593).
An app that stops accepting touches is not necessarily deadlocked. Here the view
hierarchy exposes disabled interaction during a navigation transition. Temporary
behavior changes distinguish the suspected method from a later cause. Our
application: preserve a repeatable sequence, inspect the real state, and use a
small experiment to discriminate between hypotheses before making the final patch.

### 9. Modding Plugins back into Xcode

[String references in Hopper](https://www.youtube.com/watch?v=nwWhpS1q9Ek&t=800),
[load timing](https://www.youtube.com/watch?v=nwWhpS1q9Ek&t=1200),
[packaging](https://www.youtube.com/watch?v=nwWhpS1q9Ek&t=1500).
Distinctive strings and their references provide useful starting points when symbols
alone are insufficient. Code running early at startup may need to wait until its
target classes exist. Loader compatibility checks and one-time completion prevent
surprising behavior. Our application: verify that a framework loads, that its target
class is available, and that initialization occurs once. This is a macOS/Xcode
example; its signing changes are not instructions to modify our Mac's Xcode.

### 10. Modding TikTok to only show Cat Videos

[Data source](https://www.youtube.com/watch?v=YW3jL2gI9IE&t=564),
[injection](https://www.youtube.com/watch?v=YW3jL2gI9IE&t=1170),
[refinement](https://www.youtube.com/watch?v=YW3jL2gI9IE&t=1554).
Start an injected framework with a log to prove it loads. Changing one getter can
leave related state inconsistent; the refinement updates both related collections
at a shared point. Our application: confirm the hook runs and inspect all consumers
of the affected state, including navigation and completion paths.

The video tips are research references. Confirm each technique against your own
target before relying on it.
