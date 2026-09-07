# BMInvoke

`BMInvoke` is the primary Remote Procedure Call (RPC) object in BM. It encapsulates a method call, an optional callback method for the return value, and the list of arguments to be passed to the remote function.

Whenever the client or host executes a remote command (such as `registry.register` or `setCookie`), it is packaged inside a `BMInvoke` object.

- **Class ID**: 4

## Wire Format

After the [object envelope](../serialization/object-encoding.md), a `BMInvoke` contains the following fields in order:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `id` | i32 | A sequential, incrementing ID identifying this specific invocation. Used for mapping return values back to the caller. |
| 2 | `method` | UTF | The name of the remote method to execute (e.g., `"registry.register"` or `"onHostConnected"`). |
| 3 | `returnMethod` | UTF | The callback method to execute on the remote device with the result. If no callback is expected, this is sent as an empty string (`""`). |
| 4 | `argsLength` | i32 | The number of arguments being passed to the method. If there are no arguments, this is `0`. |
| 5 | `arguments` | object | If `argsLength > 0`, this is followed by exactly `argsLength` serialized objects. |

### Total minimum size

Without any arguments: 5 (envelope) + 4 + 2 + 2 + 4 = **17 bytes**
(assuming empty `method` and `returnMethod`).

## Argument Serialization

The arguments in the `arguments` sequence are written using the standard object serialization mechanism.

Because the arguments are inherently dynamically typed in the protocol, BM wraps each argument in a [`BMParameter`](bm-parameter.md) (Class ID `3`). Therefore, if `argsLength` is greater than `0`, the standard object envelope (`@`) is followed by the `BMParameter` class ID for each argument in sequence.

## Dispatch

An endpoint does not switch on the method name. It looks the method up by
reflection on a handler object, using the **name together with the argument
types it just decoded**, and calls it. The type tags in the
[`BMParameter`](bm-parameter.md) stream are therefore not documentation. They
select the method.

The lookup is an exact match. An argument that is merely *wider* than the
parameter a method declares does not widen to fit; it matches nothing at all.

This produces the protocol's most misleading failure. When no method matches:

1. The receiver discards the call.
2. Nothing is sent back, because a failed lookup is indistinguishable from a
   method with no return value.
3. The sender sees a successful write and carries on.

An implementation can therefore call an RPC for an entire session, correctly
spelled and correctly framed, and have it do nothing, with the only evidence in
a log on the other machine. Sending an interval as a double rather than a float
is enough to trigger it. See [`BMParameter`](bm-parameter.md) for which tags are
safe to emit.

## Return Values

If the resolved method has a non-void return type, the receiver automatically
sends back a `BMInvoke` carrying the returned value, using the `returnMethod`
name it was given. A void method sends nothing.

The two names in an invocation are not the same kind of thing.

- **`method` is fixed.** It is part of the protocol, and every endpoint has to
  recognise `registry.register` as `registry.register`. This is the half that
  can be written down.
- **`returnMethod` is never fixed.** It names a method on the **caller's** own
  handler, chosen by the caller, so two implementations issuing the same call
  routinely supply different names for the reply. It cannot be written down,
  because it is not the protocol's to choose.

A reply therefore echoes the `returnMethod` it was given, verbatim and without
inspecting it. Substituting a sensible looking name calls something the peer may
not have.

An empty `returnMethod` means the call is fire and forget, and no reply is sent
even if the method returns something.