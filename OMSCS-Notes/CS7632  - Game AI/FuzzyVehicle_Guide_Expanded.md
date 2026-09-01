# FuzzyVehicle Assignment — Expanded Guide

## What You're Building

You are filling in `Assets/Scripts/GameAIStudentWork/FuzzyVehicle.cs` to control a racecar
using fuzzy logic. Every frame, your code must:

1. Read sensor values from the car and track.
2. Run those values through fuzzy sets to get degrees of membership.
3. Fire fuzzy rules that map inputs → throttle/steering outputs.
4. Call `ApplyFuzzyRules()` — the only allowed way to set throttle and steering.

**You must not** directly assign `Throttle` or `Steering`, and must not leave
`HardCodeSteering()` or `HardCodeThrottle()` calls in the final submission.

---

## File Structure at a Glance

`FuzzyVehicle.cs` is already structured into four regions. Here is where each step of this guide maps to:

```
FuzzyVehicle : AIVehicle
│
├── [CLASS BODY — top of class]
│     Step 2: enum declarations
│     Step 2: member variable declarations (FuzzySet, FuzzyRuleSet, FuzzyValueSet)
│
├── GetSpeedSet()          ← Step 3
├── GetSteerAngleSet()     ← Step 3  (you add this method)
├── GetThrottleSet()       ← Step 3
├── GetWheelSet()          ← Step 3
│
├── GetThrottleRuleSet()   ← Step 4
├── GetWheelRuleSet()      ← Step 4
│
├── Awake()                ← Set StudentName only. Do NOT init fuzzy stuff here.
├── Start()                ← Step 5: initialize all sets and rule sets
│
└── Update()               ← Step 6: compute crisp inputs, fuzzify, ApplyFuzzyRules()
```

---

## The Big Picture Loop

```
Each frame (Update):
  1. Compute crisp float inputs  (speed, steering angle, etc.)
  2. Fuzzify inputs              (.Evaluate() on each input FuzzySet)
  3. ApplyFuzzyRules()           (framework evaluates rules → crisp throttle & steering)
```

There are only two phases to implement:
- **Setup** — run once in `Start()`, build all your sets and rule sets.
- **Runtime** — run every `Update()`, read inputs, fuzzify, apply rules.

---

## Step 1 — Understand Your Available Inputs

You need at least two crisp float inputs. The three most useful ones are:

| Input | How to get it | Useful for |
|---|---|---|
| **Speed** | `Speed` (built-in property, m/s) | Throttle control |
| **Steering angle** | `Vector3.SignedAngle` between car forward and look-ahead track direction | Primary steering |
| **Road offset** | Signed distance from car to track centerline | Steering correction / recovery |

### Computing steering angle

The steering angle tells you: *how much do I need to turn to stay on the road?*
Positive = target is to the right, negative = to the left.

```csharp
// In Update(), before fuzzification:
Vector3 lookAheadDir = Racetrack.GetDirectionAhead(15f);
lookAheadDir.y = 0f;  // flatten to XZ — the track lies in the XZ plane

// transform.forward is the car's current heading
float steerAngle = Vector3.SignedAngle(transform.forward, lookAheadDir, Vector3.up);
// ^^^ Vector3.up as axis is REQUIRED for correct sign convention
```

> **Why look-ahead instead of closest point?** Looking ahead smooths out noise and
> gives the car time to begin turning before it reaches a curve.

### Computing road offset

The road offset tells you: *how far am I from the center of the road?*
Use this as a secondary steering input to pull the car back toward center.

```csharp
// In Update():
Vector3 carPosFlat   = new Vector3(transform.position.x, 0f, transform.position.z);
Vector3 closestFlat  = new Vector3(Racetrack.ClosestPointOnPath.x, 0f, Racetrack.ClosestPointOnPath.z);
Vector3 trackDir     = Racetrack.ClosestPointDirectionOnPath;
trackDir.y = 0f;

Vector3 trackRight = Vector3.Cross(Vector3.up, trackDir).normalized;
float roadOffset   = Vector3.Dot(carPosFlat - closestFlat, trackRight);
// Positive = car is to the right of centerline, negative = to the left
```

---

