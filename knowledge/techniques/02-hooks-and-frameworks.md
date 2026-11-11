# Method hooks and injected frameworks

## When to use this

Use a framework when the modification needs runtime logic, observation, or more
than a small fixed instruction edit. It can log data, replace a narrowly selected
method, transform a response, or add behavior while reusing the original app.
The framework is bundled into the app and loaded by it; the user installs one app.

## Objective-C method replacement

An Objective-C selector names a method; an `IMP` is its implementation address.
Dispatch finds the implementation for a receiver/class/selector. Changing that
mapping changes what future calls execute. `method_exchangeImplementations` swaps
two method implementations. An original implementation can also be retained as a
typed function pointer and invoked explicitly.

An implementation receives `self` and `_cmd` before the method's explicit arguments.
The replacement must match the actual argument and return conventions. A function
pointer cast is not a universal adapter for arbitrary signatures. Record whether
the target is an instance method or a class method, and inspect inheritance.
`class_addMethod` can add a subclass override, but does not replace a method already
implemented by that class. These details are documented by Apple's runtime APIs.

## A deliberate installation sequence

1. Build the correct device architecture and set an appropriate deployment target.
2. Add a tiny startup entry that emits a unique log. Inject, sign, and launch this
   baseline before adding behavior.
3. Resolve the target class and selector at runtime when the target belongs to an
   app/framework you are not linking against directly.
4. Confirm the class exists, the method exists, and the expected signature matches.
   An absent class at startup can mean its image has not loaded yet.
5. Capture the exact original implementation before replacing it. Store the
   declaring context, not merely a guessed current receiver type.
6. Install once and log that installation succeeded. Serialize competing install
   attempts so a repeated callback cannot swap the methods back.
7. Preserve original behavior on inputs outside the intended change. Test the
   matching path, the pass-through path, and repeated invocations.

Conceptual hook structure:

```text
descriptor = {
  target_class, selector, exact_signature, original_implementation
}

replacement(receiver, selector, arguments):
    if narrow_condition_matches(receiver, arguments):
        return changed_behavior_with_the_same_contract(arguments)
    return typed_original_call(descriptor, receiver, arguments)
```

This is pseudocode, not a drop-in Objective-C implementation. A real implementation
must use the target's actual signature, ownership rules, and thread assumptions.

## Initialization timing and race conditions

The Xcode plugin example shows that a loader can run before its target class exists.
Declaring a header does not create that class in memory. Resolve by name, observe
the relevant image/bundle loading, and re-evaluate the remaining prerequisites.

A useful design is to register the observer, check what is already loaded, and
repeat the check on relevant load events. Installation proceeds only when all
required classes are present and an atomic/serialized installed flag is still false.
Use a callback-once rule; initialization notifications are not a promise that your
completion should run once per event. Some load mechanisms need a different observer,
so validate that the chosen event covers the actual target image.

## Why one universal wrapper becomes difficult

A receiver's current class and selector may not uniquely identify the implementation
that was originally invoked, particularly with inheritance and calls to a superclass.
The framework-replacement video associates each intercepted method with a small
trampoline that identifies its own saved descriptor before reaching shared logic.
The shared entry preserves argument state, invokes observation logic, restores that
state, and transfers control to the original implementation.

The transferable lesson is context preservation, not “copy these assembly bytes.”
Register use, stack layout, structure returns, floating-point arguments, relocation,
and platform ABI details need explicit treatment. Instrumenting thousands of methods
also introduces startup work, locking, and reentrancy problems. Prove one hook and
measure its cost before expanding coverage.

## Failure modes and useful checks

| Symptom | Check |
| --- | --- |
| No startup log | Whether the framework was embedded, loaded, and signed successfully |
| Startup log but no hook log | Class availability, selector spelling, signature, and actual dispatch mechanism |
| Recursion | Whether the saved original really is original, or dispatch routes back to the replacement |
| Another class changes unexpectedly | Whether the modified method object belongs to a superclass |
| Works once, breaks after relaunch/event | Duplicate installation or initialization-order assumptions |
| Correct logic, corrupt arguments/result | Exact calling convention, return type, and preserved state |

Unity IL2CPP methods are native functions, so this Objective-C workflow does not
apply to them directly. Use it for a target whose methods use Objective-C dispatch.

Sources: [CatTok framework setup, 19:30](https://www.youtube.com/watch?v=YW3jL2gI9IE&t=1170),
[framework replacement and context, 5:29](https://www.youtube.com/watch?v=SuQGQ1vh9k0&t=329),
[Xcode class loading, 17:00](https://www.youtube.com/watch?v=nwWhpS1q9Ek&t=1020),
[Apple method exchange](https://developer.apple.com/documentation/objectivec/method_exchangeimplementations(_:_:)),
[Apple class_addMethod](https://developer.apple.com/documentation/objectivec/class_addmethod(_:_:_:_:)).
