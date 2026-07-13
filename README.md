# vani-calculus

Numerical calculus library for the [vāṇī compiler](https://github.com/enthusiasticgeek/vani-compiler).

Provides integration, differentiation, root-finding, ODE solvers, polynomial arithmetic, and interpolation — all as pure vāṇī source. Does **not** reimplement primitive math that is already a vāṇī compiler builtin (`sin`, `cos`, `exp`, `log`, `pow`, `sqrt`, `f64_erf`, `i64_factorial`, etc.).

## Add to your project

```toml
# vani.toml
[deps]
calculus = { registry = "kosh", version = "^0.1" }
```

```sh
vanic add calculus
vanic build
```

## What's included (v0.1 stubs — see TODO.md for implementation status)

| Module | Functions |
|---|---|
| Integration | `integrate_trapz`, `integrate_simpson`, `integrate_romberg` |
| Differentiation | `diff_central`, `diff_forward`, `diff_second`, `gradient_1d` |
| Root-finding | `bisect_step`, `secant_step`, `newton_step` |
| ODE | `euler_step`, `rk4_step` |
| Polynomials | `poly_eval` (Horner), `poly_deriv_coeffs` |
| Interpolation | `lagrange_interp` |

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