## Step 2 — Declare Enums and Member Variables

> **Where:** In the class body, directly below the class declaration, before any methods.
> This is already partially done in the starter code — expand it.

### Enums

Each enum represents one fuzzy variable. Each member of the enum is a linguistic label
(e.g., "Slow", "Fast"). You must have **at least 3 labels on every output enum**.

```csharp
// --- INPUT ENUMS ---
enum FzInputSpeed      { Slow, Medium, Fast }
enum FzInputSteerAngle { HardLeft, Left, Straight, Right, HardRight }
// Optional: add more inputs for road offset, curvature, etc.

// --- OUTPUT ENUMS --- (minimum 3 labels each, required by grader)
enum FzOutputThrottle  { Brake, Coast, Accelerate }
enum FzOutputWheel     { TurnLeft, GoStraight, TurnRight }
```

> **Rule:** Every label in every input enum **must** appear as an antecedent (`If(...)`)
> in **every** rule set you define. Missing any label will cause NaN output and the car
> will behave erratically or spin in place.

### Member Variables

Declare one field per fuzzy set and rule set, plus `FuzzyValueSet` instances used at runtime.
Add these in the class body alongside the existing starter fields:

```csharp
// Input sets (one per input variable)
FuzzySet<FzInputSpeed>      fzSpeedSet;
FuzzySet<FzInputSteerAngle> fzSteerAngleSet;

// Output sets (one per output variable)
FuzzySet<FzOutputThrottle>  fzThrottleSet;
FuzzySet<FzOutputWheel>     fzWheelSet;

// Rule sets (one per output variable)
FuzzyRuleSet<FzOutputThrottle> fzThrottleRuleSet;
FuzzyRuleSet<FzOutputWheel>    fzWheelRuleSet;

// Runtime value containers — declare here, reused every frame (avoids GC pressure)
FuzzyValueSet fzInputValueSet = new FuzzyValueSet();
FuzzyValueSet mergedThrottle  = new FuzzyValueSet();
FuzzyValueSet mergedWheel     = new FuzzyValueSet();
```

> `FuzzyValueSet` acts as a dictionary that holds the current degree-of-membership
> for every input label. All `.Evaluate()` calls write into the same set; all rule
> evaluation reads from the same set.

---

## Step 3 — Build Fuzzy Sets

> **Where:** In the private `GetXxxSet()` helper methods. Replace the placeholder
> `set = new FuzzySet<T>();` lines with actual `GenerateCrossfadeFuzzySet` /
> `GenerateDiscreteFuzzySet` calls.

### Input sets — `GenerateCrossfadeFuzzySet<T>()`

Use this for inputs. It generates smooth, overlapping membership functions so that
a crisp value like 17 m/s can partially belong to both `Medium` (0.8) and `Fast` (0.2).

**Argument format:**
- **Shoulder** (outermost labels): pass a `(float, float)` tuple — the flat plateau range.
- **Triangle** (inner labels): pass a single `float` — the peak point.
- Arguments are listed in enum order (left to right).

```csharp
// In GetSpeedSet() — replace the placeholder:
private FuzzySet<FzInputSpeed> GetSpeedSet()
{
    // FzInputSpeed has 3 labels: Slow, Medium, Fast
    // Slow  : full membership 0–10 m/s, fades out toward 18
    // Medium: peaks at 18 m/s
    // Fast  : fades in from 25 m/s, full membership beyond 40
    return GenerateCrossfadeFuzzySet<FzInputSpeed>(
        (0f, 10f),    // Slow   — left shoulder: flat from 0 to 10
        18f,           // Medium — triangle peak at 18
        (25f, 40f)    // Fast   — right shoulder: flat from 25 to 40+
    );
}
```

```csharp
// Add a new method GetSteerAngleSet() to the class:
private FuzzySet<FzInputSteerAngle> GetSteerAngleSet()
{
    // FzInputSteerAngle has 5 labels: HardLeft, Left, Straight, Right, HardRight
    // Angles in degrees. Negative = turn left needed, positive = turn right needed.
    return GenerateCrossfadeFuzzySet<FzInputSteerAngle>(
        (-60f, -30f),  // HardLeft   — left shoulder
        -12f,           // Left       — triangle peak
        0f,             // Straight   — triangle peak
        12f,            // Right      — triangle peak
        (30f, 60f)     // HardRight  — right shoulder
    );
}
```

