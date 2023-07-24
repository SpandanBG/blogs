# Primality Test with O(√n) Complexity 

---

---

The importance of prime numbers comes during cryptography. Modern encryption systems, like RSA, used in web technologies, depend on the fact that prime factorization of large numbers is huge overhead. Two person sharing a prime number can exchange messages secretly without the outside world knowing the content. But how do you check if a number is prime or not?

**The naive way**  is to check if any number from *2 *to* p-1*, where *p* is the number to be tested, completely divides *p*. That is:

<p style="text-align: center;">
 <code>p mod a  =  0;   where a ε [2,p)</code>
</p>

This technique involves going from *2* to *p-1*, checking for every number if the above condition is met. That means, if the size of the prime number is *2^(18)* length long, it would take *2^(18)-2* loops to check through all the numbers.

A sample program:

```c
/*
    group="Naive Way"
    title="C"
*/

#include<stdio.h>

int main(){
    int p, i, j=0;
    scanf("%d",&amp;p);

    if(p%2 !=0){                    // Since even numbers cannot be prime
        j=1;
        for(i=3; i<p; i++){        // Since for 2 is already checked
                if(p%i==0){
                    j=0;
                    break;
                }
        }
    } else if (p == 2) {        // If the number entered is 2
        j = 1;
    }

    if(j){
        printf("%d is a prime\n");
    } else {
        printf("%d is not a prime\n");
    }
    return 0;
}
```

```python
# group="Naive Way" 
# title="Python3"

#!/usr/bin/python3
p = int(input())
flag = False

if p%2 is not 0:                    # Since even numbers cannot be prime
    flag = True
    for a in range(3,p-1):
        if p%a is 0:
            flag= False
            break
elif p is 2:                            # If entered number is 2
    flag= True

if flag is True:
    print("%d is a prime"%(p))
else:
    print("%d is not a prime"%(p))
```

**The O(√n) solution** involves cutting down the number of loops to *√n*, where *n* is the number to be checked for primality. The idea behind it is pretty simple. Suppose we get a number *n* from a machine that creates random prime numbers. At first our range of numbers to divide *n* in order to check its primality is from *2 to n-1*. Now, let dividing *n* by *2* we get the number *x*. Then, *2* multiplied to the number *x* or a greater number will result in a number greater than or equal to *n*, i.e. `a*2=b, where if a>=x then b>=n`. Therefore, any number between *x *to* n* can be disregarded as a possible candidate that can completely divide *n*. This will result in our range's upper limit to shrink from *n-1* to *x-1*, that means, there cannot be any number between the range *[x,n)* which can completely divide *n* (i.e. disproves n's primality). Similarly, when *n* will be divided by *3*, we'll get another number, suppose *y*, which will mark the new limit of our range; since any number from y onward cannot be a candidate the disproves *n*'s primality.

As we keep dividing *n* by the numbers in range *[2,n)*; we first divide by *2* and get *x*, that means if we add *x* twice we get a number close to or equal to *n*. Then we divide by *3* and get *y*, which when added thrice we get close to or equal to *n*. Continuing, we will get the number *a* such that *a* added *a* times we get close to or equal to *n*, i.e. `a^(2) ~= n`. Therefore, after *a*, any number greater than it will result in a reverse trend, i.e. the number *(a+1)* when added *(a-1)* times will get us close to or equal to *n*, and the number *2* when added *x* times will result in close to or equal to *n*.

Therefore, the upper limit of the range can be marked by: * a = √n*

The program of O(√n) time complexity will be:

```c
/*
    group="O(root(n))"
    title="C"
*/

#include<stdio.h>
#include<math.h>

int main(){
    int p, c, i, j=0;
    scanf("%d",&amp;p);
    c = floor(sqrt(p));    // Getting the upper limit of the range

    if(p%2 != 0){        // Since even numbers cannot be prime
        j=1;
        for(i=3; i<=c; i++){
            if(p%i == 0){        // If the no. `p` is divisible by `c`
                j=0;                 // then it's not a prime
                break;
            }
        }
    } else if (p == 2) {       // If p entered is 2
        j=1;
    }

    if(j){
        printf("%d is a prime\n",p);
    } else {
        printf("%d is not a prime\n",p);
    }
    return 0;
}
```

```python
# group="O(root(n))"
# title="Python3"

#!/usr/bin/python3
import math

p = in(it())
c = math.floor(ath.sqrt(p))           # Getting the upper limit of the range
flag =False

if p%2 is not 0:                 # Since even numbers cannot be prime
    flag=True
    for a in range(3,c):
            if p%a is 0:         # If `p` is divisible by `a` then `p` is not a prime
                flag=False
                break
elif p is 2:                     # If p entered is 2
    flag=True
else:
    flag=False

if flag is True:
    print("%d is a prime",%(p))
else:
    print("%d is not a prime",%(p))
```

## Bonus

*Fermat's Little Theorem of Primality* states that if *p* is a prime and *1 < a < p*, then

<p style="text-align: center;"><em>a^(p) - a = x, where x is divisible by p<em></p>

With this simple test we can easily find out if a number is prime or not. For example, let's take the number *p=13* and let *a=2* then

<p style="text-align: center;"><em>x = 2^(13) - 2 = 8190</em></p>

And since *13 * 630 = 8190*, therefore *13* is a prime number.
Let's check another number again for fun. Let *p=561* and *a=2*, then

<p style="text-align: center;"><em>x = 2^(561) - 2 = 7547924849643082704483109161976537781833842440832880856752412600491248324784297704172253450355317535082936750061527689799541169259849585265122868502865392087298790653950</em></p>

This ridiculously large number is divisible by *561* (thanks to Python, I could do the calculations). Therefore the number *561* should be a prime number, right? Actually, *561 = 51 * 11 or 17*3*11*, which means *561 is a composite number* and is lying to be a prime number. Such numbers are known as *Carmichael numbers* (Fermat liars) and there are infinitely many of them. These Carmichael numbers are rarer than prime numbers, thus probability of Fermat liars occurring is *1/4*.

Thus, Fermat's Little Theorem can be termed as *Probabilistic Primality Test*. The probability can be improved by checking if the number is a Carmichael number, the time complexity to check which is *O(log(n)^(4))*. Other extensions of Fermat's tests include *Baillie-PSW*, *Miller-Rabin* and *Solovay-Strassen*. Libgcrypt is a cryptography library that implements Fermat test with base 2.

To have a quicker primality check, we can implement Fermat Little Theorem at constant time with a random number between *[2,p)*, where *p* is the prime number, as *a* and performing the check. If however, the need of lowering the error occurs, we can perform the check for all numbers in the range *[2,p)* which increase the time complexity to *O(n)*.

A sample program with constant time primality test:

```python
# group="Fermat Little"
# title="Python3"

#!/usr/bin/python3
p = int(input())
if p%2 is not 0 and p is not 2:
    a = (2**p - 2)%p
    if a is 0:
        print("%d is a prime"%(p))
    else:
        print("%d is not a prime"%(p))
elif p is 2:
        print("%d is a prime"%(p))
else:
        print("%d is not a prime"%(p))
```

---

Spandan Buragohain,
2017-04-12 04:30:57
