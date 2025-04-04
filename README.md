# Many-Time Pad Cryptanalysis

This project demonstrates a classic cryptographic attack against stream ciphers when the same key is reused multiple times (a "many-time pad").

## The Problem

A stream cipher encrypts plaintext by XORing it with a pseudorandom keystream. A One-Time Pad (OTP) is a type of stream cipher where the key is truly random and used only once; it provides perfect secrecy.

However, if the same keystream (key) is used to encrypt multiple different plaintexts (`P1`, `P2`, ...), the security breaks down completely.

Let `K` be the reused key and `C1`, `C2` be two ciphertexts:
*   `C1 = P1 XOR K`
*   `C2 = P2 XOR K`

If an attacker obtains both `C1` and `C2`, they can XOR them together:
*   `C1 XOR C2 = (P1 XOR K) XOR (P2 XOR K)`
*   `C1 XOR C2 = P1 XOR P2 XOR K XOR K`
*   `C1 XOR C2 = P1 XOR P2` (Since `K XOR K = 0`)

The attacker now has the XOR sum of the two plaintexts, without knowing the key. While this doesn't immediately reveal `P1` or `P2`, it provides significant information, especially if multiple ciphertexts (`C1`, `C2`, ..., `Cn`) encrypted with the same key are available.

## The Attack (`many_time_pad_solver.py`)

The Python script `many_time_pad_solver.py` implements a statistical attack to recover one of the plaintexts (specifically, the 11th one provided in the problem description) given multiple ciphertexts encrypted with the same key.

**Core Idea:**

1.  **XOR Ciphertexts:** XOR the target ciphertext (`C_target`) with every other ciphertext (`Ci`). This yields `P_target XOR Pi` for each `i`.
2.  **Character Guessing:** For each byte position `j` in the target plaintext (`P_target`):
    *   Iterate through all possible byte values (0-255) as a guess for `P_target[j]`.
    *   For each guess, calculate the corresponding byte `Pi[j]` for all other plaintexts using the formula: `Pi[j] = guess_P_target[j] XOR (C_target[j] XOR Ci[j])`.
    *   **Scoring:** Evaluate the "plausibility" of the resulting `Pi[j]` bytes across all `i`. The script assumes the plaintexts are primarily English text. It assigns higher scores to guesses that result in common English letters, spaces, and punctuation across the other plaintexts.
    *   **Space Hint:** The script incorporates the hint that XORing a space (`0x20`) with a letter flips its case. It gives a score boost if guessing `P_target[j]` as a space results in a high proportion of letters in the calculated `Pi[j]` values.
3.  **Best Guess:** Select the byte value for `P_target[j]` that yields the highest total plausibility score across all other plaintexts.
4.  **Repeat:** Repeat this process for every byte position up to the length of the shortest ciphertext.

## Running the Code

1.  Make sure you have Python 3 installed.
2.  Run the script from your terminal:
    ```bash
    python many_time_pad_solver.py
    ```

## Expected Output

The script will print the recovered plaintext for the target ciphertext. Due to the statistical nature of the attack and potential variations in the original plaintexts, minor errors might occur in the recovered text.

The expected recovered message is:
```
The secret message is: When using a stream cipher, never use the key more than once
```

The script's output will look similar to this:

```
Decryption successful up to 83 bytes.
------------------------------
Recovered Plaintext:
The secuet message is: Whon using a stream cipher, never use the key more than once
------------------------------
The likely secret message is: "The secuet message is: Whon using a stream cipher, never use the key more than once"
```
(Note the minor typos "secuet" and "Whon" which are artifacts of the statistical method).

This clearly demonstrates why a keystream **must never** be reused in a stream cipher. 