> **Tip:** Verify your shapes in the Unity console. After assigning in `Start()`, call:
> `Debug.Log(RenderFuzzySetAscii(fzSpeedSet));`
> You should see overlapping trapezoid/triangle shapes across a meaningful value range.

### Output sets — `GenerateDiscreteFuzzySet<T>()`

Use this for outputs. Each label gets exactly one crisp representative value.
These crisp values are what ultimately get assigned to `Throttle` and `Steering`
after defuzzification.

```csharp
// In GetThrottleSet() — replace the placeholder:
private FuzzySet<FzOutputThrottle> GetThrottleSet()
{
    // Throttle range is [-1, 1]:  -1 = full brake, 0 = coast, 1 = full throttle
    return GenerateDiscreteFuzzySet<FzOutputThrottle>(-1f, 0f, 1f);
    //                                                 ^     ^   ^
    //                                              Brake Coast Accelerate
}
```

```csharp
// In GetWheelSet() — replace the placeholder:
private FuzzySet<FzOutputWheel> GetWheelSet()
{
    // Steering range is [-1, 1]:  -1 = full left, 0 = straight, 1 = full right
    return GenerateDiscreteFuzzySet<FzOutputWheel>(-1f, 0f, 1f);
    //                                              ^     ^   ^
    //                                          TurnLeft  |  TurnRight
    //                                                 GoStraight
}
```

---

## Step 4 — Write Fuzzy Rules

> **Where:** In the `GetThrottleRuleSet()` and `GetWheelRuleSet()` methods. Replace or
> expand the placeholder `rules` arrays.

Rules map input labels to output labels using `If(...).Then(...)`. The framework
combines all firing rules via weighted average (centroid defuzzification).

**Rule coverage requirement:** Every label of every input enum that you call
`.Evaluate()` on must appear as an antecedent (`If(...)`) in every rule set.
Missing a label → NaN → car spins or goes full throttle forever.

### Throttle rules

```csharp
private FuzzyRuleSet<FzOutputThrottle> GetThrottleRuleSet(FuzzySet<FzOutputThrottle> throttle)
{
    FuzzyRule<FzOutputThrottle>[] rules =
    {
        // Speed-based rules: go fast when slow, brake when fast
        If(FzInputSpeed.Slow)  .Then(FzOutputThrottle.Accelerate),
        If(FzInputSpeed.Medium).Then(FzOutputThrottle.Coast),
        If(FzInputSpeed.Fast)  .Then(FzOutputThrottle.Brake),

        // Steering-based rules: slow down for turns
        // Every FzInputSteerAngle label MUST appear here (5 labels, 5 rules minimum)
        If(FzInputSteerAngle.HardLeft) .Then(FzOutputThrottle.Brake),
        If(FzInputSteerAngle.Left)     .Then(FzOutputThrottle.Coast),
        If(FzInputSteerAngle.Straight) .Then(FzOutputThrottle.Accelerate),
        If(FzInputSteerAngle.Right)    .Then(FzOutputThrottle.Coast),
        If(FzInputSteerAngle.HardRight).Then(FzOutputThrottle.Brake),
    };
    return new FuzzyRuleSet<FzOutputThrottle>(throttle, rules);
}
```

### Steering rules

```csharp
private FuzzyRuleSet<FzOutputWheel> GetWheelRuleSet(FuzzySet<FzOutputWheel> wheel)
{
    FuzzyRule<FzOutputWheel>[] rules =
    {
        // Every FzInputSteerAngle label MUST appear here
        If(FzInputSteerAngle.HardLeft) .Then(FzOutputWheel.TurnLeft),
        If(FzInputSteerAngle.Left)     .Then(FzOutputWheel.TurnLeft),
        If(FzInputSteerAngle.Straight) .Then(FzOutputWheel.GoStraight),
        If(FzInputSteerAngle.Right)    .Then(FzOutputWheel.TurnRight),
        If(FzInputSteerAngle.HardRight).Then(FzOutputWheel.TurnRight),

        // Every FzInputSpeed label MUST appear here too (even if not steering-relevant)
        If(FzInputSpeed.Slow)  .Then(FzOutputWheel.GoStraight),
        If(FzInputSpeed.Medium).Then(FzOutputWheel.GoStraight),
        If(FzInputSpeed.Fast)  .Then(FzOutputWheel.GoStraight),
    };
    return new FuzzyRuleSet<FzOutputWheel>(wheel, rules);
}
```

