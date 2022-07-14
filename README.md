# BigInt bit length for JavaScript
ECMAScript Stage-0 Proposal. J. S. Choi, 2021.

* **[Specification][]**
* **Babel plugin**: Not yet

[specification]: http://jschoi.org/21/es-bigint-math/

## Problem space
BigInts are important for a myriad of
mathematical, financial, scientific, and timing applications
(such as in the [Node.js `process.hrtime.bigint` API][hrtime]),
and they have been therefore a valuable addition to JavaScript
since their standardization in ES 2021.

[hrtime]: https://nodejs.org/api/process.html#process_process_hrtime_bigint

Because their length is theoretically unbound (unlike ordinary JavaScript
Numbers), the bit lengths of their binary representations are an essential
characteristic. They are particularly useful for calculating the log2 – for
example, [Waldemar Horwat’s integer-roots][] uses them to calculate the square
and cube roots of BigInts.

In other languages, BigInt bit lengths are provided by convenient
property/method APIs:

| Precedent      | Form                         | Negative-int behavior          | Return type                         |
| -------------- | ---------------------------- | ------------------------------ | ----------------------------------- |
| **[Python][]** | `i.bit_length()`             | Absolute value of input        | Bignum integer                      |
| **[Java][]**   | `i.bitLength()`              | Two’s complement sans sign bit | 32-bit signed integer               |
| **[.NET][]**   | `BigInteger.GetBitLength(i)` | Two’s complement sans sign bit | 64-bit signed integer               |
| **[Dart][]**   | `i.bitLength`                | Two’s complement sans sign bit | `int` type (width depends on build) |

[Python]: https://docs.python.org/3/library/stdtypes.html#int.bit_length
[Java]: https://docs.oracle.com/en/java/javase/18/docs/api/java.base/java/math/BigInteger.html#bitLength()
[.NET]: https://docs.microsoft.com/en-us/dotnet/api/system.numerics.biginteger.getbitlength?view=net-6.0#system-numerics-biginteger-getbitlength
[Dart]: https://api.dart.dev/stable/2.17.6/dart-core/BigInt/bitLength.html

In JavaScript, we currently have to [use `.toString` to count the BigInts’
digits][toString]. This is weird and inefficient for such a fundamental
operation. We therefore propose exploring the addition of a BigInt bit-length
API to the JavaScript language.

[toString]: https://stackoverflow.com/questions/54758130/how-to-obtain-the-amount-of-bits-of-a-bigint

If this proposal is approved for Stage 1, then we would explore various
directions for the API’s design. We would also examine cross-cutting concerns
with other language features both standardized and proposed, such as the
[BigInt Math proposal][].

## Potential solutions
The solution would probably be a computed property or method on the
BigInt.prototype object or a static function on the BigInt constructor –
something like one of the following:

```js
(0n).bitLength // Like in Dart and Python.
(0n).bitLength() // Like in Java.
(0n).getBitLength()
BigInt.bitLength(0n)
BigInt.getBitLength(0n) // Like in .NET.
```

It would probably return the number of bits in the minimal two’s-complement
representation of any input BigInt, excluding the sign bit. For positive
BigInts, this is equivalent to the number of bits in their ordinary binary
representations. This is also roughly equivalent to `ceil(log2(value < 0 ?
-value : value + 1))`. (See [issue #3][] for discussion about other options.)

The resulting bit length might also be a BigInt, or it might be an ordinary
Number if we are fine with limiting BigInts to be less than `2n **
BigInt(Number.MAX_SAFE_INTEGER)`. (In practice, this already happens; [V8
already caps BigInts to no more than `1<<30` bits, and Firefox already caps at
about `1<<20` bits][already capped]. The question thus is whether such a limit
should be enshrined in the specification. See [issue #2][].)

[Waldemar Horwat’s integer-roots]: https://github.com/waldemarhorwat/integer-roots

In contrast to other operations – such as `abs` and `sqrt` from the [BigInt
Math proposal][] – the new bit-length API would probably be useful only on
BigInts and not on ordinary numbers. We therefore are not plannign to add it
to the Math object. (But see [issue #4][] for more discussion.)

[BigInt Math proposal]: https://github.com/tc39/proposal-bigint-math
[already capped]: https://github.com/tc39/proposal-bigint-math/issues/21#issuecomment-1180917488
[issue #2]: https://github.com/js-choi/proposal-bigint-bit-length/issues/2
[issue #3]: https://github.com/js-choi/proposal-bigint-bit-length/issues/3
[issue #4]: https://github.com/js-choi/proposal-bigint-bit-length/issues/4
