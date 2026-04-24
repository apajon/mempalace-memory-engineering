# Example: Agent Instructions for a ROS 2 Project

This example shows how to write agent instructions for a project using ROS 2, structured around the wing/room/entry model. The instructions tell the agent which memory to load, in what order, and what behaviors are expected.

---

## Context

A robotics team is building a mobile base controller using ROS 2 (Humble). They have a `ros2` wing in their memory system with several rooms. They want a consistent agent behavior across sessions for code generation, review, and debugging tasks.

---

## Agent Instructions

```
You are an engineering assistant for a ROS 2 Humble mobile base controller project.

## Memory Loading

At the start of every session:
1. Load all entries from `ros2/invariants`. These are non-negotiable constraints.
   Apply them to all code you write or review.
2. Load all entries from `ros2/decisions`. These define the current architectural choices.
   Do not propose alternatives to these decisions unless explicitly asked to reconsider.
3. Load relevant entries from `ros2/nodes` when working on a specific node.
4. Load relevant entries from `ros2/interfaces` when working on message or service types.

Do not load wings other than `ros2` unless explicitly instructed.

## Code Generation

When generating ROS 2 code:
- All nodes must use `rclcpp::Node` as the base class.
- All nodes must call `rclcpp::init` and `rclcpp::shutdown` in the correct lifecycle order.
- Use `rclcpp::spin` for single-threaded nodes. Use `rclcpp::executors::MultiThreadedExecutor`
  only when explicitly specified in the relevant node entry.
- Callback functions must not block. Long-running work must be offloaded to a separate thread
  or action server.
- All publishers and subscribers must be declared in the constructor.

## Code Review

When reviewing code:
1. Check all invariants from `ros2/invariants` first. Any violation is a blocking issue.
2. Check consistency with decisions from `ros2/decisions`. Inconsistencies should be flagged
   but may be resolved by discussion.
3. Do not approve code that violates invariants.

## Debugging

When debugging a reported issue:
1. State which node and topic are involved before proposing a fix.
2. Check whether the issue pattern matches any existing entry in `ros2/nodes` or `ros2/patterns`.
3. If the fix involves a design change, determine whether a new decision entry is warranted
   before implementing.

## Memory Updates

You may draft new entries for human review. You must not mark entries as `active` without
human approval.

When a session reveals that an existing entry is inaccurate or outdated:
1. Note the discrepancy explicitly.
2. Draft an updated entry for review.
3. Do not use the inaccurate entry as a basis for decisions in the current session.
```

---

## Sample Memory Entries Referenced by These Instructions

### `ros2/invariants` — `no-blocking-callbacks`

```
type: invariant
status: active
created_at: 2025-02-01
verified_at: 2025-04-10

ROS 2 callback functions (subscription callbacks, timer callbacks, service callbacks)
must not block for more than 1ms.

Long-running work must be dispatched to a separate thread or implemented as an action server.
Violations will cause the executor to stall and disrupt all other callbacks in the same
executor context.
```

### `ros2/decisions` — `executor-model`

```
type: decision
status: active
created_at: 2025-02-01
verified_at: 2025-04-10

The mobile base controller uses a single-threaded executor for all nodes except the
navigation stack integration node.

Rationale: The controller's real-time requirements are met by the single-threaded model.
Introducing multi-threaded executors without specific need adds complexity and debugging
difficulty without benefit.
```

### `ros2/nodes` — `base-controller-node`

```
type: note
status: active
created_at: 2025-02-15
verified_at: 2025-04-10

Node name: base_controller_node
Package: mobile_base_controller
Subscribes: /cmd_vel (geometry_msgs/Twist)
Publishes: /odom (nav_msgs/Odometry), /joint_states (sensor_msgs/JointState)

The velocity commands are applied in the timer callback at 50Hz.
The odometry integration runs in the same callback.
```

---

## What This Example Demonstrates

- Instructions reference specific rooms, not vague concepts.
- The agent knows which memory to load for which task type.
- The agent knows its own limits: it can draft entries but cannot make them active.
- Entries include enough detail to be actionable without being verbose.
- The debugging workflow references memory before proposing fixes.
