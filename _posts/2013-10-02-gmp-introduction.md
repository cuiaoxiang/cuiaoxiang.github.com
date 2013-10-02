---
layout: post
title: "How to use GMP"
description: ""
category:
tags: [C/C++, gmp]
---
{% include JB/setup %}

[GMP](gmplib.org)(GNU Multiple Precision Arithmetic Library) is a portable library written in C for
arbitrary precision arithmetic on **integers**, **rational numbers**, and **floating-point
numbers**. It aims to provide the fastest possible arithmetic for all applications that need high
precision. There is no theoretical limitation on the precision, except impeded by memory
availability.

The main target applications for GMP are cryptography applications and research, Internet
applications, algebra systems, computational research, etc.

As you may know, several advanced languages like Java, Python innately support high precision
integer/floating number calculations. The Java BigInteger/BigDecimal class may be one of the most
important reasons for ACMers to solve a problem with Java. Unfortunately, they are naively
implemented without considering time efficiency (try multiplying two 100000 digits integers in Java
and Python, see how painfully slow to get the outcome).

The efficient multiplication for high precision numbers should be implemented by some version of
[FFT](http://en.wikipedia.org/wiki/Fast_Fourier_transform). For serious tasks involving high
precision calculation, one needs a professional tool. That's how GMP comes into being. The GMP is
implemented in a very low level with all kinds of optimization applied to achieve the ultimate time
and memory efficiency. If efficiency is the top concern, you should definitely use GMP.

In this article, I will give some C/C++ codes to show how to use GMP library. The examples will
mainly invovle high precision integer arithmetic. Although GMP is highly advanced, you will find
that it is quite simple to use it with C/C++. Especially when you use the *class* version of GMP in
C++, these high precision numbers are almost like primitive types.

### Install GMP on Ubuntu

Select packages and install `sudo apt-cache search libgmp`

After installing the gmp docs, emacs users can view GMP manual via info `C-h i`.

![emacs gmp manual]({{ site.url }}/assets/img/emacs-gmp-help.png)

For non-experts, the above fool-proof process is enough. If you want to further adjust install
arguments to best suit your system, consult manual.

### Headers and libs

To write C/C++ codes with gmp, all you need is to include header files `#include <gmp.h>` or
`#include <gmpxx.h>`

To build, link with the proper library
`gcc prog.c -lgmp`
or
`g++ prog.cc -lgmp -lgmpxx`.

If static library is prefered

```gcc prog.c /usr/lib/libgmp.a```
or ```g++ prog.cc /usr/lib/libgmp.a /usr/lib/libgmpxx.a```

###Types
* `mpz_t` -- high precision integer
* `mpq_t` -- high precision rational number
* `mpf_t` -- high precision floating number

The GMP types like `mpz_t` are pointers, once the memory is allocated, GMP takes care of all space
allocation. Additional space is allocated whenever size need to expand. Allocated space never
shrinks.

### High precision integer operations

The following code only list a fraction of the myriads of arithmetic functions and utility
functions.

{% highlight c++ %}
#include <stdio.h>
#include <gmp.h>

int main() {
  /* allocate space, initialize its value to 0. */
  mpz_t a, b;
  mpz_init(a);
  mpz_init(b);

  /* set values */
  mpz_set(a, b);
  mpz_set_si(a, -100);
  mpz_set_ui(a, 100);
  mpz_set_d(a, 123.0);

  /* allocate space and set value in one shot. */
  mpz_init_set_str(a, "12345678987654321", 10); /* base 10 */
  mpz_init_set(a, b);
  mpz_init_set_si(a, -100);
  mpz_init_set_ui(a, 100);
  mpz_init_set_d(a, 123.0);

  /* conversion, if OP is too big, the result maybe meaningless. */
  mpz_set(a, -100);
  long si = mpz_get_si(a); /* si = -100 */
  mpz_set(a, 100);
  unsigned long ui = mpz_get_ui(a); /* ui = 100 */
  mpz_set_d(a, 123.0);
  double d = mpz_get_d(a); /* d = 123.0 */
  char result[100];
  mpz_get_str(result, 10, a);

  /* arithmetic */
  mpz_t c;
  mpz_add(c, a, b); /* c = a + b */
  mpz_add_ui(c, a, 10);
  mpz_sub(c, a, b); /* c = a - b */
  mpz_sub_ui(c, a, 10);
  mpz_ui_sub(c, 10, a);
  mpz_mul(c, a, b); /* c = a * b */
  mpz_mul_si(c, a, -100);
  mpz_mul_ui(c, a, 100);
  mpz_addmul(c, a, b); /* c = c + a * b */
  mpz_addmul_ui(c, a, 100);
  mpz_submul(c, a, b); /* c = c - a * b */
  mpz_submul(c, a, 100);
  mpz_neg(c, a); /* c = -a */
  mpz_abs(c, a); /* c = |a| */

  /* formatted input & output */
  gmp_scanf("%Zd", c);
  gmp_printf("%Zd\n", c);

  /* clear memory */
  mpz_clear(a);
  mpz_clear(b);
  mpz_clear(c);
  return 0;
}
{% endhighlight %}

Manipulating these high precision objects with C++ class is much more convinient. We don't bother to
allocate & free memories, and the arithemtic operations are written in a more straight-forward way.
However, it is not as efficiency as its C counterpart since the C++ class interface is just a
wrapper of the underlying C implementation. In fact, C++ version is not fully comparable to C
version, because some C implementations do not have the corresponding C++ interface.

{% highlight c++ %}
#include <iostream>
#include <gmpxx.h>

using namespace std;

int main() {
  mpz_class a, b, c;
  a = 1234;
  b = "-5678";
  c = a + b;
  cout << "sum is " << c << endl;
  cout << "absolute value is " << abs(c) << endl;
  return 0;
}
{% endhighlight %}

### Example 1: Advanced version of Gauss's story

Some time ago, there is a sick [joke](http://www.guokr.com/post/291823/) widely spread on the
internet. The question little Gauss faced is as follows

**Given `p/q = 1+1/2+1/3+...+1/100000`, with `p`, `q` reduced. How many trailing zeros in
  denominator `q`?**

There is an intricate mathematical solution to this problem. If you want to check the solution, or
just want to know how the astronomical number `q` looks like

{% highlight c++ %}
  mpq_class ret = 0;
  for (int i = 1; i <= 100000; ++i) {
    ret += mpq_class(1, i);
  }
  cout << ret << endl;
{% endhighlight %}

### Example 2: Binomial coefficient

{% highlight c++ %}
  for (i = 0; i < N; ++i) {
    mpz_init_set_si(C[i][0], 1);
    mpz_init_set_si(C[i][i], 1);
    for (j = 1; j < i; ++j) {
      mpz_init2(C[i][j], i);
      mpz_add(C[i][j], C[i - 1][j - 1], C[i - 1][j]);
    }
  }
  gmp_printf("%Zd\n", C[N - 1][N / 2]);
  for (i = 0; i < N; ++i) {
    for (j = 0; j <= i; ++j) {
      mpz_clear(C[i][j]);
    }
  }
{% endhighlight %}

{% highlight c++ %}
  for (int i = 0; i < N; ++i) {
    C[i][0] = C[i][i] = 1;
    for (int j = 1; j < i; ++j) {
      C[i][j] = C[i - 1][j - 1] + C[i - 1][j];
    }
  }
  cout << C[N - 1][N / 2] << endl;
{% endhighlight %}

On my laptop (Ubuntu 10.04, CPU 2.0Hz), with `N=2000`, the above two code snippets run `886ms`(C)
and `1426ms`(C++) respectively.

### Example 3: Coefficients in a series

This example is exerpted from a real case when I helped my college to check a theorem in his thesis
some years ago. He asked me to calculate the first 135,000 coefficients in a series related to some
modular form. Back then, I don't even know GMP, so I spend one hour to write a C++ code fast enough
to finish checking these large coefficients within one day. If only I known GMP, the coding would
take me about 20 minutes with more time efficiency and less error prone.

{% highlight c++ %}
#include <gmp.h>
#include <stdio.h>

const int N = 20000;
const int M = 1000;
mpz_t A[N], B[N];

unsigned int max(unsigned int x, unsigned int y) {
  return x > y ? x : y;
}

void polynomial_multiply(mpz_t A[], int n) {
  for (int i = N - 1; i >= n; --i) {
    mpz_sub(A[i], A[i], A[i - n]);
  }
}

int main() {
  freopen("series.out", "w", stdout);
  mpz_array_init(A[0], N, M);
  mpz_array_init(B[0], N, M);
  mpz_set_si(A[0], 1);
  for (int i = 1; i < N; ++i) {
    polynomial_multiply(A, i);
    polynomial_multiply(A, i);
    polynomial_multiply(A, i);
  }
  for (int i = 3; i < N; i += 3) {
    polynomial_multiply(A, i);
  }
  mpz_set_si(B[0], 1);
  for (int i = 1; i < N; ++i) {
    mpz_set_si(B[i], 0);
    for (int j = 1; j <= i; ++j) {
      mpz_submul(B[i], A[j], B[i - j]);
    }
  }
  for (int i = 0; i < N; ++i) {
    gmp_printf("%5d: %Zd\n", i, B[i]);
  }

  unsigned int max_size = 0;
  for (int i = 0; i < N; ++i) {
    max_size = max(max_size, mpz_size(A[i]));
    max_size = max(max_size, mpz_size(B[i]));
  }
  printf("%u\n", max_size);
  return 0;
}
{% endhighlight %}