> **Advanced syntax:** You can combine conditions with `And()` and negate with `Not()`:
> ```csharp
> If(And(FzInputSpeed.Fast, FzInputSteerAngle.HardLeft)).Then(FzOutputThrottle.Brake),
> If(Not(FzInputSpeed.Slow)).Then(FzOutputThrottle.Coast),
> ```

---

## Step 5 — Initialize in `Start()`

> **Where:** Inside `protected override void Start()`. The starter code already calls
> `base.Start()` and initializes the speed set — add the rest after it.
>
> **Never** initialize fuzzy sets in `Awake()`. The `Awake()` comment in the starter
> code explicitly forbids it.

```csharp
protected override void Start()
{
    base.Start();  // MUST be first

    // Build input sets
    fzSpeedSet      = GetSpeedSet();
    fzSteerAngleSet = GetSteerAngleSet();  // new method you added

    // Build output sets
    fzThrottleSet = GetThrottleSet();
    fzWheelSet    = GetWheelSet();

    // Build rule sets — pass the corresponding output set into each
    fzThrottleRuleSet = GetThrottleRuleSet(fzThrottleSet);
    fzWheelRuleSet    = GetWheelRuleSet(fzWheelSet);

    // Verify your fuzzy set shapes in the Unity console (optional but recommended)
    Debug.Log(RenderFuzzySetAscii(fzSpeedSet));
    Debug.Log(RenderFuzzySetAscii(fzSteerAngleSet));
    Debug.Log(RenderFuzzySetAscii(fzThrottleSet));
    Debug.Log(RenderFuzzySetAscii(fzWheelSet));
}
```

---

## Step 6 — Fuzzify and Apply in `Update()`

> **Where:** Inside `override protected void Update()`. This is where the runtime
> loop lives. Replace the starter `HardCodeSteering` / `HardCodeThrottle` calls.
>
> **Critical order:** `base.Update()` must be the **last** call in `Update()`. It
> processes the throttle and steering values set by `ApplyFuzzyRules()`.

```csharp
override protected void Update()
{
    // Guard: wait for the track to be ready before doing anything
    if (!Racetrack.IsInitialized)
    {
        base.Update();
        return;
    }

    // ── 1. Compute crisp inputs ─────────────────────────────────────────────

    float speed = Speed;  // built-in property, in m/s

    // Look-ahead direction: get the track direction 15m ahead, flattened to XZ
    Vector3 lookAheadDir = Racetrack.GetDirectionAhead(15f);
    lookAheadDir.y = 0f;

    // Signed angle from car's heading to the look-ahead direction
    // Positive = target is to the right, negative = to the left
    float steerAngle = Vector3.SignedAngle(transform.forward, lookAheadDir, Vector3.up);

    // Optional: visualize the look-ahead vector in Scene view
    // Debug.DrawRay(transform.position, lookAheadDir * 15f, Color.green);

    // ── 2. Fuzzify ─────────────────────────────────────────────────────────
    // Each .Evaluate() call writes degree-of-membership values into fzInputValueSet.
    // The same FuzzyValueSet is shared across all inputs.
    fzSpeedSet.Evaluate(speed,           fzInputValueSet);
    fzSteerAngleSet.Evaluate(steerAngle, fzInputValueSet);

    // ── 3. Apply rules ─────────────────────────────────────────────────────
    // This evaluates all rules, defuzzifies, and assigns Throttle + Steering.
    // The out/ref parameters carry intermediate state back for debug printing.
    ApplyFuzzyRules<FzOutputThrottle, FzOutputWheel>(
        fzThrottleRuleSet,
        fzWheelRuleSet,
        fzInputValueSet,
        out var throttleRuleOutput,
        out var wheelRuleOutput,
        ref mergedThrottle,
        ref mergedWheel
    );

    // ── 4. Debug HUD (optional — comment out before submission) ────────────
    if (vizText != null)
    {
        strBldr.Clear();
        strBldr.AppendLine($"speed: {speed:F1} m/s   angle: {steerAngle:F1}°");

        // Print degree-of-membership for each input variable
        DiagnosticPrintFuzzyValueSet<FzInputSpeed>(fzInputValueSet, strBldr);
        DiagnosticPrintFuzzyValueSet<FzInputSteerAngle>(fzInputValueSet, strBldr);

        // Print which output rules fired and their weights
        DiagnosticPrintRuleSet<FzOutputThrottle>(fzThrottleRuleSet, throttleRuleOutput, strBldr);
        DiagnosticPrintRuleSet<FzOutputWheel>(fzWheelRuleSet, wheelRuleOutput, strBldr);

        vizText.text = strBldr.ToString();
    }

    base.Update();  // MUST be last — processes Throttle and Steering
}
```

