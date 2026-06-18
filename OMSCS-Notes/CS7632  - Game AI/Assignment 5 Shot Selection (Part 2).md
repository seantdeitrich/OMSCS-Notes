# HW5 Part 2 — Implementing `SelectThrow()`

> **Goal:** Decide *whether* to throw a dodgeball right now. Even if a throw is physically possible, there are situations where throwing is a bad idea. This method screens for those and defers the throw when it makes sense.

---

## Table of Contents
1. [The Big Picture](#the-big-picture)
2. [The Return Values Explained](#the-return-values-explained)
3. [Check 1 — Can We Even Predict a Throw?](#check-1--can-we-even-predict-a-throw)
4. [Check 2 — Is the Opponent Currently Accelerating?](#check-2--is-the-opponent-currently-accelerating)
5. [Check 3 — Will the Opponent Accelerate Before We Hit Them?](#check-3--will-the-opponent-accelerate-before-we-hit-them)
6. [Check 4 — Is the Path to Target Blocked?](#check-4--is-the-path-to-target-blocked)
7. [Complete Pseudocode](#complete-pseudocode)
8. [Key Unity API Calls](#key-unity-api-calls)
9. [Tuning Thresholds](#tuning-thresholds)
10. [Common Mistakes](#common-mistakes)
11. [Testing Strategy](#testing-strategy)

---

## The Big Picture

`PredictThrow()` answers: *"Can I hit them?"*

`SelectThrow()` answers: *"Should I throw **right now**?"*

The ballistic trajectory solver assumes the target moves at **constant velocity** (no turning, no speeding up, no slowing down). In real gameplay, that assumption breaks down constantly. If you throw when the assumption is breaking, you'll miss. So this method checks for violations of the assumption before committing.

### The Four-Gate Filter

```
Gate 1: CAN the throw reach the target at all?
          → if NO: return NoThrowTargettingFailed

Gate 2: Is the target CURRENTLY accelerating/turning?
          → if YES: return NoThrowOpponentCurrentlyAccelerating

Gate 3: WILL the target accelerate before the ball arrives?
          → if YES: return NoThrowOpponentWillAccelerate

Gate 4: Is the path PHYSICALLY BLOCKED by scenery?
          → if YES: return NoThrowOpponentOccluded

All gates passed → return DoThrow
```

Each gate is independent. Check them in order and return early as soon as one fails.

---

## The Return Values Explained

```csharp
public enum SelectThrowReturn
{
    DoThrow,                              // All checks passed. Throw it!
    NoThrowTargettingFailed,              // PredictThrow() returned false
    NoThrowOpponentCurrentlyAccelerating, // Target's velocity just changed
    NoThrowOpponentWillAccelerate,        // Target will hit a wall/turn
    NoThrowOpponentOccluded               // Ball path blocked by obstacle
}
```

---

## Check 1 — Can We Even Predict a Throw?

This is straightforward — just call `PredictThrow()` with the opponent's current data.

```
// Use the opponent's current velocity as our constant-velocity assumption
opponentVel = opponent.Vel

// Call your PredictThrow from Part 1
success = ThrowMethods.PredictThrow(
    thisMinion.HeldBallPosition,  // where the ball starts
    thisMinion.ThrowSpeed,        // max speed
    Physics.gravity,              // gravity (OK to use here! This is SelectThrow, not PredictThrow)
    opponent.Pos,                 // target start position
    opponentVel,                  // target velocity
    opponent.Forward,             // target facing direction
    maxAllowedThrowErrDist,       // allowed error
    out projectileDir,
    out projectileSpeed,
    out interceptT,
    out altT
)

if NOT success:
    return NoThrowTargettingFailed

// Also compute where we expect to hit them
interceptPos = opponent.Pos + opponent.Vel * interceptT
```

> **Note:** `Physics.gravity` IS allowed in `SelectThrow()`. It is only forbidden inside `PredictThrow()` itself.

---

## Check 2 — Is the Opponent Currently Accelerating?

### Why This Matters

Your ballistic solver assumes constant velocity. If the opponent's velocity is changing *right now*, that assumption is immediately wrong. You'd be predicting where they'll be based on a velocity that is no longer accurate.

### How to Detect It

You have access to the opponent's velocity from two consecutive frames:
- `opponent.Vel` — current velocity
- `opponent.PrevVel` — velocity last frame

Compare them. If they've changed significantly, the opponent is accelerating.

Two things to check:
1. **Speed change** — are they speeding up or slowing down?
2. **Direction change** — are they turning?

### Speed Change Check

```
speedCurrent  = opponent.Vel.magnitude
speedPrevious = opponent.PrevVel.magnitude
speedDelta    = Mathf.Abs(speedCurrent - speedPrevious)

// If speed changed by more than ~15% of max run speed, they're accelerating
SPEED_CHANGE_THRESHOLD = thisMinion.MaxPathSpeed * 0.15f

if speedDelta > SPEED_CHANGE_THRESHOLD:
    return NoThrowOpponentCurrentlyAccelerating
```

### Direction Change Check

```
// Only check direction if they're actually moving (avoid divide-by-zero)
if speedCurrent > 0.1f AND speedPrevious > 0.1f:
    angleChange = Vector3.Angle(opponent.Vel, opponent.PrevVel)
    
    ANGLE_CHANGE_THRESHOLD = 15f  // degrees

    if angleChange > ANGLE_CHANGE_THRESHOLD:
        return NoThrowOpponentCurrentlyAccelerating
```

### Special Case: Are They Practically Stopped?

If the opponent has nearly zero velocity right now, the constant-velocity assumption is satisfied (constant velocity of zero!). However, if they *just* stopped (PrevVel was significant), they may be about to start moving. You can handle this by also checking:

```
if speedCurrent < NEARLY_STOPPED_THRESHOLD:
    // They're standing still right now — this is actually fine for a throw.
    // The static-target scenario. Don't reject this case.
    // UNLESS they were just moving (they're decelerating to a stop):
    if speedPrevious > SPEED_CHANGE_THRESHOLD:
        return NoThrowOpponentCurrentlyAccelerating
```

---

## Check 3 — Will the Opponent Accelerate Before We Hit Them?

### Why This Matters

Even if the opponent is moving at constant velocity *right now*, they might be about to run into a wall, reach the edge of the navmesh, or hit a corner. When they do, they'll turn or stop — violating the constant-velocity assumption again.

### The Tool: `NavMesh.Raycast()`

`NavMesh.Raycast()` checks if a straight-line path along the navmesh is clear. Think of it like casting a ray along the ground in the opponent's movement direction. If the ray **hits** something, it means there's a navmesh boundary in the way — the opponent cannot walk straight there and will have to turn.

```csharp
// Unity signature:
bool NavMesh.Raycast(Vector3 sourcePosition, Vector3 targetPosition, 
                     out NavMeshHit hit, int areaMask)
// Returns TRUE if the ray was BLOCKED (hit something)
// Returns FALSE if the path is CLEAR
```

> **Important:** This is the opposite of `Physics.Raycast()`! NavMesh.Raycast returns `true` on a **hit** (obstacle found), and `false` if the path is **clear**.

### How to Use It

Predict where the opponent will be at `interceptT` assuming constant velocity. Cast a NavMesh ray from their current position to that predicted position.

```
predictedOpponentPos = opponent.Pos + opponent.Vel * interceptT

blocked = NavMesh.Raycast(
    opponent.Pos,           // start from opponent's current position
    predictedOpponentPos,   // try to reach predicted intercept position
    out navHit,
    opponentNavmask         // which navmesh areas count (passed in as parameter)
)

if blocked:
    // Opponent will run into a barrier before our ball gets there
    return NoThrowOpponentWillAccelerate
```

### Visualizing This (for debugging)

```csharp
// Draw the ray in the scene view — ONLY use Debug.DrawLine in SelectThrow()
Debug.DrawLine(opponent.Pos, predictedOpponentPos, Color.yellow);
```

---

## Check 4 — Is the Path to Target Blocked?

### Why This Matters

The `AdvancedMinionTestThrowScenario` scene has obstacles. If you throw a ball and it hits a wall before reaching the target, that's a wasted throw. Check that the projectile's path is clear.

### The Tool: `Physics.Raycast()`

`Physics.Raycast()` shoots a ray through 3D space and tells you if it hits any geometry.

```csharp
// Unity signature:
bool Physics.Raycast(Vector3 origin, Vector3 direction, 
                     out RaycastHit hit, float maxDistance, int layerMask)
// Returns TRUE if the ray hit something
// Returns FALSE if path is clear
```

### Setting Up the Layer Masks

The scaffold code already sets up the right masks for you:

```csharp
// Exclude NavMesh carver objects (they're not real walls)
int carverMask = ~(1 << Mgr.NavMeshCarverLayerIndex);

// Exclude minions (we WANT to hit the opponent, so ignore all minions)
int minionMask = ~(1 << Mgr.MinionTeamBLayerIndex) & ~(1 << Mgr.MinionTeamALayerIndex);

// Exclude dodgeballs themselves
int ballMask = ~(1 << Mgr.BallTeamALayerIndex) & ~(1 << Mgr.BallTeamBLayerIndex);

// Combine: we want to hit real walls, but ignore minions and balls
int mask = Physics.AllLayers & carverMask & ballMask & minionMask;
```

### Casting the Ray

Cast from the ball's launch position toward the predicted intercept position:

```
launchPos    = thisMinion.HeldBallPosition
targetPoint  = interceptPos                      // where the ball should land
direction    = (targetPoint - launchPos).normalized
distance     = Vector3.Distance(launchPos, targetPoint)

hit = Physics.Raycast(launchPos, direction, out hitInfo, distance, mask)

if hit:
    return NoThrowOpponentOccluded
```

### The "Two Parallel Rays" Tip

The assignment hints that casting **two parallel rays** spaced one ball-width apart gives better results. This catches cases where the center-line is clear but the ball itself would clip a corner.

```
// Offset perpendicular to the throw direction (horizontal)
perpendicular = Vector3.Cross(direction, Vector3.up).normalized
ballRadius    = 0.5f  // approximate; check actual dodgeball radius if known
offset        = perpendicular * ballRadius

hit1 = Physics.Raycast(launchPos + offset, direction, out hitInfo1, distance, mask)
hit2 = Physics.Raycast(launchPos - offset, direction, out hitInfo2, distance, mask)

if hit1 OR hit2:
    return NoThrowOpponentOccluded
```

---

## Complete Pseudocode

```
SelectThrowReturn SelectThrow(thisMinion, opponent, opponentNavmask,
                               maxAllowedThrowErrDist, deltaT,
                               out projectileDir, out projectileSpeed,
                               out interceptT, out interceptPos):

    Mgr = PrisonDodgeballManager.Instance

    // ----------------------------------------------------------------
    // GATE 1: Can we predict a valid throw at all?
    // ----------------------------------------------------------------
    opponentVel = opponent.Vel

    success = ThrowMethods.PredictThrow(
        thisMinion.HeldBallPosition,
        thisMinion.ThrowSpeed,
        Physics.gravity,
        opponent.Pos,
        opponentVel,
        opponent.Forward,
        maxAllowedThrowErrDist,
        out projectileDir,
        out projectileSpeed,
        out interceptT,
        out altT
    )

    if NOT success:
        interceptPos = opponent.Pos
        return NoThrowTargettingFailed

    interceptPos = opponent.Pos + opponent.Vel * interceptT


    // ----------------------------------------------------------------
    // GATE 2: Is opponent currently accelerating?
    // ----------------------------------------------------------------
    speedCurrent  = opponent.Vel.magnitude
    speedPrevious = opponent.PrevVel.magnitude

    SPEED_THRESHOLD = thisMinion.MaxPathSpeed * 0.15f
    ANGLE_THRESHOLD = 15f  // degrees

    if Mathf.Abs(speedCurrent - speedPrevious) > SPEED_THRESHOLD:
        return NoThrowOpponentCurrentlyAccelerating

    if speedCurrent > 0.1f AND speedPrevious > 0.1f:
        if Vector3.Angle(opponent.Vel, opponent.PrevVel) > ANGLE_THRESHOLD:
            return NoThrowOpponentCurrentlyAccelerating


    // ----------------------------------------------------------------
    // GATE 3: Will opponent accelerate (hit navmesh boundary) before intercept?
    // ----------------------------------------------------------------
    predictedPos = opponent.Pos + opponent.Vel * interceptT

    navBlocked = NavMesh.Raycast(
        opponent.Pos,
        predictedPos,
        out navHit,
        opponentNavmask
    )

    if navBlocked:
        return NoThrowOpponentWillAccelerate


    // ----------------------------------------------------------------
    // GATE 4: Is the projectile path blocked by obstacles?
    // ----------------------------------------------------------------
    carverMask = ~(1 << Mgr.NavMeshCarverLayerIndex)
    minionMask = ~(1 << Mgr.MinionTeamBLayerIndex) & ~(1 << Mgr.MinionTeamALayerIndex)
    ballMask   = ~(1 << Mgr.BallTeamALayerIndex) & ~(1 << Mgr.BallTeamBLayerIndex)
    mask       = Physics.AllLayers & carverMask & ballMask & minionMask

    launchPos   = thisMinion.HeldBallPosition
    direction   = (interceptPos - launchPos).normalized
    distance    = Vector3.Distance(launchPos, interceptPos)
    ballRadius  = 0.15f  // adjust as needed

    perpendicular = Vector3.Cross(direction, Vector3.up).normalized

    hit1 = Physics.Raycast(launchPos + perpendicular * ballRadius, direction, 
                            out hitInfo1, distance, mask)
    hit2 = Physics.Raycast(launchPos - perpendicular * ballRadius, direction, 
                            out hitInfo2, distance, mask)

    if hit1 OR hit2:
        return NoThrowOpponentOccluded


    // ----------------------------------------------------------------
    // All gates passed!
    // ----------------------------------------------------------------
    return DoThrow
```

---

## Key Unity API Calls

### `NavMesh.Raycast()`
```csharp
using UnityEngine.AI;

NavMeshHit navHit;
bool blocked = NavMesh.Raycast(startPos, endPos, out navHit, areaMask);
// blocked == true  → path has a navmesh boundary in the way (will turn/stop)
// blocked == false → opponent can walk straight to endPos
```

### `Physics.Raycast()`
```csharp
RaycastHit hitInfo;
bool hit = Physics.Raycast(origin, direction, out hitInfo, maxDist, layerMask);
// hit == true  → something is in the way
// hit == false → path is clear
```

### `Vector3.Angle()`
```csharp
float angle = Vector3.Angle(vecA, vecB);
// Returns angle in DEGREES (0–180)
// Works even with unnormalized vectors
```

### `Debug.DrawLine()` (only in `SelectThrow()`)
```csharp
Debug.DrawLine(startPos, endPos, Color.red);    // visible in Scene view during Play mode
Debug.DrawRay(startPos, direction * dist, Color.green);
```

---

## Tuning Thresholds

The thresholds below are starting points. Tune them by running the `AdvancedMinionTestThrowScenario` scene.

| Threshold | Purpose | Starting Value |
|---|---|---|
| `SPEED_THRESHOLD` | Max speed change before "accelerating" | `MaxPathSpeed * 0.15f` |
| `ANGLE_THRESHOLD` | Max direction change before "turning" | `15f` degrees |
| `ballRadius` | Width of parallel raycast offset | `0.15f` units |

**Too strict** (thresholds too small): You'll never throw — the AI will constantly defer.

**Too loose** (thresholds too large): You'll throw even when the opponent is turning, leading to misses.

---

## Common Mistakes

### 1. Confusing `NavMesh.Raycast()` return value
`NavMesh.Raycast()` returns `true` when the path is **blocked** (obstacle hit). This is the *opposite* of what you might expect. A `true` return means "bad, don't throw."

### 2. Using `Physics.gravity` inside `PredictThrow()` instead of `SelectThrow()`
`Physics.gravity` is only allowed in `SelectThrow()`. Inside `PredictThrow()`, gravity comes in as the `projectileGravity` parameter. Mixing these up will cause EditorMode tests to fail.

### 3. Not initializing output parameters on early returns
If you return early (e.g., `NoThrowTargettingFailed`), C# requires that all `out` parameters are set before returning. Make sure you assign `projectileDir`, `projectileSpeed`, `interceptT`, and `interceptPos` before every return path, even if the values are dummy zeros.

### 4. The navmask parameter
`opponentNavmask` is passed into `SelectThrow()` for you. Pass it directly into `NavMesh.Raycast()` as the `areaMask`. Don't hard-code it as `NavMesh.AllAreas` — the test scenarios use specific area masks.

### 5. Casting the physics ray along the wrong path
Your ray should follow the *ball's trajectory*, not just a straight horizontal line. The ball will arc due to gravity. For a quick approximation, casting a straight-line ray toward `interceptPos` (the expected landing point) is acceptable and what the assignment expects. A perfectly accurate check would require sampling the arc, but that's overkill here.

### 6. Forgetting to handle the case where `opponent.Vel` is near zero
If the opponent is standing still, direction-change checks will divide by near-zero. Guard against this:
```csharp
if (opponent.Vel.magnitude > 0.1f && opponent.PrevVel.magnitude > 0.1f)
{
    // Safe to check angle between them
}
```

---

## Testing Strategy

### Step 1: Test Gate 1 Only
Comment out Gates 2–4. Verify that throws happen at all and the ball goes toward the target.

### Step 2: Enable Gate 2
Watch the AdvancedMinionTestThrowScenario. The minion should stop throwing when the opponent starts turning. If it never throws, your thresholds are too strict.

### Step 3: Enable Gate 3
Add a wall near the opponent's path. The minion should stop throwing when the opponent is about to run into it.

### Step 4: Enable Gate 4
Open AdvancedMinionTestThrowScenario which has obstacles. The minion should not throw through walls.

### Step 5: Run the PlayMode Test
**Window → General → Test Runner → PlayMode**

The `DodgeballTests` test will run the full scenario and report a score. The scoring mirrors the autograder.

---

## Quick Reference Diagram

```
SelectThrow() flow:

  Call PredictThrow()
       │
       ├── returns false → NoThrowTargettingFailed
       │
       └── returns true
               │
               ↓
    Check speed/angle delta (Vel vs PrevVel)
               │
               ├── big change → NoThrowOpponentCurrentlyAccelerating
               │
               └── small change
                       │
                       ↓
         NavMesh.Raycast(current pos → predicted pos)
                       │
                       ├── blocked → NoThrowOpponentWillAccelerate
                       │
                       └── clear
                               │
                               ↓
              Physics.Raycast(launch pos → intercept pos)
                               │
                               ├── hit → NoThrowOpponentOccluded
                               │
                               └── clear
                                       │
                                       ↓
                                   DoThrow ✓
```

---

*See [[Assignment 5 Ballistic Trajectory (Part 1)]] for the `PredictThrow()` implementation.*
