# FuzzyVehicle Assignment Guide

## What You're Building

You are filling in `Assets/Scripts/GameAIStudentWork/FuzzyVehicle.cs` to control a racecar
using fuzzy logic. Every frame, your code must:

1. Read sensor values from the car and track.
2. Run those values through fuzzy sets to get degrees of membership.
3. Fire fuzzy rules that map inputs → throttle/steering outputs.
4. Call `ApplyFuzzyRules()` — the only allowed way to set throttle and steering.

---

## The Big Picture Loop

```
Each frame (Update):
  1. Read crisp inputs  (speed, steering angle, road offset, etc.)
  2. Fuzzify inputs     (feedValues into your input FuzzySets)
  3. ApplyFuzzyRules()  (framework evaluates rules → crisp throttle & steering)
```

There are only two phases to implement: **Setup** (run once in `Start()`) and **Runtime** (run every `Update()`).

---

## Step 1 — Choose Your Inputs

You need at least two good crisp inputs. Here are the three most useful ones:

| Input | How to compute it | Useful for |
|---|---|---|
| **Speed** | `Speed` (built-in, m/s) | Throttle control |
| **Steering angle** | Signed angle between car forward and look-ahead track direction | Steering control |
| **Road offset** | Signed distance from car to track centerline | Steering correction |

### Computing steering angle (look-ahead approach)

```csharp
// Project car position onto XZ plane (track lies in XZ)
Vector3 carPos = new Vector3(transform.position.x, 0f, transform.position.z);

// Get the track direction 15m ahead of the car
Vector3 lookAheadDir = Racetrack.GetDirectionAhead(15f);
lookAheadDir.y = 0f;  // flatten to XZ

// Signed angle: positive = target is to the right, negative = to the left
float steerAngle = Vector3.SignedAngle(transform.forward, lookAheadDir, Vector3.up);
```

### Computing road offset (signed distance from centerline)

```csharp
Vector3 carPosFlat    = new Vector3(transform.position.x, 0f, transform.position.z);
Vector3 closestFlat   = new Vector3(Racetrack.ClosestPointOnPath.x, 0f, Racetrack.ClosestPointOnPath.z);
Vector3 trackDirFlat  = Racetrack.ClosestPointDirectionOnPath;
trackDirFlat.y = 0f;

// Perpendicular of track direction (points left of travel)
Vector3 trackRight = Vector3.Cross(Vector3.up, trackDirFlat).normalized;
Vector3 offset     = carPosFlat - closestFlat;

// Positive = car is to the right of center, negative = to the left
float roadOffset = Vector3.Dot(offset, trackRight);
```

---

## Step 2 — Declare Fuzzy Enums

Each input and output variable is an `enum`. Replace/expand the starter enums.

```csharp
// INPUTS
enum FzInputSpeed        { Slow, Medium, Fast }
enum FzInputSteerAngle   { HardLeft, Left, Straight, Right, HardRight }
// (3 labels is fine for a starter; 5 gives more precision for steering)

// OUTPUTS  — must have at least 3 states each
enum FzOutputThrottle    { Brake, Coast, Accelerate }
enum FzOutputWheel       { TurnLeft, GoStraight, TurnRight }
```

Add member variables for every set and rule set you create:

```csharp
FuzzySet<FzInputSpeed>       fzSpeedSet;
FuzzySet<FzInputSteerAngle>  fzSteerAngleSet;

FuzzySet<FzOutputThrottle>   fzThrottleSet;
FuzzyRuleSet<FzOutputThrottle> fzThrottleRuleSet;

FuzzySet<FzOutputWheel>      fzWheelSet;
FuzzyRuleSet<FzOutputWheel>  fzWheelRuleSet;

FuzzyValueSet fzInputValueSet = new FuzzyValueSet();
FuzzyValueSet mergedThrottle  = new FuzzyValueSet();
FuzzyValueSet mergedWheel     = new FuzzyValueSet();
```