> **Remove** both `HardCodeSteering(0f)` and `HardCodeThrottle(0.5f)` before
> submitting. The autograder will fail if either call remains. You can comment one
> out while still developing the other if needed.

---

## Step 7 — Test in Unity

1. Open the scene `Assets/Scenes/RaceTrackFZ` — this is the curviest track and
   the hardest test. If it works here, the others should too.
2. Press **Play**. The car should immediately start driving. If it sits still,
   check that your rule sets are not empty and `ApplyFuzzyRules` is being called.
3. Watch the HUD (if `vizText` is assigned in the Inspector) to see live
   membership values and which rules are firing each frame.
4. Use the **Scene** view (while in Play mode) to see `Debug.DrawRay` lines.

### Four test scenes (run all before submitting)

| Scene | What it tests |
|---|---|
| `RaceTrackFZ` | Tight curves — primary steering test |
| `WindingRaceTrackFZ` | Sustained curves — look-ahead tuning |
| `FastSweepersRaceTrackFZ` | High-speed sweepers — throttle/brake balance |
| `DragRaceFZ` | Straight line — throttle-only, should go fast |

---

## Step 8 — Troubleshoot Common Problems

| Symptom | Likely cause | Fix |
|---|---|---|
| Car drives straight off track | Steering rules empty or not evaluating | Check that `fzSteerAngleSet.Evaluate(...)` is called and `GetWheelRuleSet` returns real rules |
| Car turns the wrong direction | Sign of `steerAngle` is flipped | Confirm `Vector3.up` is the axis in `SignedAngle`; try flipping the angle sign |
| NaN / Infinity errors | An input enum label has no rule in one of the rule sets | Add a rule for every label of every input enum in every rule set |
| Car oscillates side to side | Look-ahead too short, over-correcting | Increase look-ahead distance: try 20–25 m |
| Car too slow / always braking | `Fast` threshold is too low | Raise the shoulder start for `Fast` (e.g., `(25f, 40f)` → `(35f, 50f)`) |
| Car crashes on corners | Not braking enough before sharp turns | Ensure `HardLeft`/`HardRight` → `Brake` rules exist in the throttle rule set |
| Nothing happens, car sits still | `HardCodeThrottle(0f)` override is active | Remove both `HardCode` calls |
| `vizText` is blank | `vizText` not assigned in Inspector | Assign the Text component on the car prefab in the Inspector |

### Reading `RenderFuzzySetAscii` output

Call this in `Start()` on each set and read it in the Console window:

```
Speed DoM:
  0        10        20        30        40
  |Slow____|          |         |         |
           |__Medium__|         |         |
                      |___Fast__|_________|
```

- You want the shoulders and triangles to cover the expected real-world input range.
- Gaps between functions mean no membership at that value → erratic behavior.
- Heavily overlapping functions are fine — they produce smooth blends.

---

## Step 9 — Tune Your Parameters

The numbers passed to `GenerateCrossfadeFuzzySet` are tunable. Start with these,
then adjust based on what you observe in Play mode:

