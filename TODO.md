# vani-calculus — TODO

> Compiler builtins that already exist and must NOT be reimplemented:
> `sin cos tan asin acos atan atan2 sinh cosh tanh exp log log2 log10 pow sqrt abs`
> `f64_erf f64_erfc f64_tgamma f64_lgamma f64_cbrt f64_hypot f64_fma f64_lerp`
> `f64_inv_lerp f64_clamp f64_quadratic_root i64_factorial i64_binomial i64_perm`
> `i64_gcd i64_lcm i64_is_prime i64_fibonacci f64_geometric_mean f64_harmonic_mean`

---

## Integration

- [ ] `integrate_trapz` — loop body: accumulate `ys[i]` with endpoint halving
- [ ] `integrate_simpson` — alternate 4/2 weights; validate `n` even
- [ ] `integrate_romberg` — Richardson extrapolation table (k levels)
- [ ] `integrate_gauss_legendre_5` — 5-point Gauss–Legendre quadrature (fixed nodes/weights baked in)
- [ ] `integrate_adaptive_trapz` — recursive halving until error < tol (needs recursion bound annotation)
- [ ] Vec-based multi-interval version for non-uniform grids

## Differentiation

- [ ] `gradient_1d` — loop: central diff interior, forward/backward at endpoints
- [ ] `jacobian_1d` — numerical Jacobian for scalar function (2 evals per point)
- [ ] `hessian_diag` — diagonal of Hessian via second-order finite differences

## Root-finding

- [ ] `bisect` — full iterative loop: `max_iter` halvings, stop when `|b-a| < tol`
- [ ] `secant` — full iterative loop with convergence check
- [ ] `newton` — full iterative Newton-Raphson using `diff_central` for `f'`
- [ ] `brent` — Brent's method (combines bisect + secant + inverse quadratic); most robust

## ODE solvers

- [ ] `euler_solve` — apply `euler_step` over a time vector, return `Vec<f64>` of y values
- [ ] `rk4_solve` — apply `rk4_step` over a time vector; caller supplies slope function values
- [ ] `rk45_step` — Dormand-Prince adaptive step with error estimate
- [ ] `adams_bashforth_2` — two-step explicit linear multistep method

## Polynomial arithmetic

- [ ] `poly_eval` — Horner loop; use `f64_fma` (builtin) for FMA-fused variant
- [ ] `poly_deriv_coeffs` — differentiation coefficients
- [ ] `poly_add` / `poly_mul` — coefficient-wise addition and convolution multiply
- [ ] `poly_companion_matrix` — build companion matrix for eigenvalue root-finding (future)

## Interpolation

- [ ] `lagrange_interp` — double loop: basis polynomial products
- [ ] `linear_interp_table` — fast O(log n) binary-search + lerp (reuse builtin `f64_lerp`)
- [ ] `cubic_spline_natural` — compute spline coefficients (tridiagonal solve)
- [ ] `cubic_spline_eval` — evaluate spline at x given precomputed coefficients

## Series & summation

- [ ] `kahan_sum` — compensated summation for Vec<f64> (avoids float cancellation)
- [ ] `partial_sum` — prefix-sum Vec<f64>
- [ ] `running_mean_update` — Welford online mean update

## Safety / WCET annotations (future)

- [ ] Add `#[wcet(cycles=N)]` and `#[bounded_stack(bytes=N)]` to all public functions
      once implementations are complete and cost models are calibrated.
- [ ] Add `#[bounded(N)]` to any recursive helper (e.g., adaptive integration)
      and document the recursion bound.
