# BigInt `Math` for JavaScript
ECMAScript Stage-0 Proposal. J. S. Choi, 2021.

* **[Specification][]**
* **Babel plugin**: Not yet

[specification]: http://jschoi.org/21/es-bigint-math/

## Description
(A [formal draft specification][specification] is available.)

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

* [Java’s BigInteger bitLength method](https://docs.oracle.com/en/java/javase/18/docs/api/java.base/java/math/BigInteger.html#bitLength())
* [Dart’s BigInt bigLength property](https://api.dart.dev/stable/2.17.6/dart-core/BigInt/bitLength.html)
* [.NET’s BigInteger GetBitLength method](https://docs.microsoft.com/en-us/dotnet/api/system.numerics.biginteger.getbitlength?view=net-6.0#system-numerics-biginteger-getbitlength)

In JavaScript, we currently have to [use `.toString` to count the BigInts’
digits][toString]. This is weird and inefficient for such a fundamental
operation. We therefore propose exploring the addition of a BigInt bit-length
API to the JavaScript language.

[toString]: https://stackoverflow.com/questions/54758130/how-to-obtain-the-amount-of-bits-of-a-bigint

If this proposal is approved for Stage 1, then we would explore various
directions for the API’s design. We would also examine cross-cutting concerns
with other language features both standardized and proposed, such as the
[BigInt Math proposal][].

## Description
The solution would probably be a computed property or method on the
BigInt.prototype object or a static function on the BigInt constructor –
something like one of the following:

```js
(0n).bitLength // Like in Dart.
(0n).bitLength() // Like in Java.
(0n).getBitLength() // Like in .NET.
BigInt.bitLength(0n)
BigInt.getBitLength(0n)
```

It would return the number of bits in the minimal two’s-complement
representation of the BigInteger, excluding any sign bit. For positive
BigIntegers, this is equivalent to the number of bits in the ordinary binary
representation. This is also roughly equivalent to `ceil(log2(value < 0 ?
-value : value + 1))`.

The resulting bit length might also be a BigInt, or it might be an ordinary
Number if we are fine with limiting BigInts to be less than `2n **
BigInt(Number.MAX_SAFE_INTEGER)`. (In practice, this already happens; [V8
already caps BigInts to no more than `1<<30` bits, and Firefox already caps at
about `1<<20` bits][already capped]. The question thus is whether such a limit
should be enshrined in the specification.)

[Waldemar Horwat’s integer-roots]: https://github.com/waldemarhorwat/integer-roots

In contrast to other operations – such as `abs` and `sqrt` from the [BigInt
Math proposal][] – the new bit-length API would probably be useful only on
BigInts and not on ordinary numbers. We therefore are not considering adding it
to the Math object.
