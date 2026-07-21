# vani-calculus — TODO

> Compiler builtins that already exist and must NOT be reimplemented:
> `sin cos tan asin acos atan atan2 sinh cosh tanh exp log log2 log10 pow sqrt abs`
> `f64_erf f64_erfc f64_tgamma f64_lgamma f64_cbrt f64_hypot f64_fma f64_lerp`
> `f64_inv_lerp f64_clamp f64_quadratic_root i64_factorial i64_binomial i64_perm`
> `i64_gcd i64_lcm i64_is_prime i64_fibonacci f64_geometric_mean f64_harmonic_mean`

---

## Compiler dependencies (items blocked on vani-compiler changes)

These functions cannot be implemented until the corresponding compiler ticket lands.
Track progress in `vani-compiler/docs/TODO_CURRENT.md` under "Vec\<f64\> builtin parity".

| Ticket | Compiler change needed | Unblocks in this package |
|--------|----------------------|--------------------------|
| F64-3 | `vec_fold` / `vec_map` on `Vec<f64>` | `kahan_sum`, `partial_sum` (can use manual loop as interim workaround) |
| F64-3 | `vec_running_sum` on `Vec<f64>` | `partial_sum` running-prefix variant |

> **Note:** `for x in ref xs` over `Vec<f64>` **already works** (confirmed against
> `checker.rs:11661`). All integration, differentiation, root-finding, and ODE functions
> can be implemented with `while` / `for-in-ref` loops and `xs[i]` indexing **right now**.
> The only dependency above is a style preference; manual loops are a complete workaround.

---

## Can implement NOW (no compiler changes needed)

All items below use `for x in ref xs`, `while`, `xs[i]`, `set(mut ref xs, i, v)`, and
scalar f64 arithmetic — all confirmed working on `Vec<f64>` in the current compiler.

### Integration

- [x] `integrate_trapz` — Horner loop: `h/2 * (ys[0] + 2*ys[1] + ... + ys[n-1])`, `h = (b-a)/n`
- [x] `integrate_simpson` — alternate 4/2 weights; validate n even at runtime
- [x] `integrate_romberg` — Richardson extrapolation table (k levels, recursive); annotate with `#[bounded(k)]`
- [x] `integrate_gauss_legendre_5` — 5-point Gauss–Legendre quadrature (baked-in nodes/weights)
- [x] `integrate_adaptive_trapz` — recursive halving until `|coarse - fine| < tol`

### Differentiation

- [x] `diff_central` — `(f(x+h) - f(x-h)) / (2h)` (already stubbed)
- [x] `diff_forward` — `(f(x+h) - f(x)) / h` (already stubbed)
- [x] `diff_second` — `(f(x+h) - 2f(x) + f(x-h)) / h²` (already stubbed)
- [x] `gradient_1d` — loop: central diff interior, forward/backward at endpoints
- [x] `jacobian_1d` — numerical Jacobian for scalar function (2 evals per point)
- [x] `hessian_diag` — diagonal of Hessian via second-order finite differences

### Root-finding

- [x] `bisect_step` → full `bisect` — iterative: `max_iter` halvings, stop when `|b-a| < tol`
- [x] `secant_step` → full `secant` — iterative loop with convergence check
- [x] `newton_step` → full `newton` — Newton-Raphson using `diff_central` for f′
- [x] `brent` — Brent's method (bisect + secant + inverse quadratic); most robust single-function root-finder

### Optimization

- [x] `golden_section(a, b, f, tol, max_iter)` — golden-section search; 1D min of unimodal f on [a, b]; ≈2.08 log₁₀((b-a)/tol) evals
- [x] `brent_min(a, b, f, tol, max_iter)` — Brent's method for 1D minimization (parabolic interpolation + golden-section fallback); superlinear convergence
- [x] `newton_min(x0, f, tol, max_iter)` — Newton's method x ← x − f′/f′′ (central-difference approx, h=1e-5); quadratically convergent near minimum

### ODE solvers

- [x] `euler_step` → `euler_solve` — apply step over a time Vec, return `Vec<f64>` of y values
- [x] `rk4_step` → `rk4_solve` — apply step over a time Vec; caller supplies slope function values
- [x] `rk45_step` — Dormand-Prince adaptive step with error estimate
- [x] `adams_bashforth_2` — two-step explicit linear multistep method

### Polynomial arithmetic

- [x] `poly_eval` — Horner loop; use `f64_fma` builtin for FMA-fused variant
- [x] `poly_deriv_coeffs` — differentiation coefficients
- [x] `poly_add` / `poly_mul` — coefficient-wise addition and convolution multiply

### Interpolation

- [x] `lagrange_interp` — double loop: basis polynomial products
- [x] `linear_interp_table` — O(log n) binary-search + lerp (reuse builtin `f64_lerp`)
- [x] `cubic_spline_natural` — compute spline coefficients (tridiagonal solve via while loop)
- [x] `cubic_spline_eval` — evaluate spline at x given precomputed coefficients

