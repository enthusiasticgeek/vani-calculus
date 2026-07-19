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

## Safety / WCET annotations

- [x] `#[bounded_stack(bytes=N)]` added to all 26 public functions (exact values from
      `vanic stack-depth`; range 64–520 bytes per frame, 440 bytes entry-point max for
      `integrate_romberg`).
- [x] `#[bounded(52)]` added to `integrate_adaptive_trapz` (52 = f64 mantissa halvings
      before interval underflow; resolves the UNBOUNDED self-recursion warning).
- [x] `#[wcet(cycles=N)]` added to all 10 leaf functions (no loops, no fn-ptr args);
      not applicable to data-driven loop functions — the WCET checker rejects `#[wcet]`
      when a `while i < n` loop has a runtime bound.
- [x] WCET budget formula documented in per-function comments for all loop-based functions:
      `n × k cycles` where k is cycles per iteration; fn-ptr functions document
      `n × (f_cycles + k)` and note that f_cycles depends on caller's f.
