---
layout: post
title: "Bits of tricks"
description: ""
tags: [C/C++, bit]
---
{% include JB/setup %}
#### GCC built in functions

GCC provides a large number of built-in functions. Some of them are quite efficient and extremely
convenient in practice.

`__builtin_ctz(x)` count trailing zeros in `x`

`__builtin_ctzll(x)` count trailing zeros in `x`

**Note:** the result is undefined if `x` is zero.

`__builtin_popcount(x)` the number of 1-bits in `x`

`__builtin_popcountll(x)` the number of 1-bits in `x`

`__builtin_pairty(x)` the number of 1-bits in `x` modulo 2

`__builtin_pairtyll(x)` the number of 1-bits in `x` modulo 2

#### integer lg2
{% highlight c++ %}
int lg2(int x) {
  return !x ? -1 : 31 - __builtin_clz(x);
}
{% endhighlight %}

#### basic tricks
`-x` is equivalent to `(~x) + 1`

`x & (x - 1` clear the last 1-bit

`x & -x` clear all bits except the last 1-bit

`x | (x + 1)` set the last 0-bit to 1-bit

`x | (~x - 1)` set all bits to 1 excpet the last 0-bit

For me, the most frequently used one is `x & -x` as it is building block of *index tree*'s update
and query.

{% highlight c++ %}
void update(int x, int y) {
  for (; x < n; x += x & -x) {
    c[x] += y;
  }
}

int query(int x) {
  int ret = 0;
  for (; x; x -= x & -x) {
    ret += c[x];
  }
}
{% endhighlight %}

#### reverse bits
{% highlight c++ %}
inline int reverse_bits(int x){
  x = ((x >> 1) & 0x55555555) | ((x << 1) & 0xaaaaaaaa);
  x = ((x >> 2) & 0x33333333) | ((x << 2) & 0xcccccccc);
  x = ((x >> 4) & 0x0f0f0f0f) | ((x << 4) & 0xf0f0f0f0);
  x = ((x >> 8) & 0x00ff00ff) | ((x << 8) & 0xff00ff00);
  x = ((x >>16) & 0x0000ffff) | ((x <<16) & 0xffff0000);
  return x;
}

inline LL reverse_bits(LL x){
  x = ((x >> 1) & 0x5555555555555555LL) | ((x << 1) & 0xaaaaaaaaaaaaaaaaLL);
  x = ((x >> 2) & 0x3333333333333333LL) | ((x << 2) & 0xccccccccccccccccLL);
  x = ((x >> 4) & 0x0f0f0f0f0f0f0f0fLL) | ((x << 4) & 0xf0f0f0f0f0f0f0f0LL);
  x = ((x >> 8) & 0x00ff00ff00ff00ffLL) | ((x << 8) & 0xff00ff00ff00ff00LL);
  x = ((x >>16) & 0x0000ffff0000ffffLL) | ((x <<16) & 0xffff0000ffff0000LL);
  x = ((x >>32) & 0x00000000ffffffffLL) | ((x <<32) & 0xffffffff00000000LL);
  return x;
}
{% endhighlight %}

Sets can be represented by integers where each bit represent an element in the set. There are some
extremely fast routines to iterate the whole set.

#### iterate all subsets of a given set
{% highlight c++ %}
for (int i = mask; i; i = (i - 1) & mask) {
  ...
}
{% endhighlight %}

#### iterate subsets with fixed size
{% highlight c++ %}
int next(int x) {
  int a = x & -x;
  x += a;
  return (x ^ x - a) / 4 / a | x;
}
{% endhighlight %}