---

## Step 3 — Build Fuzzy Sets

### Rule of thumb
- **Inputs** → `GenerateCrossfadeFuzzySet<T>()` (smooth overlapping shapes)
- **Outputs** → `GenerateDiscreteFuzzySet<T>()` (one crisp representative value each)

### Input example: Speed (0–30 m/s is a reasonable range)

```csharp
private FuzzySet<FzInputSpeed> GetSpeedSet()
{
    // 3 enum values  → 3 arguments
    // Outermost args are shoulders; inner arg is a triangle peak
    // Slow: shoulder up to ~10 m/s
    // Medium: triangle peak at 18 m/s
    // Fast: shoulder from ~25 m/s on
    return GenerateCrossfadeFuzzySet<FzInputSpeed>(
        (0f, 10f),   // Slow  — shoulder, plateau 0–10
        18f,          // Medium — triangle peak at 18
        (25f, 40f)   // Fast  — shoulder, plateau 25+
    );
}
```

> **Tip:** Print the shape to console once in Start() to verify: `Debug.Log(RenderFuzzySetAscii(fzSpeedSet));`

### Input example: Steering angle (roughly –60° to +60° for normal driving)

```csharp
private FuzzySet<FzInputSteerAngle> GetSteerAngleSet()
{
    // 5 enum values (HardLeft, Left, Straight, Right, HardRight)
    return GenerateCrossfadeFuzzySet<FzInputSteerAngle>(
        (-60f, -30f),  // HardLeft
        -12f,           // Left
        0f,             // Straight
        12f,            // Right
        (30f, 60f)     // HardRight
    );
}
```

### Output example: Throttle

```csharp
private FuzzySet<FzOutputThrottle> GetThrottleSet()
{
    // 3 enum values → 3 representative crisp values
    // Brake = -1.0, Coast = 0.0, Accelerate = 1.0
    return GenerateDiscreteFuzzySet<FzOutputThrottle>(-1f, 0f, 1f);
}
```

### Output example: Steering wheel

```csharp
private FuzzySet<FzOutputWheel> GetWheelSet()
{
    // TurnLeft = -1.0, GoStraight = 0.0, TurnRight = 1.0
    return GenerateDiscreteFuzzySet<FzOutputWheel>(-1f, 0f, 1f);
}
```

---

## Step 4 — Write Fuzzy Rules

Rules connect input states to output states using `If(...).Then(...)`.
Every enum label of every input variable used **must** appear in at least one rule.

### Throttle rules (speed-based)

```csharp
private FuzzyRuleSet<FzOutputThrottle> GetThrottleRuleSet(FuzzySet<FzOutputThrottle> throttle)
{
    FuzzyRule<FzOutputThrottle>[] rules =
    {
        If(FzInputSpeed.Slow)  .Then(FzOutputThrottle.Accelerate),
        If(FzInputSpeed.Medium).Then(FzOutputThrottle.Coast),
        If(FzInputSpeed.Fast)  .Then(FzOutputThrottle.Brake),

        // Brake harder when there is a sharp steering demand
        If(FzInputSteerAngle.HardLeft) .Then(FzOutputThrottle.Brake),
        If(FzInputSteerAngle.HardRight).Then(FzOutputThrottle.Brake),
        // Gentle turns: don't cut throttle fully
        If(FzInputSteerAngle.Left)     .Then(FzOutputThrottle.Coast),
        If(FzInputSteerAngle.Right)    .Then(FzOutputThrottle.Coast),
        // Straight: no throttle penalty
        If(FzInputSteerAngle.Straight) .Then(FzOutputThrottle.Accelerate),
    };
    return new FuzzyRuleSet<FzOutputThrottle>(throttle, rules);
}
```

### Steering rules (angle-based)

