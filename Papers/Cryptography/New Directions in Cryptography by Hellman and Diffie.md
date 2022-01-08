# New Directions in Cryptography
## by Diffie and Hellman

Quite nice to read a seminal paper. The best part was the simplicity of notations and language.

Learnings -

1. A *cryptographic system* is a set of mappings from the space of plain text to a space of cipher text. Individual members of this set can be parameterized using a security parameter *K*.

2. There are two answers to the question how secure a **secure** cryptographic system is -
| Computationally Secure | Uncondittionally Secure |
|----|----|
| It is possible to break this system given unlimited computaitonal capacity. Such a system is so expensive to break that it is impractical to carry it out.| It is just not possible to break this system even with infinite computation capacity.|

3. Unconditional security results from the presence of multiple plain texts for a given cipher text.

4. The only unconditionally secure scheme is One Time Pad. As proven theoretically by Shannon.

5. Kinds of cryptographic attacks -
   1. Ciphertext only attack - Adversary has a list of ciphertexts
   2. Known Plaintext attack - Adversary has a list of plaintext and ciphertext pairs
   3. Chosen Plaintext attack - Adversary can generate ciphertext for any plaintext

6. A *public key cryptosystem* is a pair of *cryptoraphic systems* {E[k]} and {D[k]} where -

E[k] : {M} -> {M}

D[k] : {M} -> {M}

where {M} is a finite message space, such that
```
1. For every k belonging to {k}, E[k] is an inverse of D[k] i.e. E[k](D[k](x)) == x

2. For every k belonging to {k} and m belonging to {m} E[k] and D[k] are easy to compute for m i.e. they can be done in reasonable space and time, possibly polynomial.

3. For almost every k belonging to {k} it takes infinite time and resource to compute D[k] when E[k] is given

4. For every k belonging to {k} it is easy to compute a pair of E[k] and D[k] from k.
```

> NB- k is not ncessarily the key.

It should take exponential time to compute D[k] from E[k].

7. A system which is known to be secure against a known plaintext attack can be used to build a one way function. Converse is not true.

8. A trap door cipher is one which prevents cryptanalysis unless one is in possession of trap door information.

8. A trap-door cyptosystem can be used to produce a public key distribution system.

9. If y = (pow(alpha, x) mod q) where alpha, x and y are elements of GF(q) and q is a prime then finding x = (log(y, base=alpha) mod q) is called the discrete logarithm problem and is known to be a hard problem. While calculating y from x can take at most 2 * log(y, base=2) steps, calculating the inverse can require pow(q, 1/2) steps, which is exponential in time.

10. When we say a function is easy to compute we mean that it can be done in a reasonable time and space which grows polynommially with the size of inputs, but when we say that a related function is hard to compute then that means it will take an exponential number of steps which is dependent on the size of the input. So a function can be computed in ax time/space, where x is the size of input but calculating the inverse of this function will take pow(b,x) time/space.

12. For a purely random mapping from {x} -> {y} where every x has uniform probability in {x} and the y that will be outputted for the x is also selected with a uniform probability from {y}. Since this mapping is random it is possible that the same y is selected for multiple x i.e. x[1], x[2],...x[k]. The probability that any y has k+1 preimages in {x} is given by pow(e,-1)/(k!) for k = 0,1,2.... This is a poisson distribution of random variable k and the expected value of k comes out to be 2. Since the lambda of this distribution is 1, shifted by 1 unit. That is to say, for a purely random mapping from {x} -> {y}, every y is expected to have two pre images in {x}. Note that it is the expected number of pre-images and not fixed.