---
title: "Connection Permissions"
icon: "🔑"
created: 2024-03-21
updated: 2024-03-21
---

# Connection Permissions

The host can change some permissions for a specific `Connection`. The ideal place to set these permissions would be in the `OnActive` [network event.](/networking/network-events.md)


# Spawning Objects

You can set `Connection.CanSpawnObjects` to allow or disallow a specific connection to create their own networked objects. By default this is `true`.


# Refreshing Objects

By default only the host can send network refresh updates for networked objects. This can be changed to allow the owner of a networked object to also send these updates with `Connection.CanRefreshObjects`.
