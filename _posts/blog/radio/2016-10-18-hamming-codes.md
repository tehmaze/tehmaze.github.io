---
layout: post
title: "Hamming Codes"
categories: [blog,radio]
share: true
comments: true
excerpt: A coders view on implementing Hamming linear error-correcting codes.
tags: [radio,dmr,coding]
author: maze
image:
  feature: /posts/radio.jpg
date: 2016-01-14T20:37:00+01:00
mathjax: true
hidden: true
---

There are a [lot][] [of][] [good][] [articles][] on the *mathemathical* side of
implementing **Hamming Codes**, but less has been written on how to implement
them efficiently in code.

[lot]: http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-02-introduction-to-eecs-ii-digital-communication-systems-fall-2012/lecture-slides/MIT6_02F12_lec04.pdf
[of]: http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-02-introduction-to-eecs-ii-digital-communication-systems-fall-2012/lecture-slides/MIT6_02F12_lec05.pdf
[good]: http://orion.math.iastate.edu/linglong/Math690F04/HammingCodes.pdf
[articles]: http://www.inference.phy.cam.ac.uk/mackay/itprnn/ps/17.21.pdf

Hamming codes are a family of binary linear error-correcting codes. For each
integer $$ r \ge 2 $$ there is a code with block length
$$ n = (2^r) - 1 $$ and message length $$ k = (2^r) - r - 1 $$.

In my particular case, I was looking at the `Hamming(15, 11, 3)` ande
`Hamming(13, 9, 3)` codes that are a variation on the `Hamming(16, 11, 4)` code,
as part of the [DMR Air Interface specification][].

> The `Hamming(n, k, d)` notation means:
>
> --|-----------------
> n | Block length
> k | Message length
> d | Hamming distance
>
> Or in plain English, a `Hamming(n, k, d)` code encodes `k` bits of data into
> `n` bits by adding `d` parity bits.

# A closer look at *Hamming \(15, 11, 3\)* {#hamming-15-11-3}

With the following constants:

$$
n = 15
$$

$$
k = 11
$$

$$
d = 3
$$

The generator polynomial for the primitive code is as follows:

$$
G(x) = x^4 + x + 1 = 23_{8}
$$

The following $$k \times n$$ generator matrix is supplied in the documentation:

$$
H^m = \left[\begin{array}{ccccccccccc|cccc}
1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 \\
0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 0 & 1 \\
0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 & 1 \\
0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 & 0 \\
0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 \\
0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 1 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 & 1 & 1 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 & 1 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 & 1
\end{array}\right]
$$

Which translates into the following code in C:

{% highlight c %}
static uint16_t hamming_15_11_3_gen[11] = {
	0x4009, /* 10000000000|1001 */
	0x200d, /* 01000000000|1101 */
	0x100f, /* 00100000000|1111 */
	0x080e, /* 00010000000|1110 */
	0x0407, /* 00001000000|0111 */
	0x020a, /* 00000100000|1010 */
	0x0105, /* 00000010000|0101 */
	0x008b, /* 00000001000|1011 */
	0x004c, /* 00000000100|1100 */
	0x0026, /* 00000000010|0110 */
	0x0013  /* 00000000001|0011 */
};

{% endhighlight %}

From this we can derive the following parities:

$$
H^{''} = \begin{bmatrix}
1 & 1 & 1 & 1 & 0 & 1 & 0 & 1 & 1 & 0 & 0 \\
0 & 1 & 1 & 1 & 1 & 0 & 1 & 0 & 1 & 1 & 0 \\
0 & 0 & 1 & 1 & 1 & 1 & 0 & 1 & 0 & 1 & 1 \\
1 & 1 & 1 & 0 & 1 & 0 & 1 & 1 & 0 & 0 & 1
\end{bmatrix}
$$

Let's quickly test checking a message (with length $$ k $$):

$$
D_{1xk} \times G_{kxn} = C_{1xn}
$$

Or:

$$
Output = \begin{bmatrix}
D_{1} & D_{2} & ... & D_{11}
\end{bmatrix} \times \left[\begin{array}{ccccccccccc|cccc}
1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 \\
0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 0 & 1 \\
0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 & 1 \\
0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 & 0 \\
0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 \\
0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 1 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 & 1 & 1 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 & 1 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 & 1
\end{array}\right] = \begin{bmatrix}
D_{1} & D_{2} & ... & D_{11} & P_{1} & P_{2} & P_{3} & P_{4}
\end{bmatrix}
$$

Or:

{% highlight c %}
uint16_t hamming_15_11_3_encode(uint16_t in)
{
	uint16_t i, out = 0;
    /* For each of the 11 bits */
	for (i = 0; i < 11; i++) {
        /* If the bit is set */
		if (in & (1 << (10 - i))) {
            /* XOR bit with our generator matrix */
			out ^= hamming_15_11_3_gen[i];
		}
	}
	return out;
}
{% endhighlight %}



[DMR Air Interface specification]: https://www.etsi.org/deliver/etsi_ts/102300_102399/10236101/02.02.01_60/ts_10236101v020201p.pdf
