# Practical 7: Booth's Multiplication Algorithm in 8086 Assembly (MASM)

## Aim

To implement Booth's multiplication algorithm for two binary numbers using 8086 assembly language and display the product.

## Requirements

- DOSBox (or another DOS environment)
- MASM assembler and LINK utility
- A text editor to create the `.ASM` file

## Program

Save the following as **`BOOTH.ASM`**. GitHub automatically adds a **Copy** button to this code block.

```asm
.MODEL SMALL
.STACK 100H

.DATA
M       DB 6                ; Multiplicand
Q       DB 5                ; Multiplier
A       DB 0                ; Accumulator
Q_1     DB 0                ; Previous multiplier bit
COUNT   DB 8                ; Number of iterations
RESULT  DW ?

MSG1 DB 13,10,'BOOTH MULTIPLICATION ALGORITHM$'
MSG2 DB 13,10,'Multiplicand = 6$'
MSG3 DB 13,10,'Multiplier = 5$'
MSG4 DB 13,10,'Result = $'

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX

    ; Display heading and inputs
    LEA DX, MSG1
    MOV AH, 09H
    INT 21H

    LEA DX, MSG2
    MOV AH, 09H
    INT 21H

    LEA DX, MSG3
    MOV AH, 09H
    INT 21H

    ; Initialize Booth's algorithm
    MOV A, 0
    MOV Q_1, 0
    MOV COUNT, 8

BOOTH_LOOP:
    ; Read current Q0
    MOV AL, Q
    AND AL, 01H

    ; Compare Q0 and Q-1
    CMP AL, Q_1
    JE SHIFT

    ; Q0 = 1, Q-1 = 0: subtract M
    CMP AL, 01H
    JE SUBTRACT

    ; Q0 = 0, Q-1 = 1: add M
    MOV AL, A
    ADD AL, M
    MOV A, AL
    JMP SHIFT

SUBTRACT:
    MOV AL, A
    SUB AL, M
    MOV A, AL

SHIFT:
    ; Save current Q0 as next Q-1
    MOV AL, Q
    AND AL, 01H
    MOV Q_1, AL

    ; Save the old least-significant bit of A
    MOV AL, A
    AND AL, 01H
    MOV BL, AL

    ; Shift Q right by one bit
    MOV AL, Q
    SHR AL, 1
    MOV Q, AL

    ; Copy old A0 into the most-significant bit of Q
    CMP BL, 01H
    JNE NO_MSB
    MOV AL, Q
    OR AL, 80H
    MOV Q, AL

NO_MSB:
    ; Arithmetic shift right for A (preserve sign)
    MOV AL, A
    SAR AL, 1
    MOV A, AL

    DEC COUNT
    JNZ BOOTH_LOOP

    ; Combine A and Q into one 16-bit result
    MOV AL, Q
    MOV AH, A
    MOV RESULT, AX

    ; Display product
    LEA DX, MSG4
    MOV AH, 09H
    INT 21H

    MOV AX, RESULT
    CALL PRINT_NUMBER

    MOV AH, 4CH
    INT 21H
MAIN ENDP

; Print the unsigned decimal number in AX
PRINT_NUMBER PROC
    MOV BX, 10
    XOR CX, CX

CONVERT:
    XOR DX, DX
    DIV BX
    PUSH DX
    INC CX
    CMP AX, 0
    JNE CONVERT

DISPLAY:
    POP DX
    ADD DL, '0'
    MOV AH, 02H
    INT 21H
    LOOP DISPLAY

    RET
PRINT_NUMBER ENDP

END MAIN
```

## How to Run in DOSBox

1. Put `BOOTH.ASM`, `MASM.EXE`, and `LINK.EXE` into a Windows folder such as `C:\8086`.
2. Open DOSBox and enter:

```dos
MOUNT C C:\8086
C:
MASM BOOTH.ASM;
LINK BOOTH.OBJ;
BOOTH.EXE
```

3. View the output in the DOSBox window.

## Expected Output

```text
BOOTH MULTIPLICATION ALGORITHM
Multiplicand = 6
Multiplier = 5
Result = 30
```

**Verification:** `6 * 5 = 30` in decimal (`001EH` in hexadecimal).

## Theory

Booth's algorithm multiplies signed binary numbers in two's-complement form using an **accumulator (`A`)**, **multiplier (`Q`)**, **multiplicand (`M`)**, and an extra bit **`Q-1`**. For each multiplier bit, the algorithm chooses an operation from the pair `(Q0, Q-1)`:

| `(Q0, Q-1)` | Operation |
| :--: | :-- |
| `00` | No addition or subtraction |
| `01` | Add `M` to `A` |
| `10` | Subtract `M` from `A` |
| `11` | No addition or subtraction |

After each step, the combined bits `(A, Q, Q-1)` undergo an **arithmetic right shift**. This process is repeated eight times for the given 8-bit operands. The final 16-bit product is formed by combining `A` (high byte) and `Q` (low byte).

**Note:** The supplied printing procedure displays the result as an **unsigned** decimal number. The shown example uses positive numbers, so its output is correct; displaying negative products as signed decimal values would need an additional conversion step.

## Conclusion

Booth's multiplication algorithm was implemented in 8086 assembly language. The program successfully multiplies `6` and `5` and displays the result `30`, demonstrating the use of bit checks, addition/subtraction, and arithmetic shifts.
