---
title: "Stats"
icon: "📉"
created: 2024-08-23
updated: 2024-09-09
---

# Stats

Your game can record stats for each player that plays it. Here are some examples of things stats can record.

* Number of zombies killed
* Fastest time
* Total meters walker
* Coins collected
* Longest time
* Bullets fired
* Kills per weapon

You can query and display these stats, or use them in any other way you want. You can query stats from individual players, and you can get the compound stats globally.

# Recording Stats

Depending on what you're doing, you either want to increment your stat..

```csharp
public void OnZombieKilled()
{
    Sandbox.Services.Stats.Increment( "zombies-killed", 1 );
}
```

Or just set it directly..

```javascript
public void OnGameFinished()
{
    Sandbox.Services.Stats.Increment( "wins", 1 );
    Sandbox.Services.Stats.SetValue( "win-time", SecondsTaken );
}
```


:::info
You can call these apis as often as you like. We batch the stats and send them when we're ready.

:::


You can also add data when calling SetValue. This is in the format of a `Dictionary<string, object>` and this data is available when querying leaderboards.


# Reading Stats

Stats are available in two forms. Either global, or player.

```csharp
// get global zombie kill count
var zombies = Sandbox.Services.Stats.Global.Get( "zombies_killed" );

Log.Info( $"there have been {zombies.Sum} zombies killed by {zombies.Players} players!" );
```

```csharp
// Get local player zombie kill count
var zombies = Sandbox.Services.Stats.LocalPlayer.Get( "zombies_killed" );

Log.Info( $"You have killed {zombies.Sum} zombies!" );
```

```csharp
// Get stats for Garry in SS1
var stats = Sandbox.Services.Stats.GetPlayerStats( "facepunch.ss1", 76561197960279927 );

// wait for them to download
await stats.Refresh();

var zombies = stats.Get( "zombies_killed" );

Log.Info( $"Garry has killed {zombies.Sum} zombies!" );
```
