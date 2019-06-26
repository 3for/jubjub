# jubjub [![Crates.io](https://img.shields.io/crates/v/jubjub.svg)](https://crates.io/crates/jubjub) #

<img
 width="15%"
 align="right"
 src="https://raw.githubusercontent.com/zcash/zips/master/protocol/jubjub.png"/>

This is a pure Rust implementation of the Jubjub elliptic curve group and its associated fields.

* **This implementation has not been reviewed or audited. Use at your own risk.**
* This implementation targets Rust `1.33` or later.
* All operations are constant time unless explicitly noted.

## Features

* `std` (on by default): Enables APIs that leverage the Rust standard library.
* `nightly`: Enables `subtle/nightly` which prevents compiler optimizations that could jeopardize constant time operations.

## [Documentation](https://docs.rs/jubjub)

## Curve Description

Jubjub is the [twisted Edwards curve](https://en.wikipedia.org/wiki/Twisted_Edwards_curve) `-u^2 + v^2 = 1 + d.u^2.v^2` of rational points over `GF(q)` with a subgroup of prime order `r` and cofactor `8`.

```
q = 0x73eda753299d7d483339d80809a1d80553bda402fffe5bfeffffffff00000001
r = 0x0e7db4ea6533afa906673b0101343b00a6682093ccc81082d0970e5ed6f72cb7
d = -(10240/10241)
```

The choice of `GF(q)` is made to be the scalar field of the BLS12-381 elliptic curve construction.

Jubjub is birationally equivalent to a [Montgomery curve](https://en.wikipedia.org/wiki/Montgomery_curve) `y^2 = x^3 + Ax^2 + x` over the same field with `A = 40962`. This value of `A` is the smallest integer such that `(A - 2) / 4` is a small integer, `A^2 - 4` is nonsquare in `GF(q)`, and the Montgomery curve and its quadratic twist have small cofactors `8` and `4`, respectively. This is identical to the relationship between Curve25519 and ed25519.

Please see [./doc/evidence/](./doc/evidence/) for supporting evidence that Jubjub meets the [SafeCurves](https://safecurves.cr.yp.to/index.html) criteria. The tool in [./doc/derive/](./doc/derive/) will derive the curve parameters via the above criteria to demonstrate rigidity.

## Acknowledgements

Jubjub was designed by Sean Bowe. Daira Hopwood is responsible for its name and specification. The security evidence in [./doc/evidence/](./doc/evidence/) is the product of Daira Hopwood and based on SafeCurves by Daniel J. Bernstein and Tanja Lange. Peter Newell and Daira Hopwood are responsible for the Jubjub bird image.

Please see `Cargo.toml` for a list of primary authors of this codebase.

## License

Licensed under either of

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in the work by you, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms or
conditions.


### Benchmark & some conclusions
```
cargo bench --all
```

with 4G RAM and 1 CPU. The bench results below shows some conclusions that:
* `invert` and `sqrt` arithmetic is time expensive, need to be avoided;
* Arithmetic in scalar field `Fr` is cheaper than in base field `Fq`;
* AffinePoint `add` and  `subtract` is cheaper than ExtendedPoint, so it's worthy to converte `ExtendedPoint` into `AffinePoint`.


```
root@zyd-VirtualBox:/home/zyd/jubjub# cargo bench --all
    Updating `git://mirrors.ustc.edu.cn/crates.io-index` index
   Compiling byteorder v1.3.2
   Compiling rand_core v0.4.0
   Compiling subtle v2.1.0
   Compiling rand_core v0.3.1
   Compiling rand_xorshift v0.1.1
   Compiling jubjub v0.2.0 (/home/zyd/jubjub)
    Finished release [optimized] target(s) in 2m 55s
     Running target/release/deps/jubjub-c7e405832863014f

running 56 tests
test find_curve_generator ... ignored
test find_eight_torsion ... ignored
test fq::test_addition ... ignored
test fq::test_debug ... ignored
test fq::test_equality ... ignored
test fq::test_from_bytes ... ignored
test fq::test_from_bytes_wide_maximum ... ignored
test fq::test_from_bytes_wide_negative_one ... ignored
test fq::test_from_bytes_wide_r2 ... ignored
test fq::test_from_raw ... ignored
test fq::test_from_u512_max ... ignored
test fq::test_from_u512_r ... ignored
test fq::test_from_u512_r2 ... ignored
test fq::test_from_u512_zero ... ignored
test fq::test_inv ... ignored
test fq::test_inversion ... ignored
test fq::test_invert_is_pow ... ignored
test fq::test_multiplication ... ignored
test fq::test_negation ... ignored
test fq::test_sqrt ... ignored
test fq::test_squaring ... ignored
test fq::test_subtraction ... ignored
test fq::test_to_bytes ... ignored
test fq::test_zero ... ignored
test fr::test_addition ... ignored
test fr::test_debug ... ignored
test fr::test_equality ... ignored
test fr::test_from_bytes ... ignored
test fr::test_from_bytes_wide_maximum ... ignored
test fr::test_from_bytes_wide_negative_one ... ignored
test fr::test_from_bytes_wide_r2 ... ignored
test fr::test_from_raw ... ignored
test fr::test_from_u512_max ... ignored
test fr::test_from_u512_r ... ignored
test fr::test_from_u512_r2 ... ignored
test fr::test_from_u512_zero ... ignored
test fr::test_inv ... ignored
test fr::test_inversion ... ignored
test fr::test_invert_is_pow ... ignored
test fr::test_multiplication ... ignored
test fr::test_negation ... ignored
test fr::test_sqrt ... ignored
test fr::test_squaring ... ignored
test fr::test_subtraction ... ignored
test fr::test_to_bytes ... ignored
test fr::test_zero ... ignored
test test_affine_niels_point_identity ... ignored
test test_assoc ... ignored
test test_batch_normalize ... ignored
test test_d_is_non_quadratic_residue ... ignored
test test_extended_niels_point_identity ... ignored
test test_is_identity ... ignored
test test_is_on_curve_var ... ignored
test test_mul_consistency ... ignored
test test_serialization_consistency ... ignored
test test_small_order ... ignored

test result: ok. 0 passed; 0 failed; 56 ignored; 0 measured; 0 filtered out

     Running target/release/deps/fq_bench-6c7b5dc2e0d03f3f

running 6 tests
test bench_add_assign    ... bench:           6 ns/iter (+/- 180)
test bench_invert        ... bench:      28,521 ns/iter (+/- 9,700)
test bench_mul_assign    ... bench:          97 ns/iter (+/- 77)
test bench_sqrt          ... bench:     222,943 ns/iter (+/- 158,988)
test bench_square_assign ... bench:          77 ns/iter (+/- 57)
test bench_sub_assign    ... bench:          12 ns/iter (+/- 7)

test result: ok. 0 passed; 0 failed; 0 ignored; 6 measured; 0 filtered out

     Running target/release/deps/fr_bench-17c4dbd8e26f6c53

running 6 tests
test bench_add_assign    ... bench:           6 ns/iter (+/- 65)
test bench_invert        ... bench:      26,573 ns/iter (+/- 20,452)
test bench_mul_assign    ... bench:          35 ns/iter (+/- 225)
test bench_sqrt          ... bench:      29,672 ns/iter (+/- 15,429)
test bench_square_assign ... bench:          27 ns/iter (+/- 204)
test bench_sub_assign    ... bench:           5 ns/iter (+/- 49)

test result: ok. 0 passed; 0 failed; 0 ignored; 6 measured; 0 filtered out

     Running target/release/deps/point_bench-6c765194de47c8b2

running 7 tests
test bench_cached_affine_point_addition    ... bench:         696 ns/iter (+/- 493)
test bench_cached_affine_point_subtraction ... bench:         685 ns/iter (+/- 537)
test bench_cached_point_addition           ... bench:         741 ns/iter (+/- 569)
test bench_cached_point_subtraction        ... bench:         769 ns/iter (+/- 513)
test bench_point_addition                  ... bench:       1,181 ns/iter (+/- 1,070)
test bench_point_doubling                  ... bench:         691 ns/iter (+/- 548)
test bench_point_subtraction               ... bench:         470 ns/iter (+/- 2,806)

test result: ok. 0 passed; 0 failed; 0 ignored; 7 measured; 0 filtered out
```