```csharp
private FuzzyRuleSet<FzOutputWheel> GetWheelRuleSet(FuzzySet<FzOutputWheel> wheel)
{
    FuzzyRule<FzOutputWheel>[] rules =
    {
        If(FzInputSteerAngle.HardLeft) .Then(FzOutputWheel.TurnLeft),
        If(FzInputSteerAngle.Left)     .Then(FzOutputWheel.TurnLeft),
        If(FzInputSteerAngle.Straight) .Then(FzOutputWheel.GoStraight),
        If(FzInputSteerAngle.Right)    .Then(FzOutputWheel.TurnRight),
        If(FzInputSteerAngle.HardRight).Then(FzOutputWheel.TurnRight),
    };
    return new FuzzyRuleSet<FzOutputWheel>(wheel, rules);
}
```

> **Rule coverage check:** Every value of `FzInputSteerAngle` and `FzInputSpeed` appears as an antecedent in each rule set. If you skip any, the framework may produce NaN.

---

## Step 5 — Initialize in Start()

```csharp
protected override void Start()
{
    base.Start();

    // Input sets
    fzSpeedSet      = GetSpeedSet();
    fzSteerAngleSet = GetSteerAngleSet();

    // Output sets + rule sets
    fzThrottleSet     = GetThrottleSet();
    fzThrottleRuleSet = GetThrottleRuleSet(fzThrottleSet);

    fzWheelSet     = GetWheelSet();
    fzWheelRuleSet = GetWheelRuleSet(fzWheelSet);

    // Print shapes to Console once to verify your DoM boundaries look correct
    Debug.Log(RenderFuzzySetAscii(fzSpeedSet));
    Debug.Log(RenderFuzzySetAscii(fzSteerAngleSet));
    Debug.Log(RenderFuzzySetAscii(fzThrottleSet));
    Debug.Log(RenderFuzzySetAscii(fzWheelSet));
}
```

---

## Step 6 — Fuzzify and Apply in Update()

```csharp
override protected void Update()
{
    if (!Racetrack.IsInitialized)
    {
        base.Update();
        return;
    }

    // --- 1. Compute crisp inputs ---
    float speed = Speed;  // m/s

    Vector3 lookAheadDir = Racetrack.GetDirectionAhead(15f);
    lookAheadDir.y = 0f;
    float steerAngle = Vector3.SignedAngle(transform.forward, lookAheadDir, Vector3.up);

    // --- 2. Fuzzify ---
    fzSpeedSet     .Evaluate(speed,      fzInputValueSet);
    fzSteerAngleSet.Evaluate(steerAngle, fzInputValueSet);

    // --- 3. Apply rules ---
    ApplyFuzzyRules<FzOutputThrottle, FzOutputWheel>(
        fzThrottleRuleSet,
        fzWheelRuleSet,
        fzInputValueSet,
        out var throttleRuleOutput,
        out var wheelRuleOutput,
        ref mergedThrottle,
        ref mergedWheel
    );

    // --- 4. Optional debug HUD ---
    if (vizText != null)
    {
        strBldr.Clear();
        strBldr.AppendLine($"speed: {speed:F1} m/s  angle: {steerAngle:F1}°");
        DiagnosticPrintFuzzyValueSet<FzInputSpeed>(fzInputValueSet, strBldr);
        DiagnosticPrintFuzzyValueSet<FzInputSteerAngle>(fzInputValueSet, strBldr);
        DiagnosticPrintRuleSet<FzOutputThrottle>(fzThrottleRuleSet, throttleRuleOutput, strBldr);
        DiagnosticPrintRuleSet<FzOutputWheel>(fzWheelRuleSet, wheelRuleOutput, strBldr);
        vizText.text = strBldr.ToString();
    }

    base.Update();  // must be last
}
```

> **Critical:** Remove `HardCodeSteering(0f)` and `HardCodeThrottle(0.5f)` from the starter code. The autograder will fail if those calls remain.

---

## Step 7 — Test and Tune

