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

Because the arguments are inherently dynamically typed in the protocol, BM wraps each argument in a [`BMParameter`](bm-parameter.md) (Class ID `3`). Therefore, if `argsLength` is greater than `0`, you will typically see the standard object envelope (`@`) followed by the `BMParameter` class ID for each argument sequentially.