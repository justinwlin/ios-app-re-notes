# Runtime debugging, crashes, and address mapping

## Separate the failure types

A debugger failing to attach, an app exiting, the device restarting its UI, and a
newly injected build crashing are different observations. Preserve the first error
and determine which one occurred. An injection crash is not proof of an explicit
anti-injection check; an ordinary startup assumption, missing dependency, incompatible
architecture, or changed image layout can produce the same visible outcome.

The undebuggable-app video investigates several independent failures. Its useful
lesson for this repo is to use each newly available observation to investigate the
next failure, rather than assuming one explanation accounts for every symptom.
Its jailbroken-device setup should not be mistaken for the capabilities of a normal
Sideloadly installation on the user's phone.

## Triage procedure

1. Record whether the failure happens during signing, installation, loading, hook
   installation, or the actual changed interaction.
2. Capture the crash report or debugger stop before applying another patch.
3. Identify the exception/signal, faulting thread, top relevant application frame,
   and binary image containing that frame.
4. Confirm the image version/hash or UUID matches the file being analyzed.
5. Convert the address into the disassembler's address space.
6. Inspect the instructions around the fault and the caller's assumptions. Check
   argument values, null values, load order, and ownership/cleanup requirements.
7. Make a narrow change supported by that evidence and repeat the original scenario.

If a debugger is already available, useful LLDB inspection commands include:

```text
thread backtrace all
image list
image lookup --address <runtime-address>
```

Replace the address placeholder with the observed value. These are inspection
commands within an established debug session; they do not attach a debugger or
grant debugging access to a stock-device app.

## Keep three address spaces distinct

| Address | What it means |
| --- | --- |
| Runtime address | Where an instruction currently lives in the process, after image relocation |
| Static virtual address | Where the disassembler places it using the Mach-O image layout |
| File offset | Where the instruction's bytes occur in the file on disk |

Use debugger/image information to establish the relationship, rather than assuming
one constant offset applies to every loaded binary. With an image's runtime Mach-O
header address `L`, static header address `H`, and observed runtime address `R`:

```text
image-relative offset = R - L
static address        = H + (R - L)
file offset           = segment.fileoff + (static address - segment.vmaddr)
```

The final conversion is valid only for the containing segment's file-backed range.
If a crash report already supplies an image-relative offset, use that as the first
quantity; do not subtract the runtime base from it a second time. Record which
quantity the tool actually returned.

For our Unity framework's patched code locations, file offsets and static addresses
were equal. That was verified for this binary and is not a general property of IPAs.

## Test lifecycle timing explicitly

A check at startup can behave differently depending on whether an observer/debugger
is attached before or after it executes. When runtime access permits an early-launch
investigation, record the order of events. A successful attach to an already-running
process does not test the startup path, and a late attach failure does not identify
the exact startup instruction that caused it.

For injected frameworks, use separate markers for “image loaded,” “prerequisites
available,” “hook installed,” and “hook called.” This creates a useful progression:

```text
image loaded -> target available -> replacement installed -> target executed
```

The last marker observed narrows the failure stage. Keep the logging narrow so it
does not itself change timing or trigger recursion through the subsystem being hooked.

## Evidence to retain

Save the precise reproduction, first crash/error, device/OS/build, relevant image
identity, address conversion, instruction context, and the result of the next test.
If no device log is available, state that limit and keep the conclusion at the
static-analysis level.

Sources: [separate failure modes, 0:36](https://www.youtube.com/watch?v=ih6gWZDuNME&t=36),
[runtime-to-static mapping, 8:58](https://www.youtube.com/watch?v=ih6gWZDuNME&t=538),
[injected-build crash investigation, 13:14](https://www.youtube.com/watch?v=ih6gWZDuNME&t=794),
[Disneyland execution diagnosis, 1:03](https://www.youtube.com/watch?v=SpHl5_0n3Ps&t=63).
The triage checklist and address explanation are our engineering synthesis.
