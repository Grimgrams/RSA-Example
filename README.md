# RSA-Example

This document is used to explain some code for a math assignment. The code below is an example of [RSA Encryption](https://en.wikipedia.org/wiki/RSA_(cryptosystem)).

## Table Of Contents

- [Finding the GCD](#GCD)
- [Public Key](#GenKeys)
- [Encrypting](#Enc)

## Entire Code

```c
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>
#include <math.h>
 
int e, d, n;
 
int gcd(int a, int b)
{
    int q, r1, r2, r;
 
    if (a > b)
    {
        r1 = a;
        r2 = b;
    }
    else {
        r1 = b;
        r2 = a;
    }
 
    while (r2 > 0)
    {
        q = r1 / r2;
        r = r1 - q * r2;
        r1 = r2;
        r2 = r;
    }
 
    return r1;
}
 
int PrimarityTest(int a, int i)
{
    int n = i - 1;
    int k = 0;
    int j, m, T;
 
    while (n % 2 == 0)
    {
        k++;
        n = n / 2;
    }
 
    m = n;
    T = FindT(a, m, i);
 
    if (T == 1 || T == i - 1) {
        return 1;
    }
 
    for (j = 0; j < k; j++)
    {
        T = FindT(T, 2, i);
        if (T == 1) {
            return 0;
        }
        if (T == i - 1) {
            return 1;
        }
    }
    return 0;
}
 
int FindT(int a, int m, int n)
{
    int r;
    int y = 1;
 
    while (m > 0)
    {
        r = m % 2;
        FastExponention(r, n, &y, &a);
        m = m / 2;
    }
    return y;
}
 
int FastExponention(int bit, int n, int* y, int* a)
{
    if (bit == 1) {
        *y = (*y * (*a)) % n;
    }
 
    *a = (*a) * (*a) % n;
}
 
int inverse(int a, int b)
{
    int inv;
    int q, r, r1 = a, r2 = b, t, t1 = 0, t2 = 1;
 
    while (r2 > 0)
    {
        q = r1 / r2;
        r = r1 - q * r2;
        r1 = r2;
        r2 = r;
 
        t = t1 - q * t2;
        t1 = t2;
        t2 = t;
    }
 
    if (r1 == 1) {
        inv = t1;
    }
 
    if (inv < 0) {
        inv = inv + a;
    }
 
    return inv;
}
 
int KeyGeneration()
{
    int p, q;
    int phi_n;
 
    do {
        do
            p = rand();
        while (p % 2 == 0);
 
    } while (!PrimarityTest(2, p));
 
    do {
        do
            q = rand();
        while (q % 2 == 0);
    } while (!PrimarityTest(2, q));
 
    n = p * q;
    phi_n = (p - 1) * (q - 1);
 
    do
        e = rand() % (phi_n - 2) + 2; // 1 < e < phi_n
    while (gcd(e, phi_n) != 1);
 
    d = inverse(phi_n, e);
}
 
int Encryption(int value, FILE* out)
{
    int cipher;
    cipher = FindT(value, e, n);
    fprintf(out, "%d ", cipher);
}
 
int Decryption(int value, FILE* out)
{
    int decipher;
    decipher = FindT(value, d, n);
    fprintf(out, "%c", decipher);
}
 
int main(void)
{
    FILE *inp, *out;
 
    // destroy contents of these files (from previous runs, if any)
    out = fopen("cipher.txt", "w+");
    fclose(out);
    out = fopen("decipher.txt", "w+");
    fclose(out);
 
    KeyGeneration();
 
    inp = fopen("plain.txt", "r+");
    if (inp == NULL)
    {
        printf("Error opening Source File.\n");
        exit(1);
    }
 
    out = fopen("cipher.txt", "w+");
    if (out == NULL)
    {
        printf("Error opening Destination File.\n");
        exit(1);
    }
 
    // encryption starts
    while (1)
    {
        char ch = getc(inp);
        if (ch == -1) {
            break;
        }
        int value = toascii(ch);
        Encryption(value, out);
    }
 
    fclose(inp);
    fclose(out);
 
    // decryption starts
    inp = fopen("cipher.txt", "r");
    if (inp == NULL)
    {
        printf("Error opening Cipher Text.\n");
        exit(1);
    }
 
    out = fopen("decipher.txt", "w+");
    if (out == NULL)
    {
        printf("Error opening File.\n");
        exit(1);
    }
 
    while (1)
    {
        int cip;
        if (fscanf(inp, "%d", &cip) == -1) {
            break;
        }
        Decryption(cip, out);
    }
    fclose(out);
 
    return 0;
}
```

## <a name="GCD"></a> GCD

```c
int gcd(int a, int b)
{
    int q, r1, r2, r;
 
    if (a > b)
    {
        r1 = a;
        r2 = b;
    }
    else {
        r1 = b;
        r2 = a;
    }
 
    while (r2 > 0)
    {
        q = r1 / r2;
        r = r1 - q * r2;
        r1 = r2;
        r2 = r;
    }
 
    return r1;
}

```
- The code starts by checking if the first integer is greater than the second, in which case it sets r1 to be equal to a and r2 to be equal to b.
If not, then it sets r1 to be equal to b and r2 to be equal
The code computes the greatest common divisor of two integers.

## <a name="GenKeys"></a> Creating the Keys

```c
int KeyGeneration()
{
    int p, q;
    int phi_n;
 
    do {
        do
            p = rand();
        while (p % 2 == 0);
 
    } while (!PrimarityTest(2, p));
 
    do {
        do
            q = rand();
        while (q % 2 == 0);
    } while (!PrimarityTest(2, q));
 
    n = p * q;
    phi_n = (p - 1) * (q - 1);
 
    do
        e = rand() % (phi_n - 2) + 2; // 1 < e < phi_n
    while (gcd(e, phi_n) != 1);
 
    d = inverse(phi_n, e);
}

- This is a function that generates random integers.
- The function starts by generating two random integers, p and q.
- It then calculates the value of phi_n using the formula: phi_n = (p - 1) * (q - 1); The do-while loop iterates until it has calculated all values of phi_n from 2 to n-1.
- Then it calculates d which is the inverse of phi_n modulo e where e is one less than the number of iterations in the loop.
- The code attempts to generate a prime number.
- The while loop iterates until the value of e is greater than or equal to phi_n.
- Inverse(phi_n, e) calculates the inverse of phi_n modulo e.
```

### <a name="Enc"></a> Encrypting The Data

```c
int Encryption(int value, FILE* out)
{
    int cipher;
    cipher = FindT(value, e, n);
    fprintf(out, "%d ", cipher);
}
 
int Decryption(int value, FILE* out)
{
    int decipher;
    decipher = FindT(value, d, n);
    fprintf(out, "%c", decipher);
}
```