### Series & summation

- [x] `kahan_sum` — compensated summation for `Vec<f64>` using manual loop
      (workaround until F64-3 lands; `vec_fold` on `Vec<f64>` is the clean form)
- [x] `partial_sum` — prefix-sum `Vec<f64>` via `while` loop + `set`
- [x] `running_mean_update` — Welford online mean update (single-step, no Vec needed)

---

## v0.3.0 additions — implicit ODE solvers and boundary-value problems

Fills two gaps flagged in kosh-index/ROADMAP.md's "Known gaps within
'mostly done' rows" (N1: implicit/stiff ODE solvers; N2: ODE
boundary-value-problem solver).

### Implicit ODE solvers (N1)

- [x] `backward_euler_step` / `backward_euler_solve` — each step solves its
      implicit equation via Newton's method (central-difference derivative,
      same technique `newton()` uses), NOT fixed-point iteration -- fixed-
      point iteration on backward Euler is only a contraction for
      `h < 1/|df/dy|`, which would fail to converge on exactly the stiff
      problems this solver exists for. Validated: matches the algebraic
      solution of the implicit equation for a linear ODE, and stays bounded
      on a stiff problem (`dy/dt=-50y`, `h=0.1`) where `euler_solve`
      diverges -- see `tests/test_implicit_ode.vani`.
- [x] `crank_nicolson_step` / `crank_nicolson_solve` — same Newton-based
      implicit solve, trapezoidal (second-order) instead of backward
      (first-order). Validated as more accurate than backward Euler at the
      same step size on a non-stiff problem.

### ODE boundary-value problems via shooting (N2)

- [x] `bvp_rk4_step` — paired RK4 step for `y''=f(t,y,y')`, packed as
      `[y,v]` (a function can only return one value, so the 2D state is a
      `Vec<f64>` of length 2 rather than two separate returns)
- [x] `bvp_shoot_integrate` / `bvp_shoot_trajectory` — forward-integrate
      from a guessed initial slope; `_integrate` returns just the endpoint
      (cheap, used inside the search), `_trajectory` returns the full path
- [x] `bvp_shoot_solve` — secant search over the initial slope (own inlined
      loop, not a reuse of `secant()` -- `secant` takes a `fn(f64)->f64`
      and vāṇी has no closures to capture `a`/`b`/`ya`/`yb`/`f` into one).
      Validated against `y''=-y`, `y(0)=0`, `y(pi/2)=1` (exact: `sin(t)`)
      from deliberately wrong initial-slope guesses, checked at every point
      of the returned trajectory (max error ~4e-9), not just the endpoint
      -- see `tests/test_bvp.vani`.

### Tests and examples

- [x] `tests/test_implicit_ode.vani`, `tests/test_bvp.vani`
- [x] `examples/stiff_ode_demo.vani` — explicit Euler diverging vs backward
      Euler / Crank-Nicolson staying bounded, side by side
- [x] `examples/bvp_shoot_demo.vani` — the `sin(t)` BVP, full trajectory
      printed against the exact solution

### Not done (N3, out of scope for v0.3.0)

- [ ] Interval arithmetic / rigorous error propagation -- flagged in
      ROADMAP.md as "no obvious home yet and unclear real-world pull,"
      intentionally left for a future version if real demand shows up.
- [ ] BDF2 (2-step backward differentiation) -- backward Euler and
      Crank-Nicolson already cover "implicit/stiff," and BDF2 needs a
      special startup step (only one previous point exists initially);
      not worth the added complexity without a concrete need.

---

## Safety / WCET annotations

- [x] `#[bounded_stack(bytes=N)]` added to all v0.2.0 public functions (exact values from
      `vanic stack-depth`; range 64–520 bytes per frame, 440 bytes entry-point max for
      `integrate_romberg`). The 8 v0.3.0 additions follow the same discipline via
      `vanic check`'s exact reported worst-case; largest is `bvp_shoot_solve` at
      784 bytes (composes the secant loop with `bvp_shoot_integrate` -> `bvp_rk4_step`).
- [x] `#[bounded(52)]` added to `integrate_adaptive_trapz` (52 = f64 mantissa halvings
      before interval underflow; resolves the UNBOUNDED self-recursion warning).
- [x] `#[wcet(cycles=N)]` added to all 10 leaf functions (no loops, no fn-ptr args);
      not applicable to data-driven loop functions — the WCET checker rejects `#[wcet]`
      when a `while i < n` loop has a runtime bound.
- [x] WCET budget formula documented in per-function comments for all loop-based functions:
      `n × k cycles` where k is cycles per iteration; fn-ptr functions document
      `n × (f_cycles + k)` and note that f_cycles depends on caller's f.
