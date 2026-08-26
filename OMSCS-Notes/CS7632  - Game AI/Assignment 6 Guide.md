# Step-by-Step Guide: Optimizing the Prison Dodgeball Finite State Machine (FSM)

This guide outlines a highly efficient strategy to complete Homework 6 by identifying the structural flaws in the provided demo FSM and implementing a high-performance solution with minimal changes (approximately 4 lines added and 2 lines deleted), as suggested by course staff.

---

## 1. Architectural Analysis & The "Sitting Duck" Flaw

The baseline implementation struggles significantly when scaled to $M$ minions and $N$ balls ($5 \geq M \geq N$) due to two primary bottlenecks:

1. **Forced Linear Progression (`GoToThrowSpotState`):** When a minion collects a ball, it enters `GoToThrowSpotState` and handles movement sequentially:
   ```csharp
   Minion.GoTo(Mgr.TeamAdvance(Team).position);
   ```
   It waits until `Minion.ReachedTarget()` evaluates to `true` before it even considers throwing or rescuing. This makes its pathing 100% predictable, allowing opponent AI ("Glass Joe") to easily score hits during the transition.
2. **The Standing Still Vulnerability:** If a minion transitions to `ThrowBallState` but `ShotSelection.SelectThrow` does not return `DoThrow` immediately (due to targeting windows or timing), the minion executes `Minion.FaceTowardsForThrow(intercept); return null;`. Returning `null` keeps the minion in `ThrowBallState` without updating its movement path, causing it to stand completely still and become a "sitting duck."

---

## 2. Strategy: Continuous Mobility & Instant Triggering

To reliably beat Glass Joe and undisclosed opponents at least **2/3rds of the time**, your minions must throw on the run and maintain defensive positioning while searching for shots. 

### The Tweak Plan
* **Bypass `GoToThrowSpotState` entirely** to eliminate predictable forward charges.
* **Inject high-mobility tracking** into the active throwing/targeting phase so minions never stand still.

---

## 3. Step-by-Step Implementation

### Step 1: Redirect Transitions to Bypass `GoToThrowSpotState`
Open `MinionStateMachine.cs` and locate the transitions inside `CollectBallState` and `DefensiveDemoState`. We will remove the bottleneck state and route directly into `ThrowBallState`.

1. In **`CollectBallState.Update()`**, locate the line:
   ```csharp
   // DELETE this line:
   return ParentFSM.CreateStateTransition(GoToThrowSpotStateName);
   ```
   Replace it with:
   ```csharp
   // ADD this line:
   return ParentFSM.CreateStateTransition(ThrowBallStateName);
   ```

2. In **`DefensiveDemoState.Update()`**, locate the line:
   ```csharp
   // DELETE this line:
   return ParentFSM.CreateStateTransition(GoToThrowSpotStateName);
   ```
   Replace it with:
   ```csharp
   // ADD this line:
   return ParentFSM.CreateStateTransition(ThrowBallStateName);
   ```

### Step 2: Keep Minions Moving During the Throw Phase
To ensure your minions do not stand still while calculating trajectories or waiting for a clean shot alignment, add active navigation to `ThrowBallState`.

1. In **`ThrowBallState.Enter()`**, add a navigation command to keep the agent in motion toward a safe zone or defensive posture:
   ```csharp
   // ADD this line inside Enter():
   Minion.GoTo(Mgr.TeamHome(Team).position); 
   ```
   *Alternative high-performance variant:* You can also add this to `ThrowBallState.Update()` to dynamically reposition, or use `Mgr.TeamAdvance(Team).position` if you want an aggressive push while throwing on the move.

### Step 3: (Optional Optimization) Handle Rescue Triggers
If you want to maintain the rescue logic originally found in `GoToThrowSpotState`, you can add a single check inside `ThrowBallState.Update()` before executing the shot selection:
```csharp
// ADD these lines inside ThrowBallState.Update() if rescuing is prioritized:
if (FindRescuableTeammate(out var m)) 
    return ParentFSM.CreateStateTransition<MinionScript>(RescueStateName, m, true);
```

---

## 4. Testing Protocols ($5 \geq M \geq N$)

To validate your changes against the homework's grading criteria, use the built-in Unity Test Runner:

1. **Open the Test Runner:** Go to `Window > General > Test Runner`.
2. **Select PlayMode Tab:** Ensure you are evaluating the integration suite under `PlayMode`.
3. **Configure the Match Scenarios:**
   * Test with extreme configurations to satisfy the constraint $5 \geq M \geq N$.
   * Focus on **5 Minions vs 1 Ball** (high competition for resources, tests pathing efficiency).
   * Focus on **5 Minions vs 5 Balls** (high fire rate, tests evasion and target selection under pressure).
4. **Win-Rate Verification:** Run at least 20-30 automated matches back-to-back to verify that your win rate against "Glass Joe" is consistently $\geq 66.7\%$.

---

## 5. Pre-Submission Checklist

* [ ] **Remove Debugging Code:** Ensure all `Debug.Log` statements added during testing are completely removed to prevent performance drops or autograder failures.
* [ ] **Verify Namespaces:** Do not use `UnityEditor` anywhere in your state classes, as the tournament and grading suites run from compiled standalone builds.
* [ ] **File Completeness:** Ensure your submission bundle includes `MinionStateMachine.cs`, `ThrowMethods.cs`, and `ShotSelection.cs`.
