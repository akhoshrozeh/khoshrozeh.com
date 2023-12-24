+++
title = 'Drone Paths'
date = 2023-12-24T08:51:14-05:00
draft = false
tags = ["Graph Theory", "Python"]
+++

This was a really interesting project I did for my Graph Theory class at UCLA. The context for the problem is the following:

_A company plans to use drones to drop sensors to the locations your scheme designated on the map, and wants your help on planning the routes of the drones. You are asked to work out two scenarios:_

**Scenario 1:**

_The company will use a large drone that can carry all the sensors; however, this drone consumes fuel fast, and thus it is important to minimize the total travel distance of the drone. The drone will start from a drone station at the center of the map (where the receiver/sink will be), and needs to return to the same point. Moreover, because it is a large drone, as it flies above a square x, it can simultaneously drop sensors not only on the square x but also on all neighboring squares inside a 5 ˆ 5 square with center x._


**Scenario 2:**

_The company will use multiple drones, that are smaller and need to obey the following restrictions:_

_• Each drone can carry up to 300 sensors at a time._

_• Drones depart again from the central node (sink) and can fly to any location on the map even across blocked areas, however, due to fuel constraints, each of these drones can travel a distance of at most 1100 units (counting the return distance) and then it needs to return to the central node to refuel._

_• The company has only M drones that can be used for deployment._


The following is my report and a link to the Python notebook I made that solves these 2 scenarios.

[Code](https://github.com/akhoshrozeh/dronePaths/blob/main/Drone_Paths_notebook.ipynb) \
[Report](../assets/Drone_Paths_Report.pdf)