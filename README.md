# solution-crackme3
Solution using Binary Ninja for crackme/5b7dd53233c5d441d87ccbef by @rextco

https://crackmes.one/crackme/5b7dd53233c5d441d87ccbef

Find a valid password.

## Binary Ninja

![checkMagicSquare](https://github.com/user-attachments/assets/9ddaab09-3ca4-4eaa-9974-8aa24a3ba03e)

### Strings

![strings1](https://github.com/user-attachments/assets/4520aa10-4015-4165-8715-1938eac02354)

Observed and renamed the following strings.

*Notable:*
- ```Nope!```
- ```You rock, now write a tutorial```

```
0040600c  char Instructions2[0x17] = "Get a valid password\r\n", 0

00406024  char Instructions3[0x23] = "usage:\tcrackme-3.exe <password>\r\n\n", 0

00406048  char Instructions4[0x21] = "crackme-3 by @rextco - for x86\r\n", 0

0040606c  char SuccessMessage[0x21] = "You rock, now write a tutorial\r\n", 0

00406090  char SuccessMessage2[0x2d] = "and join to [+] https://t.me/crackslatinos\r\n", 0

004060c0  char FailMessage[0x8] = "Nope!\r\n", 0
```
![strings2](https://github.com/user-attachments/assets/1025cc87-7409-46a4-924f-020732c3716d)

### Graph

- Identifyied the Main function in the Symbols view

  ![findMain](https://github.com/user-attachments/assets/9a7388bf-bb69-4471-b43b-3312c2fd2180)
  
- Started graph view and selected the main function
  
  ![graphMain](https://github.com/user-attachments/assets/b4dbf61e-6b30-4487-b9db-b7a432ce5981)

- Followed the function calls to the found strings.
  - sub_401550
    
![j_sub_401550](https://github.com/user-attachments/assets/8c1faae7-2e73-48fb-bd36-633104b0f178)

- Inspected sub_401400 (password check)
    - reference to sub_401550

![CheckArgs_LinearPseudoC](https://github.com/user-attachments/assets/7b12bd44-a2cb-4559-93b3-05e0c934acd2)

### Linear

sub_401400

![checkMagicSquare_LinearPseudoC](https://github.com/user-attachments/assets/c917e2bc-b57a-4d14-8096-436c9d9809d0)

### Password Validation Logic

- This code starts by doing validation on a 16-byte input string (`arg1`).
- The code then parses the input into a magic square and tests its values.

![checkMagicSquare_LinearPseudoC](https://github.com/user-attachments/assets/e9c34e5e-6959-4b50-aafc-2c610d7bb6d5)


---

**Initial Check**
```c
if (strlen(arg1) != 0x10) // 0x10 = 16
    return 0;
```
It first checks that the input string is exactly 16 characters long. If not, it returns `0` (fail).

---

**Row Sum Check**
```c
for (int i = 0; i < 4; i++) {
    int sum = 0;
    for (int j = 0; j < 4; j++) {
        sum += sx.d(arg1[i + (j << 2)]);
    }
    if (sum != 0x1c2) // 0x1c2 = 450
        return 0;
}
```

This is checking the sum of values (after being passed through a function `sx.d()`) **by row**, assuming the 16 characters form a 4x4 grid **in column-major order**.

- `(j << 2)` = `j * 4`, so `i + (j * 4)` steps through vertically (same column).
- So it's summing the **rows** of a 4x4 grid.

If any row’s sum is not 450, return `0`.

---

**Column Sum Check**
```c
for (int i = 0; i < 4; i++) {
    int sum = 0;
    for (int j = 0; j < 4; j++) {
        sum += sx.d(arg1[j + (i << 2)]);
    }
    if (sum != 0x1c2)
        return 0;
}
```

Same idea, but this time it’s iterating the input in such a way that it now sums **columns**. Again, each column must sum to 450 after `sx.d()` conversion.

---

**Diagonal Sum Check**
```c
int diag1 = 0;
int diag2 = 0;

for (int i = 0; i < 4; i++) {
    diag1 += sx.d(arg1[i * 5]);        // top-left to bottom-right (indices 0,5,10,15)
    diag2 += sx.d(arg1[(i + 1) * 3]);  // top-right to bottom-left (indices 3,6,9,12)
}
if (diag1 == 0x1c2 && diag2 == 0x1c2)
    return 1;
```

It calculates both **main diagonals** of the 4x4 matrix. If both diagonals also sum to 450, the check passes and it returns `1`.

---

**Conclusion**

This code:
- Requires the input string to be 16 characters long.
- Treats it as a 4x4 matrix.
- Uses a function `sx.d()` to convert each character (possibly to ASCII or something else).
- Validates:
  - Each row sums to 450
  - Each column sums to 450
  - Both diagonals sum to 450

---

### Generating the Password (Magic Square)

A magic square is a grid of numbers arranged so that the sum of each row, each column and its diagonals are all equal to the same value, called the magic sum (or magic constant).

[Wikipedia - Magic Square](https://en.wikipedia.org/wiki/Magic_square)

[Magic Square Generator](https://www.dcode.fr/magic-square)

```
112	115	118	105
117	106	111	116
107	120	113	110
114	109	108	119
```

---

**Convert the resulting numbers into ASCII char codes**

*I guessed here*

```
p   s   v   i  
u   j   o   t  
k   x   q   n  
r   m   l   w  
```

### Solution

```
psviujotkxqnrmlw  
```
