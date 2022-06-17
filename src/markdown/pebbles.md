# Description

This article is a run through of my [master's
thesis](./media/masters/thesis.pdf) from which [this
publication](https://doi.org/10.1007/s11370-021-00350-1) is derived. This
work deals with coordinating multiple robots in online order fulfilment
systems.

# Visiting Pebbles on Rectangular Grids - The Ideas 

The name was inspired by [Kornhauser et
al](https://doi.org/10.1109/SFCS.1984.715921)'s work entitled *Coordinating
Pebble Motion On Graphs* (PMOG). **Abstractly**, pebbles represent agents
requiring access to shared resources.
[Graph](https://en.wikipedia.org/wiki/Graph_theory) nodes represent the
shared resources. Specifically, only one pebble can access a node at a
time. The goal of PMOG is to determine whether a target configuration is
reachable from the initial arrangement of pebbles and provide the sequence
of required moves if it is. 

**Concretely**, PMOG is applicable in robotic warehousing systems: the
pebbles represent robots that access physical locations. This example is
not as far fetched as it sounds: thousands of robots are delivering
packages from storage to packing stations in warehouses every day and Kiva
Systems is the company that pioneered their use. It's also not new: Staples
was the first company to go live with a Kiva system in 2006! A 2016 report
estimated that Amazon was operating ~30k units. The image below shows the
layout of a typical robotic distribution center.

![Distribution Center Layout ([image
source](https://dl.acm.org/doi/abs/10.5555/2908675.2908681))](media/masters/kivaLayout.png)

Robots pick up and deliver shelves of inventory to pick/pack workers in the
Kiva system as shown below. 

![Kiva Robot Carrying Inventory Pod ([image
source](https://dl.acm.org/doi/abs/10.5555/2908675.2908681))](media/masters/kivabot.png)

PMOG generalizes and solves a fundamental problem. However, the constraints
could be relaxed for this application: robots only need to **visit**
destination nodes, they don't need to be in a pre-defined end configuration
at the same time. **Hence the 'visiting' part of the title.**

This relaxation has implications for the feasibility check of the original
PMOG: it's easy to come up with feasible visiting PMOG cases that are
infeasible in PMOG. Since warehouses are rectangular workspaces, my work is
restricted to graphs made up of
[bi-connected](https://en.wikipedia.org/wiki/Biconnected_graph) rectangular
grids. As long as there is at least one open node and the graph is
bi-connected, every destination assignment is feasible in **Visiting Pebble
Motion on Rectangual Grids.**

# The Implementation

## A first try

Let robots find their way using something like
[A\*](https://en.wikipedia.org/wiki/A*_search_algorithm), if another robot
gets in the way, simply wait until the node is free again. Using the
[Alphabet Soup](https://research.csc.ncsu.edu/alphabetsoup/) multi-robot
simulator, I modified the default behaviour to strictly adhere to the
underlying navigation graph. Then, I only allowed robots to enter a node if
it is empty. The snapshot below shows what it looked like: the robots are
pink, replenishing stations are blue, shelves are purple and packing
stations are green. The shelves contain letters and the parcels being
packed are words from a dictionary.  

<video alt="Alphabet Soup Simulation of Distribution Centre" controls>
<source src="media/masters/soup.mp4" type="video/mp4"> </video>

There is a problem though: eventually the robots get stuck waiting for
each other as shown in the image below. This is known as
[deadlock](https://en.wikipedia.org/wiki/Deadlock).

![Robots in an infinite wait](media/masters/deadlock.png)

## A step back - dealing with deadlocks... one at a time.

Let's focus on a single robot that is stuck and assume it can order others
to move out of its way.  A [Breadth First
Search](https://en.wikipedia.org/wiki/Breadth-first_search) (BFS) for the
first open node reveals the shortest path of moves to clear the blocked
node (i.e the 'swap path'). The process is shown below. In some cases,
robots start in their target positions, or reach them by moving out of the
way for others, this is a [fortunate
accident](https://en.wikipedia.org/wiki/Serendipity).

![Visiting Pebble Motion Algorithm Demonstration](media/masters/algo.gif)

The video below shows a simulator written in Java to develop the deadlock
resolution strategy. Square brackets represent grid positions and numbers
represent robots. Each node is the target of the robot with the same number
and the grid is numbered in row major order, meaning if every robot were in
its destination position, it would look like this:

```
[ 1] [ 2] [ 3] [ 4]
[ 5] [ 6] [ 7] [ 8]
[ 9] [10] [11] [12]
[13] [14] [15] [16]
```

The previously described process is repeated for every robot. The robot
currently in control of all moves is indicated as 'Moving pebble' and all
unfinished robots are shown in the 'Pebbles queue'. The pebbles queue
determines which robot gains control next and was populated randomly, since
the order doesn't matter. As expected, all robots eventually end up in
'Completed'.

<video width="320" height="240" controls>
  <source src="media/masters/sync/sync_serial.mp4" type="video/mp4">
</video>

## A jump forward - dealing with deadlocks... many at a time.

Simultaneous motion is achieved by assigning a priority to each robot and
defining a round consisting of two phases. In a first phase, robots
instruct each other to move along the swap path determined using BFS. A
robot sends instructions either to itself or the robot at the tip of the
swap path. Robots send instructions to themselves in two scenarios: 1) to
move to the next node, and 2) to remain in place while ordering others to
move. This ensures that high priority robots decrease their distance from
their target monotonically, which in turn ensures that progress is being
made.  Idle robots don't send instructions, but receive them.  At the end
of this phase, each robot selects the highest priority instruction and
forwards it as a node access request, maintaining the original sender's
priority.

In the second phase, **nodes** select the highest priority request to
notify successful requesters and these robots move. Then, all robots empty
their instruction queues. This ends the round and the process repeats until
all tasks are complete, at which point no one sends instructions and
everything stops.

In the video below, the same task assignments are used as before. Lower
numbers mean higher priorities, making life hard for robot 10!

<video width="320" height="240" controls>
  <source src="media/masters/sync/sync_parallel.mp4" type="video/mp4">
</video>

## Going all the way - eliminating the synchronous assumption.

So far actions were synchronized into rounds and robots/nodes knew when the
transitions occur. However, the real world is asynchronous and these
timings aren't known beforehand. Unavoidable variances mean that some
robots are faster than others and it is wasteful to synchronize node
transitions into fixed time slices. But, without a centralized
synchronization signal, robots don't know when to replan and nodes don't
know when to grant access. 

The answer to reducing this waste is to notify robots when their
instructions have failed. Nodes notify all requesters when access is
granted, and requesters react accordingly. Failed requests are relayed by
the requesters to their originators. Instructed robots don't know whether
their instruction queues already contain the highest priority instruction
they will get, so they immediately request what they have, and update
accordingly when new requests arrive by withdrawing/resubmitting requests.

Getting the programming right is **really difficult**. Instead of wading
through the gory details, let's see it in action! The grid in the video
below has tasks assigned in row major order. Robots indicate completion by
highlighting themselves in red and indicate node visits by marking them
with a purple circle. These robots have randomly initialized acceleration
and speed limits to simulate real-world variances.

<video width="320" height="240" controls>
  <source src="media/masters/async/rowmajor.mp4" type="video/mp4">
</video>

Note that any task assignment is allowed. In fact, robots can even all have
the same target node as shown below. In fact, this is an example of an
infeasible PMOG assignment. 

<video width="320" height="240" controls>
  <source src="media/masters/async/central.mp4" type="video/mp4">
</video>

So far only single tasks were assigned. Multiple tasks can be assigned by
setting the robot's priority equal to the task number. Even if one task is
delayed by many others, as overriding tasks complete, the blocked task's
priority effectively raises until it overrides every other robot. The video
below shows robots cyclically completing multiple random node visits. 

<video width="320" height="240" controls>
  <source src="media/masters/async/random_cyclic.mp4" type="video/mp4">
</video>

## A Final Test

Using the random cyclic task assignments, the figure below shows the number
of completed missions versus number of robots for one hour of simulated
time, 144 grid nodes and up to 143 robots. The legend indicates experiments for
grids with different aspect ratios.

![Completed tasks vs number of robots and
board](media/masters/tasks_completed.png)

The maximum number of completed tasks always occurs for around 30 robots
independent of the grid aspect ratio. The simulations were run headless and
no deadlocks or collisions occurred, that's 1001 flawless simulations!

# Conclusion

The relaxations on PMOG allows robots to deliver payloads without
collisions or deadlocks, in continuous asynchronous time and require
comparatively little computation. These were the extents of my research
scope and there is much room for improvement in terms of optimality.
However, I am proud of what I achieved and am glad to have made a
contribution to robotics research, however big or small.