### Test in Unity
1. Open `Scenes/RaceTrackFZ` (most curvy — hardest test).
2. Press Play, watch the car. Use the HUD and Scene view to see fuzzy states.
3. Use `Debug.DrawRay(transform.position, lookAheadDir * 15f, Color.green)` to visualize the look-ahead vector.

### Common problems and fixes

| Symptom | Likely cause | Fix |
|---|---|---|
| Car drives straight off track | Steering rules not firing / wrong sign | Check `SignedAngle` axis is `Vector3.up`; print `steerAngle` each frame |
| NaN / Infinity errors | A rule set has an input label with no rule | Add rules for every enum value of every input used |
| Car oscillates left/right | Look-ahead distance too short | Increase look-ahead from 15 → 25 m at high speed |
| Car too slow | Throttle rules brake too aggressively | Widen the `Fast` shoulder start point (e.g., 25 → 35) |
| Car flips on sharp turns | Not braking enough for tight corners | Add `HardLeft`/`HardRight` → `Brake` rule with stronger weight |

### Tuning fuzzy set boundaries

The numbers you pass to `GenerateCrossfadeFuzzySet` are tunable parameters. Adjust them until behavior feels right:

```
Fast starts at 25 m/s  →  too early braking? raise to 30 m/s
HardLeft at -30°       →  still oversteering? tighten to -20°
```

---

## Step 8 — Submission Checklist

- [ ] `StudentName` in `Awake()` reflects your real name (not "George P. Burdell")
- [ ] **No** `HardCodeSteering(...)` or `HardCodeThrottle(...)` calls anywhere
- [ ] **No** direct assignment to `Throttle` or `Steering`
- [ ] Every enum label of every input variable appears in each rule set's antecedents
- [ ] Output enums have at least 3 states each
- [ ] Remove or comment out the `// compile_check` line at the top before graded submission
- [ ] Run Playmode tests for all four tracks: `RaceTrackFZ`, `WindingRaceTrackFZ`, `FastSweepersRaceTrackFZ`, `DragRaceFZ`
- [ ] Submit only `FuzzyVehicle.cs`

---

## Optional: Improving Performance

Once the basics work, try these upgrades:

### Longer look-ahead at higher speeds

Instead of a fixed 15 m look-ahead, scale it with speed:

```csharp
float lookahead = Mathf.Clamp(Speed * 1.5f, 10f, 40f);
Vector3 lookAheadDir = Racetrack.GetDirectionAhead(lookahead);
```

### Add road offset as a second steering input

```csharp
// signed distance from centerline (negative = left of center)
Vector3 carPosFlat   = new Vector3(transform.position.x, 0f, transform.position.z);
Vector3 closestFlat  = new Vector3(Racetrack.ClosestPointOnPath.x, 0f, Racetrack.ClosestPointOnPath.z);
Vector3 trackDir     = Racetrack.ClosestPointDirectionOnPath; trackDir.y = 0f;
Vector3 trackRight   = Vector3.Cross(Vector3.up, trackDir).normalized;
float roadOffset     = Vector3.Dot(carPosFlat - closestFlat, trackRight);
```

Then add an enum + fuzzy set for it and rules like:
```csharp
If(FzInputRoadOffset.FarRight).Then(FzOutputWheel.TurnLeft),
If(FzInputRoadOffset.Center)  .Then(FzOutputWheel.GoStraight),
If(FzInputRoadOffset.FarLeft) .Then(FzOutputWheel.TurnRight),
```

### Look further ahead for curvature

Sample two points ahead and compare their directions to estimate upcoming curve sharpness:

```csharp
Vector3 near = Racetrack.GetDirectionAhead(10f);
Vector3 far  = Racetrack.GetDirectionAhead(30f);
float curvature = Vector3.Angle(near, far);  // 0 = straight, large = sharp bend
```

Use `curvature` as an input to brake earlier for upcoming tight turns.
