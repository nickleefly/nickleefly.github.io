---
layout: post
title: "Karatsuba Multiplication and big O"
date: 2020-01-15 22:00
comments: true
categories: algorithm karatsuba
---

In an effort to fill my knowledge on algorithm, I recently watched [1. Integer Arithmetic, Karatsuba Multiplication][1]

What is the Karatsuba Algorithm?

> a classic divide and conquer algorithm

from [Karatsuba algorithm wiki page][2],

```
x = a* B^m + b

y = c* B^m + d

x * y = B^m(ac) + B^m/2(ad+bc) + bd
```

Now, instead of multiplying 1234 x 5678 the long way, you can do this:

```
a = 12
b = 34
c = 56
d = 78
n = 4
base of 10
x * y = 10^n(ac) + 10^(n/2)(ad + bc) + bd
x * y = 10^4 * (672) + 10^2 * (2840) + 2652
x * y = 7006652
```

Karatsuba's basic step works for any base B and any m, but the recursive algorithm is most efficient when m is equal to n/2, rounded up.

To elucidate, for integers with n digits, m is the ceiling of half n (m being the exponent applied to the Base in the algorithm). This can be seen if you simply look at the definition of the algorithm.

In particular, if n is 2^k, for some integer k, and the recursion stops only when n is 1, then the number of single-digit multiplications is 3^k, which is n^c where c = log3


some derivation details
```
n = 2^k
=> logn = log2^k
=> logn = k * log2
=> logn = k

Verify n^log3 = 3^logn
Define x = n^log3
=> logx = log3 * logn
=> logx = logn * log3
=> logx = log3^logn
hence x = 3^logn and we get 3^logn = n^log3
```

for n digit number,
```
T(n) = n^c = 3^k = 3^logn
=> T(n) = n^log3
and c = log3 = 1.585
```

Python code

```
def karatsuba(num1, num2):
    if len(str(num1)) == 1 or len(str(num2)) == 1:
        return num1 * num2
    else:
        # Calculates the size of the numbers
        m = max(len(str(num1)), len(str(num2)))
        m2 = m // 2

        # Split the digit sequences in the middle
        high1 = num1//10**m2
        low1 = num1%10**m2
        high2 = num2//10**m2
        low2 = num2%10**m2

        # 3 calls made to numbers approximately half the size.
        z0 = karatsuba(low1, low2)
        z1 = karatsuba((low1 + high1), (low2 + high2))
        z2 = karatsuba(high1, high2)

        return (z2 * 10 ** (2**m2)) + ((z1 - z2 - z0) * 10 ** m2) + z0
```

More information, please refer to [Karatsuba Multiplication Stanford University Coursera][3]

[1]: https://www.youtube.com/watch?v=eCaXlAaN2uE&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb&t=2034s
[2]: https://en.wikipedia.org/wiki/Karatsuba_algorithm
[3]: https://www.youtube.com/watch?v=ZCtB7I3i6vk
