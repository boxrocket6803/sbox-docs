---
title: "Network Events"
icon: "📆"
created: 2023-11-25
updated: 2026-09-04
---

# Network Events

Your games will likely want to react to people joining and leaving the game. To help with this you can implement `Component.INetworkListener` or `Component.INetworkSpawn` on a component and place it in the scene.


# Example


Here's an example, where on joining the game a player object is created and the incoming player is assigned as the owner.


```csharp
public sealed class GameNetworkManager : Component, Component.INetworkListener
{
	[Property] public GameObject PlayerPrefab { get; set; }
	[Property] public GameObject SpawnPoint { get; set; }

	/// <summary>
	/// Called on the host when someone successfully joins the server (including the local player)
	/// </summary>
	public void OnActive( Connection connection )
	{
		// Spawn a player for this client
		var player = PlayerPrefab.Clone( SpawnPoint.WorldTransform );

		// Find the NameTag component and set their name correctly
		var nameTag = player.Components.Get<NameTagPanel>( FindMode.EverythingInSelfAndDescendants );
		if ( nameTag is not null )
		{
			nameTag.Name = connection.DisplayName;
		}

		// Spawn it on the network, assign connection as the owner
		player.NetworkSpawn( connection );
	}
}
```


# INetworkListener


The interface `INetworkListener` has multiple methods that you can optionally override. These are all called on the host only.

| Method | Description |
|--------|-------------|
| `AcceptConnection` | Called before a connection is let in, so you can turn it away. |
| `OnConnected` | The client has connected to the server. They're about to start handshaking, in which they'll load the game and download all the required packages. |
| `OnDisconnected` | The client has disconnected from the server. |
| `OnActive` | The client is fully connected and completed the handshake. After this call they will close the loading screen and start playing. |
| `OnBecameHost` | The previous host left and you are now the host. Called on the new host only. |
| `OnHostChanged` | The previous host left and another player took over. Called on everyone except the new host. |

## AcceptConnection

Return `false` to deny the connection, and set `reason` to the message the client is shown. If any component implementing this returns `false`, the connection is denied.

```csharp
public bool AcceptConnection( Connection channel, ref string reason )
{
	if ( BannedSteamIds.Contains( channel.SteamId ) )
	{
		reason = "You're banned from this server.";
		return false;
	}

	if ( Connection.All.Count >= MaxPlayers )
	{
		reason = "Server is full.";
		return false;
	}

	return true;
}
```

## OnBecameHost

In a lobby, the host can leave. When that happens one of the remaining clients is promoted. It loads the old host's snapshot as the new host and gets this callback with the connection of the host that left, followed by `OnDisconnected` for that connection. All synced state is already in place; this is where you restart anything only the host was *doing*, like a round loop.

```csharp
public void OnBecameHost( Connection previousHost )
{
	Log.Info( $"{previousHost.DisplayName} left, we're the host now" );
	_ = RunRoundLoop();
}
```

## OnHostChanged

Everyone else gets this once their scene has been rebuilt from the new host's snapshot. Useful for a notice, or to drop anything you cached about the old host.

```csharp
public void OnHostChanged( Connection previousHost, Connection newHost )
{
	Log.Info( $"{newHost.DisplayName} is the host now" );
}
```

See [Host Migration](/networking/host-migration.md) for what survives a handoff and what needs your help.


# INetworkSpawn


The interface `INetworkSpawn` has a method to react to objects spawning on the network.

| **Method** | **Description** |
|--------|-------------|
| `OnNetworkSpawn` | Called when this object is spawned on the network. |
