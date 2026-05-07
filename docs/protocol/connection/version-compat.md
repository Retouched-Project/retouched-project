# Version Compatibility

After the [handshake](handshake.md), each peer checks whether its own version satisfies the other peer's minimum requirement.

## Compatibility Check

The check uses only the **major** and **minor** components of the version. The **build** component is ignored.

A version A is considered "at least" version B if:

1. A's major version is greater than B's major version, **or**
2. A's major version equals B's major version **and** A's minor version is greater than or equal to B's minor version.

Each peer performs this check: "Is my `currentVersion` at least as high as the remote peer's `minVersion`?"

## Outcome

| My version vs. remote minimum | Result |
|---|---|
| My `currentVersion` >= remote `minVersion` | Compatible. Connection proceeds. |
| My `currentVersion` < remote `minVersion` | Incompatible. Connection is closed. |

Both peers perform this check independently. If either side fails, the connection is terminated. The implementation distinguishes between two failure cases: the local device being too old for the remote peer's requirements, and the remote peer being too old for the local device's requirements.