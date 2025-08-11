# Challenge Name : `back to roots`

- **Category:** Cryptography
- **Points:** 50  
- **Author:** epistemologist (lol he made all the challenges)

---

##  Description

> I don't think you can predict my secret number from just its square root - can you prove me wrong?

https://www.youtube.com/watch?v=ulNswX3If6U&list=RDulNswX3If6U&t=54s

---

## Files Provided

- A python code : `chal.py`

The relevant code -
```python
K = randint(10**10, 10**11)
print('K', K)
leak = int( str( Decimal(K).sqrt() ).split('.')[-1] )

print(f"leak = {leak}")
ct = AES.new(
	md5(f"{K}".encode()).digest(),
	AES.MODE_ECB
).encrypt(pad(FLAG, 16))

print(f"ct = {ct.hex()}")
```
- A text file : `output.txt`

```
leak = 4336282047950153046404
ct = 7863c63a4bb2c782eb67f32928a1deceaee0259d096b192976615fba644558b2ef62e48740f7f28da587846a81697745
```
---

## What Do We Get From This

The decimal portion of the square root of our **VERY SECRET KEY**

--- 

## The Maths

Remember the beautiful little formula taught to us back in middle school? 

<p align = "center"> $(a + b)^2 = a^2 + b^2 + 2ab$ </p>

"When will we ever use this in real life?"  **HERE!!**

We know that our key is between $10^{10}$ and $10^{11}$, so its root must be between $10^5$ and $3.2*10^5$.

We can EASILY go through all of these values in less than a second. So we start picking values for the integer part of this square root, pair it up with the given fractional part, and try to use it as the key to decrypt the ciphertext.

Optional, but a further optimization you can do is to check if the square will be a perfect integer before you use it in the AES decryption.

---

## Code

```python
from Crypto.Cipher import AES
from Crypto.Util.number import *
from hashlib import md5

def poss(a):
    dec = 0.4336282047950153046404
    val = dec**2 + a**2 + 2*a*dec
    if val%1<0.000001:                    # Check if the square is close enough to an integer
        return dec+a
    else:
        return -1

i = 10**5
while True:
    i+=1
    K = round(poss(i)**2)                 # Get the original key value
    if K!=1:
        pt = AES.new(md5(f"{K}".encode()).digest(),AES.MODE_ECB).decrypt(long_to_bytes(0x7863c63a4bb2c782eb67f32928a1deceaee0259d096b192976615fba644558b2ef62e48740f7f28da587846a81697745))
        if b'uiuctf' in pt:                     # Just a final check to ensure nothing went wrong
            print("Flag - ",pt.decode())
            exit() 

```

---

## Result

```
Flag -  uiuctf{SQu4Re_Ro0T5_AR3nT_R4nD0M}
```

---

<p align="center"> <img width="1165" height="865" alt="{AAA0FF22-C901-4410-8DAF-CC3E1EB1036D}" src="https://github.com/user-attachments/assets/f4dcb863-2318-48dc-87ec-d5f7ef41b52c" /> </p>


