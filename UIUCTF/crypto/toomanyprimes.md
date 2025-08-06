# Challenge Name : `too many primes`

- **Category:** Cryptography
- **Points:** 50  
- **Author:** epistemologist

---

##  Description

> Normal RSA uses two primes - that's too few in my opinion, so I've added some more.

<p align = "center">	<img width="805" height="548" alt="{4588E2EC-2272-4DC0-ADC0-EFB560262C7F}" src="https://github.com/user-attachments/assets/0c44f7ab-b1b3-44dd-8ed3-1d9a97b0a66e" /> </p>


---

## Files Provided

- A python code : `chal.py`

The relevant code -
```python
p = randprime(2**127, 2**128)
N = 1
while N < 2**2048:
	N *= p
	p = nextprime(p)

assert gcd(phi_N, 65537) == 1

pt = bytes_to_long(FLAG)
ct = pow(pt, 65537, N)
print("N = ", N)
print("ct = ", ct)
# N =  34546497157207880069779144631831207265231460152307441189118439470134817451040294541962595051467936974790601780839436065863454184794926578999811185968827621504669046850175311261350438632559611677118618395111752688984295293397503841637367784035822653287838715174342087466343269494566788538464938933299114092019991832564114273938460700654437085781899023664719672163757553413657400329448277666114244272477880443449956274432819386599220473627937756892769036756739782458027074917177880632030971535617166334834428052274726261358463237730801653954955468059535321422372540832976374412080012294606011959366354423175476529937084540290714443009720519542526593306377
# ct =  32130352215164271133656346574994403191937804418876038099987899285740425918388836116548661879290345302496993945260385667068119439335225069147290926613613587179935141225832632053477195949276266017803704033127818390923119631817988517430076207710598936487746774260037498876812355794218544860496013734298330171440331211616461602762715807324092281416443801588831683678783343566735253424635251726943301306358608040892601269751843002396424155187122218294625157913902839943220894690617817051114073999655942113004066418001260441287880247349603218620539692362737971711719433735307458772641705989685797383263412327068222383880346012169152962953918108171850055943194

```

---

## What Do We Get From This

THE BIGGEST RED FLAG - `p = nextprime(p)`

And the slightly smaller red flag - `p = randprime(2**127, 2**128)`

--- 

## The Maths
A small overview of RSA for those who need it. We require -

- A Message - `m`
- Two Primes (usually atleast 1024 bits long) - `p,q`
- A Public Key - `e`    (most commonly set to 65537)

How does it work?
- We define `n=p*q`
- We also define the [Euler Totient Function](https://www.geeksforgeeks.org/dsa/eulers-totient-function/) of `n`, `phi = (p-1)*(q-1)`
- We generate a [modular inverse](https://www.geeksforgeeks.org/dsa/multiplicative-inverse-under-modulo-m/) `d = e^-1 (mod phi)`

- The ciphertext is given by `c = m^e (mod n)`
- And decrypting the ciphertext is done by `m = c^d (mod n)`

Here, `n` and `e` are made public, along with the ciphertext `c`. Everything else is private. 

So you may ask me, "Pastime, if I give out these important values, how exactly is RSA secure?"

The most important part is the calculation of the private key `d` using `phi(n)` (which i have represented as just `phi`). What makes RSA secure is the fact that you need the factors of `n` to calculate `phi`, which in turn is a necessity to calculate `d`. It is extremely hard to factor huge numbers (If the factors themselves are ~1000 bits), which pretty much guarantees that if anybody has the value of `n`, they would still not be able to calculate `d`.

This challenge differs from the standard RSA system since it has more than 2 prime factors for `n`, but that just means that our formula for `phi` will be different. The rest of the process stays the same.


Now here comes the part which makes us able to break this,

`p = randprime(2**127, 2**128)`

This is much smaller than the standard 1024 bit numbers and that implies it is very highly likely that this number's factors have been found. A very nice place to check for factors of big numbers is [factordb.com](factordb.com). Plugging in the given `n` gives us its prime factorization, but honestly, I only need the first one, I can get the rest like in the given code.

From there, I keep finding the next primes as long as they divide `n`.

The new `phi` can be represented as `(p1 - 1)*(p2 - 1)*(p3 - 1)...` (Check out the link above to understand why exactly it equates to that). Once we have the new `phi`, we can proceed with the usual steps of RSA.

---

## Code

```python
from sympy import nextprime
from Crypto.Util.number import *

n = 34546497157207880069779144631831207265231460152307441189118439470134817451040294541962595051467936974790601780839436065863454184794926578999811185968827621504669046850175311261350438632559611677118618395111752688984295293397503841637367784035822653287838715174342087466343269494566788538464938933299114092019991832564114273938460700654437085781899023664719672163757553413657400329448277666114244272477880443449956274432819386599220473627937756892769036756739782458027074917177880632030971535617166334834428052274726261358463237730801653954955468059535321422372540832976374412080012294606011959366354423175476529937084540290714443009720519542526593306377
nn=n        # Made a copy of n since I need it later
primes = [242444312856123694689611504831894230373]  # First factor I got from factordb

while True:
    n=n//primes[-1]                         # keep updating n to check if I need more primes 
    if n==1:
        break
    primes.append(nextprime(primes[-1]))    # keep adding the next primes

phi = 1
for i in primes:
    phi *= (i-1)                            # calculate phi as discussed 
    

e = 65537
d = pow(e,-1,phi)
c = 32130352215164271133656346574994403191937804418876038099987899285740425918388836116548661879290345302496993945260385667068119439335225069147290926613613587179935141225832632053477195949276266017803704033127818390923119631817988517430076207710598936487746774260037498876812355794218544860496013734298330171440331211616461602762715807324092281416443801588831683678783343566735253424635251726943301306358608040892601269751843002396424155187122218294625157913902839943220894690617817051114073999655942113004066418001260441287880247349603218620539692362737971711719433735307458772641705989685797383263412327068222383880346012169152962953918108171850055943194

print("Flag - ",long_to_bytes(pow(c,d,nn)))           # AAAAAND  *insert drumroll*, WE'RE IN
```

---

## Result

```
Flag -  b'uiuctf{D0nt_U5e_Cons3cUt1vE_PriMeS}'
```

---

## Possible Learnings

Now this challenge was clearly aimed at beginners, so here's a few things you can take away from here - 
- The key to solving this challenge was understanding how RSA works internally. I would suggest taking a little time to grasp the math behind it properly, cos once you know **how** and **why** it works, it's like looking into a transparent machine.
- Sort of adding on to the last point, also pay attention to why something is used the way it is. For example, this challenge is much more trivial if you know that we need to use **large** primes for RSA vs just knowing you need to use primes.
- And yeah, obviously, you get a new free site that might help you in future challenges - [factordb.com](factordb.com)


---
