---
title: "Leaderboards"
icon: "🏆"
created: 2024-08-23
updated: 2024-08-23
---

# Leaderboards

Leaderboards are just stats, aggregated and ordered.


## Basic Example

Here's how to get a leaderboard. We want to get a leaderboard to show who has killed the most zombies, globally.

```csharp
var board = Sandbox.Services.Leaderboards.GetFromStat( "facepunch.ss1", "zombies_killed" );
await board.Refresh();

foreach ( var entry in board.Entries )
{
	Log.Info( $"{entry.Rank} - {entry.DisplayName} - {entry.Value}" );
}
```



:::info
By default, leaderboards are aggregated using the sum of all the stats and ordered descending.

:::


## By Country

You can filter a leaderboard by country. This will only show stats that were created in that country.

```csharp
var board = Sandbox.Services.Leaderboards.GetFromStat( "facepunch.ss1", "zombies_killed" );
board.SetCountryCode( "gb" );
await board.Refresh();

foreach ( var entry in board.Entries )
{
	Log.Info( $"{entry.Rank} - {entry.DisplayName} - {entry.Value} [{entry.CountryCode}]" );
}
```


:::success
If you pass countrycode as "auto" it will use the current player's location

:::


## By Date

You can filter leaderboards by date, allowing yearly, weekly, monthly or daily leaderboards.

```csharp
var board = Sandbox.Services.Leaderboards.GetFromStat( "facepunch.ss1", "zombies_killed" );
board.FilterByMonth();
board.SetDatePeriod( new System.DateTime( 2024, 8, 1 ) );
await board.Refresh();

foreach ( var entry in board.Entries )
{
	Log.Info( $"{entry.Rank} - {entry.DisplayName} - {entry.Value} [{entry.CountryCode}]" );
}
```


:::success
If you don't set a date period, it'll use the current date

:::


## Centering

You can focus the leaderboard on a certain player. This will show the results around that player. It is nice to show a player's contemperies rather than showing the top 20 all the time.

```csharp
var board = Sandbox.Services.Leaderboards.GetFromStat( "facepunch.ss1", "zombies_killed" );
board.CenterOnSteamId( 76561197960279927 );
await board.Refresh();

foreach ( var entry in board.Entries )
{
	Log.Info( $"{entry.Rank} - {entry.DisplayName} - {entry.Value}" );
}
```


:::success
You can also call `.CenterOnMe()` to center on the local player.

:::


## Aggregation and Sorting

So maybe instead of the sum of something, we want to show people's shortest result. Like in this example, we're showing a list of the fastest win times.

```csharp
var board = Sandbox.Services.Leaderboards.GetFromStat( "facepunch.ss1", "victory_elapsed_time" );

board.SetAggregationMin(); // select the lowest value from each player
board.SetSortAscending(); // order by the lowest value first
board.FilterByMonth(); // only show results from this month
board.CenterOnMe(); // offset so I'm in the middle of the results
board.MaxEntries = 100; 

await board.Refresh();

foreach ( var entry in board.Entries )
{
	Log.Info( $"{entry.Rank} - {entry.DisplayName} - {entry.Value} [{entry.Timestamp}]" );
}
```


:::success
You can aggregate by sum, min, max, avg or last.

:::


:::info
The entry timestamp holds the time of the selected result

:::
