# HW5 Part 1 — Implementing `PredictThrow()`

> **Goal:** Given a launch position, a target position (and velocity), and a max throw speed, calculate the exact launch direction and speed so the projectile intercepts the target. Return `false` if the target is physically unreachable.

---

## Table of Contents
1. [The Big Picture](#the-big-picture)
2. [The Physics You Need to Know](#the-physics-you-need-to-know)
3. [Step 1 — Static Target Solver (Core Math)](#step-1--static-target-solver-core-math)
4. [Step 2 — Moving Target with Iterative Refinement](#step-2--moving-target-with-iterative-refinement)
5. [Step 3 — Putting It All Together in `PredictThrow()`](#step-3--putting-it-all-together-in-predictthrow)
6. [Handling the Return Value](#handling-the-return-value)
7. [Complete Pseudocode](#complete-pseudocode)
8. [Common Mistakes](#common-mistakes)
9. [Testing Strategy](#testing-strategy)

---

## The Big Picture

You are solving a **ballistic intercept** problem. Imagine throwing a ball in an arc: gravity pulls it down while it flies forward. You need to find *exactly* what angle and speed to throw it so it lands on a moving target.

There are two layers to this problem:
1. **Static target** — where should I throw if the target is standing still?
2. **Moving target** — the target is running. Where will it *be* when the ball arrives?

The recommended strategy (from the assignment) is:
- **Millington's static solver** → solves layer 1 exactly with quadratic math
- **Iterative refinement** → runs the static solver in a loop, each time predicting where the moving target will be, until the answer converges

---

## The Physics You Need to Know

A projectile launched with velocity vector **V** from position **P** follows this path:

```
position(t) = P + V·t + ½·g·t²
```

- `t` = time elapsed (seconds)
- `g` = gravity vector, e.g. `(0, -9.8, 0)` in Unity
- `V` = the launch velocity we want to solve for

We want the projectile to be at target position **T** at some time `t`:

```
T = P + V·t + ½·g·t²
```

Rearranging to solve for **V**:

```
V = (T - P) / t  -  (g·t / 2)
```

Let's name the displacement vector `d = T - P`. So:

```
V = d/t  -  g·t/2
```

This gives us the required launch velocity for any given flight time `t`. The trick is finding the **right** `t`.

---

## Step 1 — Static Target Solver (Core Math)

### The Goal
Find the flight time `t` such that `|V| = maxProjectileSpeed`. We want to use full speed for maximum range.

### Deriving the Equation

Square both sides of `|V| = maxSpeed`:

```
|V|² = maxSpeed²
|d/t - g·t/2|² = maxSpeed²
```

Expanding the squared magnitude (using dot product rules):

```
|d|²/t²  -  (d·g)  +  |g|²·t²/4  =  maxSpeed²
```

Multiply every term by `t²` to clear the fraction:

```
|d|²  -  (d·g)·t²  +  (|g|²/4)·t⁴  =  maxSpeed²·t²
```

Rearrange into standard polynomial form:

```
(|g|²/4)·t⁴  +  (-(d·g) - maxSpeed²)·t²  +  |d|²  =  0
```

**This is a quadratic equation in `t²`!** Let `u = t²`:

```
A·u²  +  B·u  +  C  =  0
```

Where:
```
A = |g|² / 4
B = -(d·g) - maxSpeed²
C = |d|²
```

> **Note on dot products:** `d·g` in Unity is `Vector3.Dot(d, g)`. Since `g = (0, -9.8, 0)`, this simplifies to `d.y * (-9.8)`. **But keep your code general** — just use `Vector3.Dot()`.

### Applying the Quadratic Formula

```
discriminant = B² - 4·A·C

u = (-B ± sqrt(discriminant)) / (2·A)
```

- If `discriminant < 0` → **no real solution** → target is **unreachable** → return `false`
- Each positive value of `u` gives a valid `t = sqrt(u)`
- You get up to **two solutions** (two different arcs that can reach the target)

### Choosing the Best `t`

You'll typically get two valid times. Pick the **smaller positive one** — that's the faster, flatter arc. It's almost always the right choice for dodgeball.

```
t1 = sqrt(u1)   // from +sqrt(discriminant)
t2 = sqrt(u2)   // from -sqrt(discriminant)
bestT = the smallest t that is > 0
```

If neither `u` is positive, the target is unreachable — return `false`.

### Computing the Final Launch Vector

Once you have `bestT`:

```
V = d / bestT  -  g * bestT / 2
projectileDir   = V.normalized
projectileSpeed = V.magnitude        // should be ≈ maxProjectileSpeed
interceptT      = bestT
```

---

## Step 2 — Moving Target with Iterative Refinement

The static solver assumes the target isn't moving. To handle a moving target, we repeat the static solver in a loop, each time improving our estimate of *where the target will be when the ball arrives*.

### The Key Insight

If the target moves at constant velocity `vel`, then at time `t` it will be at:
```
T_predicted = targetInitPos + targetConstVel * t
```

We don't know `t` yet — that's what we're solving for. So we guess, solve, then use the result to make a better guess. This is the "iterative" part.

### The Loop

```
t_estimate = 0   // start with: target is right where it is now

repeat up to MAX_ITERATIONS times:
    T_predicted = targetInitPos + targetConstVel * t_estimate
    
    solve static problem for T_predicted → get new t_new
    
    if |t_new - t_estimate| < CONVERGENCE_THRESHOLD:
        break  // good enough, we've converged
    
    t_estimate = t_new
```

Good values to start with:
- `MAX_ITERATIONS = 10` (usually converges in 3–5)
- `CONVERGENCE_THRESHOLD = 0.001f` (1 millisecond)

### Why Does This Work?

Each iteration gives a better prediction. If the target is moving slowly relative to the projectile speed, it converges very quickly. For typical dodgeball scenarios it's extremely reliable.

---

## Step 3 — Putting It All Together in `PredictThrow()`

Here is how the two pieces combine. Notice the static solver is a helper that the iterative loop calls.

### Architecture

```
PredictThrow()
│
├── Iterative loop:
│   ├── Predict target position at current t_estimate
│   └── Call SolveStatic() with predicted position
│           └── Quadratic math → returns new t
│
└── Final verification check → return true/false
```

### The `altT` Parameter

`altT` is the *other* solution from the quadratic formula (the longer arc). You can store it in `altT` and use it as the starting `t_estimate` for your iterative loop, since it may converge faster for certain scenarios. If you don't want to use it, just set `altT = -1f`.

---

## Handling the Return Value

You **must** correctly return `true` or `false`. The grader explicitly tests this. Return `false` when:

1. `discriminant < 0` — mathematically impossible (projectile too slow to reach target)
2. Both solutions `u1` and `u2` are negative or imaginary
3. After solving, the predicted intercept position is too far from the actual target position (exceeds `maxAllowedErrorDist`)

For check #3, verify your answer after solving:

```
Vector3 projectilePosAtT = projectilePos 
                         + projectileDir * projectileSpeed * interceptT 
                         + 0.5f * projectileGravity * interceptT * interceptT;

Vector3 targetPosAtT = targetInitPos + targetConstVel * interceptT;

float error = Vector3.Distance(projectilePosAtT, targetPosAtT);

if (error > maxAllowedErrorDist)
    return false;
```

---

## Complete Pseudocode

```
// ============================================================
// HELPER: Solve for a STATIC target
// Returns true and fills out t, dir, speed if reachable
// ============================================================
bool SolveStatic(projectilePos, maxProjectileSpeed, gravity, targetPos,
                 out dir, out speed, out t, out altT):

    d = targetPos - projectilePos

    A = Vector3.Dot(gravity, gravity) / 4.0f
    B = -Vector3.Dot(d, gravity) - maxProjectileSpeed * maxProjectileSpeed
    C = Vector3.Dot(d, d)

    discriminant = B*B - 4*A*C

    if discriminant < 0:
        dir = Vector3.forward   // dummy values
        speed = 0
        t = -1
        altT = -1
        return false            // TARGET UNREACHABLE

    sqrtDisc = Mathf.Sqrt(discriminant)
    u1 = (-B + sqrtDisc) / (2 * A)
    u2 = (-B - sqrtDisc) / (2 * A)

    // Store the alt solution (larger t = higher arc)
    t1 = (u1 > 0) ? Mathf.Sqrt(u1) : -1
    t2 = (u2 > 0) ? Mathf.Sqrt(u2) : -1

    // Pick smallest positive t as primary
    if t1 > 0 and (t2 <= 0 or t1 <= t2):
        bestT = t1
        altT  = (t2 > 0) ? t2 : -1
    else if t2 > 0:
        bestT = t2
        altT  = (t1 > 0) ? t1 : -1
    else:
        return false            // NO POSITIVE SOLUTION

    V     = d / bestT  -  gravity * bestT / 2
    dir   = V.normalized
    speed = V.magnitude
    t     = bestT

    return true


// ============================================================
// MAIN: PredictThrow — handles moving targets via iteration
// ============================================================
bool PredictThrow(projectilePos, maxProjectileSpeed, gravity,
                  targetInitPos, targetConstVel, targetForwardDir,
                  maxAllowedErrorDist,
                  out projectileDir, out projectileSpeed,
                  out interceptT, out altT):

    MAX_ITERATIONS = 10
    CONVERGE_THRESH = 0.001f

    // --- EDGE CASE: zero gravity ---
    // If gravity is zero, this is a straight-line problem.
    // Handle separately if needed (see Common Mistakes section).

    // --- INITIAL GUESS ---
    // Use the static solution at current target position to seed the loop.
    // altT from the first static solve can seed the iteration too.
    t_estimate = 0f
    altT = -1f

    for i = 0 to MAX_ITERATIONS - 1:

        // Predict where target will be at our current time estimate
        T_predicted = targetInitPos + targetConstVel * t_estimate

        // Solve static problem for that predicted position
        success = SolveStatic(projectilePos, maxProjectileSpeed, gravity,
                               T_predicted,
                               out projectileDir, out projectileSpeed,
                               out t_new, out altT)

        if not success:
            return false   // Can't reach even the predicted position

        if Mathf.Abs(t_new - t_estimate) < CONVERGE_THRESH:
            break          // Converged! We're done.

        t_estimate = t_new

    interceptT = t_new (or t_estimate after loop)

    // --- VERIFICATION ---
    // Check that the projectile actually reaches the target within error tolerance
    Vector3 projPosAtT  = projectilePos
                        + projectileDir * projectileSpeed * interceptT
                        + 0.5f * gravity * interceptT * interceptT

    Vector3 targPosAtT  = targetInitPos + targetConstVel * interceptT

    float error = Vector3.Distance(projPosAtT, targPosAtT)

    if error > maxAllowedErrorDist:
        return false    // Solution drifted too far; not reliable

    return true
```

---

## Common Mistakes

### 1. Forgetting to handle zero gravity
The quadratic formula breaks down if `|g| = 0` (division by zero in A). Check for this edge case first:
```
if (projectileGravity.sqrMagnitude < 1e-6f):
    // Straight-line throw: dir = (T - P).normalized, speed = maxProjectileSpeed
    // t = |T - P| / maxProjectileSpeed
    // Handle moving target: solve for t where |T + vel*t - P| / maxSpeed = t
    // This becomes a simpler quadratic (no t^4 term)
```

### 2. Taking the wrong square root
`u = t²`, so you need `t = sqrt(u)`. Only take the square root if `u > 0`. If `u` is slightly negative due to floating-point error (like `-0.0001`), clamp it to zero.

### 3. Using `d·g` wrong
`d·g` is a **dot product**, not a component-wise multiply. Use `Vector3.Dot(d, projectileGravity)`.

### 4. Not normalizing `projectileDir`
`V` (the launch velocity) has both direction and magnitude. `projectileDir` must be a **unit vector** — always call `.normalized` on it.

### 5. Returning `true` always
The grader tests unreachable targets. A target that is too far away, too fast, or at an extreme elevation must return `false`.

### 6. Using `Physics.gravity` directly inside `PredictThrow()`
The assignment says `PredictThrow()` cannot access live game state. The gravity is **passed in as a parameter** — use `projectileGravity`, not `Physics.gravity`. Calling `Physics.gravity` inside this method will fail the EditorMode tests.

---

## Testing Strategy

### Phase 1 — Test the Static Solver First
Set `targetConstVel = Vector3.zero` and test with:
- Target directly in front at various distances
- Target above and below
- Target that is clearly too far away (expect `false`)

### Phase 2 — Test the Iterative Solver
Add slow velocity to the target. Check that `interceptT` is reasonable and the ball path visually arcs to the target in the ShootingRange scene.

### Phase 3 — Use the ShootingRange
- Open `Assets/Scenes/ShootingRange.unity`
- Press **Space** to toggle to your algorithm (check the name shown)
- Press `4` for the mode most like Prison Dodgeball
- Press `r` to reset stats
- Aim for high accuracy AND good shots-per-minute

### Phase 4 — Run the EditorMode Unit Test
In Unity: **Window → General → Test Runner → EditorMode**
Run `ThrowTestEditorMode`. Add your own test cases to that file!

---

*Continue to [[Assignment 5 Shot Selection (Part 2)]] for `SelectThrow()` implementation.*
