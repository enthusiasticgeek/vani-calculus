# vani-calculus

Numerical calculus library for the [vāṇī compiler](https://github.com/enthusiasticgeek/vani-compiler).

Provides integration, differentiation, root-finding, ODE solvers, polynomial arithmetic, and interpolation — all as pure vāṇī source. Does **not** reimplement primitive math that is already a vāṇī compiler builtin (`sin`, `cos`, `exp`, `log`, `pow`, `sqrt`, `f64_erf`, `i64_factorial`, etc.).

## Add to your project

```toml
# vani.toml
[deps]
calculus = { registry = "kosh", version = "^0.2" }
```

```sh
vanic add calculus
vanic build
```

## What's included (v0.2.0)

| Module | Functions |
|---|---|
| Integration | `integrate_trapz`, `integrate_simpson`, `integrate_romberg`, `integrate_gauss_legendre_5`, `integrate_adaptive_trapz` |
| Differentiation | `diff_central`, `diff_forward`, `diff_second`, `gradient_1d`, `jacobian_1d`, `hessian_diag` |
| Root-finding | `bisect`, `secant`, `newton`, `brent` |
| Optimization | `golden_section`, `brent_min`, `newton_min` |
| ODE solvers | `euler_solve`, `rk4_solve`, `rk45_step`, `adams_bashforth_2` |
| Polynomials | `poly_eval`, `poly_deriv_coeffs`, `poly_add`, `poly_mul` |
| Interpolation | `lagrange_interp`, `linear_interp_table`, `cubic_spline_natural`, `cubic_spline_eval` |
| Series | `kahan_sum`, `partial_sum`, `running_mean_update` |

## What this library does NOT provide

These are already vāṇī compiler builtins — call them directly, no import needed:

`sin` `cos` `tan` `asin` `acos` `atan` `atan2` `sinh` `cosh` `tanh`
`exp` `log` `log2` `log10` `pow` `sqrt` `abs` `floor` `ceil`
`f64_erf` `f64_erfc` `f64_tgamma` `f64_lgamma` `f64_cbrt` `f64_hypot` `f64_fma`
`f64_lerp` `f64_inv_lerp` `f64_clamp` `f64_quadratic_root`
`i64_factorial` `i64_binomial` `i64_perm` `i64_gcd` `i64_lcm`
`i64_is_prime` `i64_fibonacci` `f64_geometric_mean` `f64_harmonic_mean`

## License

MIT
