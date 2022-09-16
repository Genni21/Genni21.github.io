---
layout: post
title: RC5 without rotations
category: Cryptography
date: 2022-09-16 22:00:39 +0200
math: true
---

This post concerns the first cryptanalytic attack on Bruce Schneier's Self Study Course on Block Cipher Cryptanalysis.

The RC5 cipher is quite an old cipher, nowadays noticably less used than before due to the rise of AES. One of the strenght of RC5, for didactic purposes, is its simplicity. 

The RC5 cipher starts with a Key Expansion algorithm, which takes in a key, divides it into words, and outputs an array of the sort
$$S[0], S[1], ... , S[n]$$, where all of the elements of the array $$S$$ are words. For now I will avoid how the array is constructed from the key.

Once the array $$S$$ is built, one can call the encryption and decryption functions. The encrytion function takes in two words, $$A$$ and $$B$$, as inputs, and works as follows (^ is the xor operator):

{% highlight python %}
A = A + S[0]
B = B + S[1]
for i in range(1, r+1): #r is the number of rounds
	A = ((A ^ B) << B) + S[2*i]
	B = ((A ^ B) << A) + S[2*i+1]
return A, B
{% endhighlight %}

It is worth nothing that A and B are storing only 32 bits, so the additions are concretely modular additions, where the modulus is $$2^{32}$$, equivalent to 
using a binary AND operator with the value $$2^{32} - 1$$.

The first target I will try to break is 8 rounds RC5 without rotations, so there is no left/right shift of bits at all.
Here follows the (pseudo)code for it:

{% highlight python %}

A = A + S[0]
B = B + S[1]
for i in range(1, r+1): #r is the number of rounds
	A = (A ^ B) + S[2*i]
	B = (A ^ B) + S[2*i+1]
return A, B

{% endhighlight %}

At first, I thought there was a full break on this, linear in the number of rounds, but I wasn't really thinking straight, and I thought I could somehow
just walk back the cipher bruteforcing only the last round subkey.
Reading online, nothing really made sense, when, after a while, I came across a guy saying "combinations of LSBs". At this point everything clicked, I can try all of the 
combinations of the LSBs of A and B, and see which one matches the output. For some reason, my mind didn't instantly go to it when I read "lookup table" in a previous link,
it probably has to do with the fact that I didn't think that my lookup table would be surjective, but in hindsight I should've expected this because both XOR and addition 
don't lose any information.

Notice that the only carry is upwards, this means that only the lower x bits of the plaintext influence the lower x bits of the ciphertext.
Let $$A$$, $$B$$ be our 2 32-bit words of plaintext, and let $$C$$, $$D$$ be our 2 32-bit words of ciphertext. Let also $$A^{'} = A \pmod{2^i}$$, 
$$B^{'} \equiv B \pmod{2^i}$$, $$C^{'} \equiv C \pmod{2^i}$$, $$D^{'} \equiv D \pmod{2^i}, \forall i<32$$. We know the following (because of exclusive upward
carry):

$$C^{'} = f(A^{'}, B^{'}), \textrm{ for some function }f$$

This basically just means that the lowest n bits of the output are determined only by the lowest bits of the input. We can basically build an algorithm
from this, that allows to retrieve the original plaintext with just $$4w$$ calls to the encryption oracle (where $$w$$ is the bit length of a single word, in this case 32 bits) by just checking which of the 4 combination of two bits produces the actual ciphertext bits.

{% highlight python %}

from random import getrandbits
NROUNDS = 8

"""
S=[]
for i in range(18):
    S.append(getrandbits(32))
#S is spat out to be the following
"""
S=[374668424, 1378453119, 257466301, 3304611826, 1239658714, 4175194812, 1559293949, 1663552235, 1551517565, 2635359816, 3334028342, 1308450419, 377743761,
        1278263930, 1949520292, 1889111537, 649193607, 1806516393]

def encrypt(A, B):
    A = A + S[0]
    B = B + S[1]
    for i in range(1, NROUNDS+1):
        A = ((A ^ B) + S[2*i]) % (2**32)
        B = ((A ^ B) + S[2*i +1]) % (2**32)

    return A, B

A = getrandbits(32)
B = getrandbits(32)
o1, o2 = encrypt(A, B)

tmpA, tmpB = 0, 0
for i in range(32):
    for [x, y] in ([0, 0], [1, 0], [0, 1], [1, 1]): #for all combinations of the i-th LSB
        tmp1, tmp2 = encrypt(tmpA + 2**i*x, tmpB + 2**i*y) # compute encryption thanks to the encryption oracle
        if tmp1 % 2**(i+1) == o1 % 2**(i+1) and tmp2 % 2**(i+1)==o2 % 2**(i+1): # if the lowest bits match, keep x and y as bits of the original pt
            tmpA = tmpA + 2**i*x
            tmpB = tmpB + 2**i*y
            break

assert(tmpA==A and tmpB==B)

{% endhighlight %}
