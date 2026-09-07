---
title: "Host Migration"
icon: "🔁"
created: 2026-09-04
updated: 2026-09-04
---

# Host Migration

In a lobby one player is the host. When they leave, the game is handed to another player and everyone carries on. This is on by default and needs no code.

If the host crashes or loses connection there is nothing to hand over, so the game ends for everyone.

The whole page in one example. **State** moves to the new host. **Running code** does not.

```csharp
// Fine. The value is state, the new host has it and keeps counting down.
[Sync] public TimeUntil RoundEnds { get; set; }

protected override void OnUpdate()
{
	if ( Networking.IsHost && RoundEnds )
		EndRound();
}
```

```csharp
// Not fine. The value is state, but the task waiting on it is code running on the old host.
// When they leave, nothing ends the round.
[Sync] public TimeUntil RoundEnds { get; set; }

protected override void OnStart()
{
	_ = EndRoundLater();
}

async Task EndRoundLater()
{
	await GameTask.DelaySeconds( RoundEnds );
	EndRound();
}
```

The same goes for a plain field instead of `[Sync]`, an `Invoke`, or a `static`. If it isn't in the snapshot, the new host doesn't have it.


## What Happens

1. The leaving host picks the longest connected player and tells everyone.
2. It sends that player a snapshot of the game and waits for them to confirm. Everything sent before it, RPCs included, arrives first.
3. The new host loads the snapshot as the host. It gets `OnBecameHost`, then `OnDisconnected` for the player who left.
4. Everyone else rebuilds their scene from the new host and gets `OnHostChanged`.

`OnActive` is not called again for players already in the game, and `ISceneStartup.OnHostInitialize` does not run on the new host.


## What Survives

| Survives | Lost |
|----------|------|
| `[Sync]` properties, on components and `GameObjectSystem`s | Plain fields and properties |
| `[Property]` values | `static` fields |
| Networked objects and their owners | Pending `Invoke` calls |
| `INetworkSnapshot` data | Running `async` methods |
| `Time.Now` | The new host's own local-only objects |
| Connection permissions and replicated ConVars | |

Anything a snapshot carries survives. Anything that was code running on the old host does not.

Synced values from the snapshot are applied again after `OnAwake`, `OnStart` and `OnEnabled` run, so initialising a `[Sync]` property there doesn't overwrite what the host had.


## Host State In Systems

A dictionary on a `GameObjectSystem` is the most common host-only state. `GameObjectSystem` supports `[Sync]`, so this is one attribute if the key is already a `SteamId` or `Guid`.

```csharp
// Lost
Dictionary<long, int> _propsPerPlayer = new();

// Survives
[Sync] public NetDictionary<long, int> PropsPerPlayer { get; set; } = new();
```


## Timers Are State

A pending `Invoke` is code waiting on the old host, so it dies with it. Keep the deadline in a synced `TimeUntil` and act on it in `OnUpdate`.

```csharp
[Sync] public bool IsEnabled { get; set; } = true;
[Sync] public TimeUntil RespawnAt { get; set; }

void OnPickedUp()
{
	IsEnabled = false;
	RespawnAt = RespawnTime;
}

protected override void OnUpdate()
{
	if ( Networking.IsHost && !IsEnabled && RespawnAt )
		IsEnabled = true;
}
```

The same goes for `async` loops that animate doors or platforms. Drive them from `OnFixedUpdate` off synced state, or the door is stuck with `IsMoving` set forever.


## Sync Connections

A `Connection` in a plain field means nothing on another machine. Synced, it travels as its id, and ids survive migration.

```csharp
[Sync] public Connection Killer { get; set; }
```


## Restart Host Loops In OnBecameHost

Anything the host was doing, rather than storing, has to be started again.

```csharp
public void OnBecameHost( Connection previousHost )
{
	// The round clock is [Sync] and kept ticking, so pick up where it is
	_ = RunRoundLoop();
}
```

A plain "previous state" field used to detect transitions resets too. Sync it, or the transition fires again.


## Warnings

The analyzers that ship with the engine warn about these patterns in Visual Studio and Rider. They're off for projects that destroy the lobby when the host leaves.

| Id | Catches |
|----|---------|
| `SB3002` | An unsynced `Connection` stored on a component or system |
| `SB3003` | A plain `TimeSince` or `TimeUntil` used by host-gated code |
| `SB3004` | `Invoke` on a component with synced state |
| `SB3005` | A `[Sync]` property written after an `await` |


## Opting Out

Set **Destroy Lobby When Host Leaves** in the project's networking settings, or `DestroyWhenHostLeaves = true` on your `LobbyConfig`. The game then ends when the host leaves.


## Testing

While hosting in the editor, pick **Migrate host to new instance** from the network menu. An instance is spawned, joins, and takes over as host once the editor disconnects. Its log shows `Becoming the host`. See [Testing Multiplayer](/networking/testing-multiplayer.md).
