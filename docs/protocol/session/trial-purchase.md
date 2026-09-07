# Trial & Purchase

Some Brass Monkey games were free to try and paid to keep. That meant the controller and the game needed a way to talk about trials, prompts to buy, and in-game currency. These methods carry that conversation, and like the rest of the session they travel as [`BMInvoke`](../objects/bm-invoke.md) calls on the Message channel.

## startTrial / endTrial

The controller uses these to let the host know when a trial begins and ends. Neither takes any arguments; the host simply keeps track of whether the device is currently in trial mode.

| Method | Sent by | Arguments |
|--------|---------|-----------|
| `startTrial` | Controller | 0 |
| `endTrial` | Controller | 0 |

## promptTrialUpsell

The host sends this to ask the controller to show its "buy the full version" UI. It carries no arguments, and the controller takes it from there.

| Field | Value |
|-------|-------|
| **Method** | `promptTrialUpsell` |
| **Arguments** | 0 |

## updateWallet

The host sends this after something may have changed the player's in-game currency, telling the controller to refresh its wallet balance. It carries no arguments and is sent reliably.

| Field | Value |
|-------|-------|
| **Method** | `updateWallet` |
| **Arguments** | 0 |

## WaitForNewHost

This one is about handoff. The current host sends it to tell the controller to let go of this connection and wait for another host, naming the portal the controller should come back through, for example when one game instance hands the controller off to another.

| Field | Value |
|-------|-------|
| **Method** | `WaitForNewHost` |
| **Arguments** | 1 |

### Arguments

| # | Type | Description |
|---|------|-------------|
| 1 | `string` | The portal identifier the controller should return through. Same kind of value as [`GetPortalId`](portal-id.md) answers with. |

## Behavior

`startTrial` and `endTrial` only flip a flag on the host's side, and `promptTrialUpsell` and `updateWallet` are one-way nudges that the controller acts on by itself, so none of the four expects a reply. `WaitForNewHost` is a little different: the controller stores the portal id it was given and switches itself into [`SetControlMode`](set-control-mode.md) `WAIT`, parking on its idle screen while the next host comes up. The mode change is part of handling the call rather than a separate message the host has to send.
