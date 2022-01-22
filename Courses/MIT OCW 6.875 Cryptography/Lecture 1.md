1. Perfect Secrecy/Shannon Secrecy -

For all messages m belonging to all probability distribution of messages M, a scheme has perfect secrecy if P(m) == P(m|c) i.e. even if the adversary has seen the ciphertext, the probability of the plaintext being m stays the same for the adversary as it was before it saw the ciphertext.

2. Perfect distinguishability

For any pair of messages m and m', even if the adversary sees the ciphertext c, it cannot decide if the message was m or m' i.e. P(m|c) == P(m'|c)

3. Perfect secrecy and perfect indistinguishability are equivalent.

4. Shannon's Theorem proves that for an encryption scheme (G,E,D) to have perfect secrecy the size of the Key Space of the scheme must be gerater than the size of it's Message Space.