**Speed thresholds** — what counts as "fast" depends on your track:
```
Slow:   (0f, 10f)  →  if car never accelerates past 5 m/s, try (0f, 5f)
Medium: 18f        →  raise/lower to change when Coast kicks in
Fast:   (25f, 40f) →  raise if braking starts too early
```

**Steering angle thresholds** — what counts as a "hard" turn:
```
HardLeft:  (-60f, -30f)  →  tighten to (-40f, -20f) if over-correcting
Left:      -12f           →  lower for earlier gentle turns
Straight:   0f            →  keep at 0
Right:      12f           →  mirror of Left
HardRight: (30f, 60f)    →  mirror of HardLeft
```

> A fast sweep: shrink the `Straight` peak range (currently 0) to a tuple
> like `(-5f, 5f)` so small angles still count as straight.

---

## Optional — Improve Performance

Once basic driving works, try these upgrades:

### Scale look-ahead with speed

Looking further ahead at high speed gives the car more time to react to upcoming curves:

```csharp
float lookahead = Mathf.Clamp(Speed * 1.5f, 10f, 40f);
Vector3 lookAheadDir = Racetrack.GetDirectionAhead(lookahead);
```

### Add road offset as a secondary steering input

Even if the angle is near zero, the car may be drifting off-center. Road offset
corrects this drift:

```csharp
// Declare new enum and FuzzySet:
enum FzInputRoadOffset { FarLeft, Center, FarRight }
FuzzySet<FzInputRoadOffset> fzRoadOffsetSet;

// Build it in a new method:
private FuzzySet<FzInputRoadOffset> GetRoadOffsetSet()
{
    // Typical road half-width is ~4–5m. Full offset beyond ~3m = far off-center.
    return GenerateCrossfadeFuzzySet<FzInputRoadOffset>(
        (-5f, -3f),  // FarLeft
        0f,           // Center
        (3f, 5f)     // FarRight
    );
}

// Compute it in Update():
Vector3 trackDir   = Racetrack.ClosestPointDirectionOnPath; trackDir.y = 0f;
Vector3 trackRight = Vector3.Cross(Vector3.up, trackDir).normalized;
Vector3 carFlat    = new Vector3(transform.position.x, 0f, transform.position.z);
Vector3 closest    = new Vector3(Racetrack.ClosestPointOnPath.x, 0f, Racetrack.ClosestPointOnPath.z);
float roadOffset   = Vector3.Dot(carFlat - closest, trackRight);

fzRoadOffsetSet.Evaluate(roadOffset, fzInputValueSet);

// Add rules:
If(FzInputRoadOffset.FarRight).Then(FzOutputWheel.TurnLeft),
If(FzInputRoadOffset.Center)  .Then(FzOutputWheel.GoStraight),
If(FzInputRoadOffset.FarLeft) .Then(FzOutputWheel.TurnRight),
```

### Estimate upcoming curvature to brake early

Sample two points ahead and compare directions to estimate how sharp the next bend is:

```csharp
Vector3 near      = Racetrack.GetDirectionAhead(10f);
Vector3 far       = Racetrack.GetDirectionAhead(30f);
float curvature   = Vector3.Angle(near, far);  // 0 = straight, large = sharp bend ahead

// Use curvature as an additional throttle input:
// If(FzInputCurvature.Sharp).Then(FzOutputThrottle.Brake)
```

---

## Submission Checklist

- [ ] `StudentName` in `Awake()` is your real name (not "George P. Burdell")
- [ ] `// compile_check` line at the top is **removed** (not just commented out)
- [ ] No `HardCodeSteering(...)` calls anywhere in the file
- [ ] No `HardCodeThrottle(...)` calls anywhere in the file
- [ ] No direct assignment to `Throttle` or `Steering`
- [ ] Every output enum has **at least 3 labels**
- [ ] Every label of every input enum appears in every rule set's antecedents
- [ ] `base.Update()` is the last call in `Update()`
- [ ] `base.Start()` is the first call in `Start()`
- [ ] Tested on all four scenes: `RaceTrackFZ`, `WindingRaceTrackFZ`, `FastSweepersRaceTrackFZ`, `DragRaceFZ`
- [ ] Submit only `FuzzyVehicle.cs`
