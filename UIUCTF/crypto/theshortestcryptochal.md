# Challenge Name : `the shortest crypto chal`

- **Category:** Cryptography
- **Points:** 50  
- **Author:** epistemologist

---

##  Description

> I've designed the shortest crypto challenge - can you find the flag?

Well, it did live up to its name.

---

## Files Provided

- A python code : `chal.py`

The only actual line of code in this cute little challenge was 
```python
assert a**4 + b**4 == c**4 + d**4 + 17 and max(a,b,c,d) < 2e4 and AES.new( f"{a*b*c*d}".zfill(16).encode() , AES.MODE_ECB).encrypt(FLAG).hex() == "41593455378fed8c3bd344827a193bde7ec2044a3f7a3ca6fb77448e9de55155"
```

---

## What Do We Get From This

Let's split it into three parts - 

- ```a**4 + b**4 == c**4 + d**4 + 17``` 

    This gives us a nice little relation between four numbers

- ```max(a,b,c,d) < 2e4``` 

    And a very helpful constraint to go with it

- ```AES.new( f"{a*b*c*d}".zfill(16).encode() , AES.MODE_ECB).encrypt(FLAG).hex() == "41593455378fed8c3bd344827a193bde7ec2044a3f7a3ca6fb77448e9de55155"``` 

    And here is where the actual encryption takes place. Now we know that AES is a secure mode of encryption, so the only hope we have of recovering this is to recover that key

--- 

## The Maths
Well now we don't really know any of the values beforehand, and straight up bruteforcing 4 different values(limited at 2e4) sounds like a pain(it is).

So, how do we go about this?

Let's look at it this way -
- I want a sum of 2 quartics (anything^4)
- And then I want another sum of 2 quartics
- And the difference between these is 17.

So now what I'm going to do is, instead of finding combinations of 4 numbers, I am going to start storing sums of 2 quartics, since I need these for both sides of the comparisons. Once I have calculated all the sums in a range, I will go individually to each quartic sum `x` and check if `x-17` is also a valid quartic sum.

Once I have the correct values that had been used as the key, I can simply decrypt it

---

## Code

```python
from Crypto.Cipher import AES

# First we calculate all quartic sums in the range of 1000 integers and the corresponding numbers used for it
# (I took a smaller number cos it was taking forever lol, if the solution did not fall in this range, it would take a few minutes at worst)
sums = {}
for i in range(1000):
    for j in range(i,1000):
        sums[i**4+j**4] = [i,j]
                
# Now we check if any two have a difference of 17 and store the numbers responsible in my candidates list
# (There are probably a hundred more efficient ways to check, but ig the range is small enough for the complexity gods to forgive my sins on this one)
candidates = []
for x in sums.keys():
    if (x-17) in sums.keys():
        candidates.append(sums[x]+sums[x-17])
print("Possible candidates -",candidates) 
       
# And finally, we try and decrypt the ciphertext using the same key
for candidate in candidates:
    a,b,c,d = candidate
    pt = AES.new( f"{a*b*c*d}".zfill(16).encode() , AES.MODE_ECB).decrypt(bytes.fromhex("41593455378fed8c3bd344827a193bde7ec2044a3f7a3ca6fb77448e9de55155"))
    if b'uiuctf' in pt:
        print(pt)
        break
```

---

## Result

```
Possible candidates - [[1, 2, 0, 0], [264, 651, 530, 570]]
--------------------------
Found - b'uiuctf{D1oPh4nTine__Destr0yer__}'
Using numbers - 264 651 530 570
```

---

## Possible Learnings

Now this challenge was obviously aimed at beginners, so here's a few things you can take away from here - 
- A quick thumb rule you can use to identify if your code complexity is too high is the fact that your laptop/PC can probably do around 10^9 or 2^32 computations in 1 second. If i tried bruteforcing all 4 numbers, it would have a complexity of (2e4)^4 which is of the order of 10^17(bad!!!)
- As I mentioned before, AES is secure. Especially if it is not constructed manually for the challenge. Seeing an AES encryption is an indicator that there is nothing you can do about the encryption process itself, try to find a weaknesses around it (in this case, the key).
- This method has a few similarities to a very useful cryptographic attack called [Meet in the Middle](https://www.baeldung.com/cs/security-meet-in-the-middle-attack). Check it out, it's a very sweet and easy to understand optimization.

---