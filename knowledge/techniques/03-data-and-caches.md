# Intercept a data boundary and account for caching

## When to use this

Use a data transformation when the requested behavior depends on a payload supplied
to the app: a puzzle, list, configuration, or label. The ideal boundary exposes
structured data while leaving rendering, navigation, and ordinary networking intact.
It may require less app-specific reverse engineering than changing every consumer.

## Locate the narrowest useful boundary

1. Perform one specific UI action while observing requests.
2. Match a distinctive piece of visible content to a response or parsed object.
3. Record the request identity and response schema, including IDs referenced by
   later requests. Do not assume every matching string has the same semantic role.
4. Find a point that sees the desired data before it is cached or consumed.
5. Start with logging and pass-through behavior. Confirm that the expected action
   reaches this point and that unrelated actions still work.
6. Transform one narrowly matched input while preserving the surrounding schema
   and completion behavior.

For example, a response listing active puzzle IDs may cause separate requests for
each puzzle. Replacing only the latter response has no effect when the application
already has that ID's puzzle cached. The dependency is part of the protocol, even
when the response bodies are individually simple JSON.

## URLProtocol's role

`URLProtocol` is an extension point in Foundation's URL loading system. A subclass
declares which requests/tasks it handles, supplies loading behavior, and reports
response/data/completion or failure to its client. Registering a class makes it
available to the loading system; it does not prove every networking stack in the
target will use it. Apple's current subclassing notes favor task-based overrides
in addition to or instead of request-only ones where applicable.

An observational first version can log candidate requests while declining to handle
them. A transformation version must provide a valid response contract and handle
cancellation. Scope matching to the intended host/path/method instead of accepting
every URL. Prove interception in the actual session configuration.

## Forwarding without intercepting yourself

If the interceptor forwards a request through networking that invokes the same
interceptor, it can recurse. Mark an internally forwarded request or use a forwarding
configuration that excludes the interceptor. The specific implementation depends on
the stack; test that one application request results in the intended bounded number
of downstream requests.

Conceptually:

```text
can_handle(request):
    return matches_target(request) and not already_forwarded(request)

handle(request):
    if local_response_is_available:
        deliver_response_and_data()
        complete_once()
    else:
        forward_without_reentering_this_handler()
        relay_response_data_failure_or_completion()
```

This is an implementation checklist, not compiled code from the video. Cancellation
and errors must end the same operation the caller is waiting on; do not emit both
failure and successful completion for a single finished request.

## Distinguish caches

| Layer | Typical evidence | Useful experiment |
| --- | --- | --- |
| UI or in-memory model | Old content persists without a request | Recreate the relevant screen/model and observe requests again |
| HTTP response cache | Repeated request is served without normal network traffic | Inspect cache policy and the request/response identity |
| Application database | A known object ID suppresses a detail fetch | Use a controlled new ID in test data while maintaining referential consistency |
| Web content's own storage/cache | Native requests are visible but embedded content behaves differently | Inspect the web view's process and networking path separately |

The wedding-game example used new IDs in a controlled replacement payload to force
a content fetch. That is a technique for a test data set, not a rule to randomize
IDs everywhere. Stable identity can be essential to progress, joins, and persistence.

## Web views and binary payloads

A web view can use a different networking process from the native app. Seeing
native requests in `URLProtocol` does not establish coverage of `WKWebView` traffic.
The video explores a private WebKit mechanism, which is a historical, version-sensitive
technique—not a current universal interception API. First identify the process and
supported hooks for the actual web view and OS.

Similarly, a canvas label need not exist as DOM text. A binary or compressed
response may become readable only after a decoder runs. Trace a parsed object or
decoder output and transform the meaningful value there. Editing arbitrary byte
substrings can violate lengths, encoding, checksums, or the surrounding data schema.

Sources: [network interception, 6:03](https://www.youtube.com/watch?v=vohvFpjNBz8&t=363),
[cache investigation, 27:32](https://www.youtube.com/watch?v=vohvFpjNBz8&t=1652),
[web views, 40:35](https://www.youtube.com/watch?v=vohvFpjNBz8&t=2435),
[map-label decoding, 4:25](https://www.youtube.com/watch?v=F5m2JxplnXk&t=265),
[Apple URLProtocol](https://developer.apple.com/documentation/foundation/urlprotocol).
The forwarding/checklist guidance is our synthesis for robust implementations.
