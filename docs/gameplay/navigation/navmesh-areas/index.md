---
title: "NavMesh Areas"
icon: "⏹️"
created: 2024-12-03
updated: 2025-10-29
---

# NavMesh Areas

NavNesh Areas can affect NavNesh generation and agent behavior/pathing.
 ![](./images/navmesh-areas-2.png)The NavMeshArea component is used to define the location, shape and type of an area.

![](./images/navmesh-areas-1.png)

You can also specify the Area for a link component.

![](./images/navmesh-areas.png)The NavMeshAreaDefinition resource is used to define properties of the Area.

# Limitations

* Currently there is a limit of 24 NavMeshAreaDefinition, but you can assign them to as many Area Components as you like
* **Static** areas are basically free.
* **Moving** areas are a bit more expensive, but you should be able to have at least a couple of dozens of them.
