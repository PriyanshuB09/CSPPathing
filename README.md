# 4188's Take on Autonomous Pathfinding

Since January, I worked on researching and prototyping different ways to run an autonomous. We had an grid-based A* algorithm last year, which we ran at our Week 4, DCMP, and offseason events. I've iterated through using AutoBuilder with PathPlanner pathFindToPose and its AD* algorithm. We ran into a couple of issues while chaining paths together, as our robot paused for a second to compute the next path. Then I tried BLine, which was a distance-parameterized path-following library, which did work, especially with little tuning; however, after listening to an Open Alliance team talk about how important a time-parameterized solution is, especially while having a modular auto, to make time for climbing. Time-parameterized just means it creates a trajectory and samples it for a point in time, makes sure the robot is at that point at the same time.

Currently, CSPPathing can generate a list of waypoints. Currently, to allow for all waypoints to be generated at the start of the auto, even for different paths, I made it save the last pose. There are a lot of features I want to add and bugs I want to fix, so this topic might turn out to be a build thread. Currently, there is some logic that exists outside of CSPPathing in our robot code, therefore I'll explain those bits and pieces on this post. CSPPathing uses a node-based A* algorithm to drastically reduce how many nodes it has to consider, taking inspiration from Team 1418, Vae Victis and their [VictiPath](https://www.chiefdelphi.com/t/1418-vae-victis-build-thread-open-alliance-2026/507948/13). Then it applies trajectory evaluations to check if a spline it generates collides with any obstacles (through PathPlanner's Waypoint class).

Link to file: [CSPPathing](https://github.com/PriyanshuB09/CSPPathing)

There are still a few kinks to work out, and I'm optimizing, but this is what I have currently. CSPPathing isn't really a library; it's meant to provide full customizability of a template for path following. It is recommended that you are familiar with AutoBuilder and PathPlannerLib to some extent, as well as how to configure PathplannerLib.

To configure CSPPathing, here are the methods you have to run on robot initialization.

```java
CSPPathing.setRobotDimensions(ROBOT_CROSS_LENGTH / 2);

CSPPathing.addObstacle(cornerPose, nonAdjacentCornerPose);

CSPPathing.configureConstraints(pathConstraints, robotConfig);

// template pathconstraints
public PathConstraints pathConstraints = new PathConstraints(
DRIVE_MAX_VEL, DRIVE_MAX_ACC,
TURN_MAX_VEL, TURN_MAX_ACC);

public RobotConfig robotConfig = new RobotConfig(MASS, MOI,
new ModuleConfig(WHEELRADIUS, MAX_VEL, COEFFICIENT_OF_FRICTION,
driveMotor, CURRENT_LIMIT, NUM_OF_MOTORS),
moduleOffsets);

CSPPathing.configureNodes(
  CSPPathing.NavNode(idString1, nodePose1, List.of(idString2, idString3,...)),
  CSPPathing.NavNode(idString2, nodePose2, List.of(idString1, idString3,...)),
  CSPPathing.NavNode(idString3, nodePose3, List.of(idString1, idString2,...)),
  ...
);
```

CSPPathing returns a list of waypoints, like this:

```java
// you can have startPose as the robot pose,
// but it won't work if you chain paths and run
// generatePath at the very beginning.

CSPPathing.generatePath(startPose, endPose);

CSPPathing.generatePath(startPose, midPose1, midPose2,..., endPose);
```

There is a lot of freedom to decide what to do with these waypoints. For example, a basic pathfollow with AutoBuilder is like this:

```java
// AutoBuilder.followPath returns a command

AutoBuilder.followPath(
    new PathPlannerPath(
      CSPPathing.generatePath(...),
      pathConstraints,
      null,
      new GoalEndState(0.0, Rotation2d.kZero) // i don't have rotation implemented
  )
);
```

This is a demonstration of it running:

![127 0 0 1 — AdvantageScope 2026-03-01 08-51-34 (online-video-cutter com)](https://github.com/user-attachments/assets/1faf2d7d-0fc9-4bf0-b61a-a9dbab99b73d)


