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

- [ ] `integrate_trapz` — Horner loop: `h/2 * (ys[0] + 2*ys[1] + ... + ys[n-1])`, `h = (b-a)/n`
- [ ] `integrate_simpson` — alternate 4/2 weights; validate n even at runtime
- [ ] `integrate_romberg` — Richardson extrapolation table (k levels, recursive); annotate with `#[bounded(k)]`
- [ ] `integrate_gauss_legendre_5` — 5-point Gauss–Legendre quadrature (baked-in nodes/weights)
- [ ] `integrate_adaptive_trapz` — recursive halving until `|coarse - fine| < tol`

### Differentiation

- [ ] `diff_central` — `(f(x+h) - f(x-h)) / (2h)` (already stubbed)
- [ ] `diff_forward` — `(f(x+h) - f(x)) / h` (already stubbed)
- [ ] `diff_second` — `(f(x+h) - 2f(x) + f(x-h)) / h²` (already stubbed)
- [ ] `gradient_1d` — loop: central diff interior, forward/backward at endpoints
- [ ] `jacobian_1d` — numerical Jacobian for scalar function (2 evals per point)
- [ ] `hessian_diag` — diagonal of Hessian via second-order finite differences

### Root-finding

- [ ] `bisect_step` → full `bisect` — iterative: `max_iter` halvings, stop when `|b-a| < tol`
- [ ] `secant_step` → full `secant` — iterative loop with convergence check
- [ ] `newton_step` → full `newton` — Newton-Raphson using `diff_central` for f′
- [ ] `brent` — Brent's method (bisect + secant + inverse quadratic); most robust single-function root-finder

### ODE solvers

- [ ] `euler_step` → `euler_solve` — apply step over a time Vec, return `Vec<f64>` of y values
- [ ] `rk4_step` → `rk4_solve` — apply step over a time Vec; caller supplies slope function values
- [ ] `rk45_step` — Dormand-Prince adaptive step with error estimate
- [ ] `adams_bashforth_2` — two-step explicit linear multistep method

### Polynomial arithmetic

- [ ] `poly_eval` — Horner loop; use `f64_fma` builtin for FMA-fused variant
- [ ] `poly_deriv_coeffs` — differentiation coefficients
- [ ] `poly_add` / `poly_mul` — coefficient-wise addition and convolution multiply

### Interpolation

- [ ] `lagrange_interp` — double loop: basis polynomial products
- [ ] `linear_interp_table` — O(log n) binary-search + lerp (reuse builtin `f64_lerp`)
- [ ] `cubic_spline_natural` — compute spline coefficients (tridiagonal solve via while loop)
- [ ] `cubic_spline_eval` — evaluate spline at x given precomputed coefficients

### Series & summation

- [ ] `kahan_sum` — compensated summation for `Vec<f64>` using `for x in ref xs` manual loop
      (workaround until F64-3 lands; `vec_fold` on `Vec<f64>` is the clean form)
- [ ] `partial_sum` — prefix-sum `Vec<f64>` via `while` loop + `set`
- [ ] `running_mean_update` — Welford online mean update (single-step, no Vec needed)

---

## Safety / WCET annotations (after all implementations complete)

- [ ] Add `#[wcet(cycles=N)]` and `#[bounded_stack(bytes=N)]` to all public functions
      once implementations are complete and cost models are calibrated.
- [ ] Add `#[bounded(N)]` to any recursive helper (e.g., `integrate_romberg`, `integrate_adaptive_trapz`)
      and document the recursion bound in the function's comment.
- [ ] `kahan_sum` and all loop-over-Vec functions: document expected WCET budget formula
      (`N * cycles_per_iteration`) for the WCET checker.
