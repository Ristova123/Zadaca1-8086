# Zadaca1-8086

За микропроцесорскиот систем реализиран во претходната задача напишете рутина која ќе детектира кога е
притиснат тастер на матричната тастатура, а потоа врши декодирање на вредноста која одговара на тој тастер
(хексадецимална цифра) и ја прикажува вредноста на седум-сегментниот дисплеј. Во случај на грешка, на
дисплејот треба да се прикаже знакот H. Адресата на излезната порта на седум-сегментниот дисплеј е 90h,
излезната порта за тастатурата е 80h, а влезната порта за тастатурата е 81h. 



**Resenie:**

```
DSEG SEGMENT 'DATA'
;|----------------------------------------------------------|
;| 0 1 2 3 4 5 6 7 |
;|----------------------------------------------------------|
TABLE DB 077h, 07Bh, 07Dh, 07Eh, 0B7h, 0BBh, 0BDh, 0BEh
;|----------------------------------------------------------|
;| 8 9 A B C D E F |
;|----------------------------------------------------------|
 DB 0D7h, 0DBh, 0DDh, 0DEh, 0E7h, 0EBh, 0EDh, 0EEh
;|------------------------------------------------|
;| 0 1 2 3 4 5 6 7 |
;|------------------------------------------------|
HEX DB 7Eh, 30h, 6Dh, 79h, 33h, 5Bh, 5Fh, 72h
;|------------------------------------------------|
;| 8 9 A B C D E F |
;|------------------------------------------------|
 DB 7Fh, 7Bh, 77h, 1Fh, 4Eh, 3Dh, 4Fh, 47h
 TAST_OUT EQU 80h
 TAST_IN EQU 81h
 DISPLEJ EQU 90h
DSEG ENDS
SSEG SEGMENT 'STACK'
 DW 30 DUP(?)
 TOP_STACK LABEL WORD
SSEG ENDS
CSEG SEGMENT 'CODE'
 ASSUME CS: CSEG, DS: DSEG, SS: SSEG
START:
 MOV AX, SSEG
 MOV SS, AX
 MOV SP, OFFSET TOP_STACK
 MOV AX, DSEG
 MOV DS, AX
JAMKA:
 CALL KEYBRD ;процедура за читање од тастатура
 CALL PRIKAZ ;процедура за приказ на дисплеј
 JMP JAMKA

KEYBRD PROC NEAR
 PUSHF
 PUSH BX
 PUSH CX
 PUSH DX
 MOV AL, 0 ;прати нули на влез на тастатурата
 OUT TAST_OUT, AL
WAIT_OPEN: ;чекаме да се стабилизира состојбата
 IN AL, TAST_IN
 AND AL, 0Fh ;за избегнување на било каква грешка
 CMP AL, 0Fh
 JNE WAIT_OPEN
WAIT_PRESS: ;чекаме да биде притиснат тастер
 IN AL, TAST_IN ;откога ќе се притисне
 AND AL, 0Fh ;се утврдува кој тастер е притиснат
 CMP AL, 0Fh
 JE WAIT_PRESS
 MOV CX, 16EAh ;генерираме debounce
DELAY: LOOP DELAY ;одредено каснење
 IN AL, TAST_IN ;испитуваме дали и понатаму
 AND AL, 0Fh ;тастерот е притиснат
 CMP AL, 0Fh
 JE WAIT_PRESS
 MOV AL, FEh ;потоа се скенира тастатурата за да се утврди тастерот
 MOV CL, AL
NEXT_ROW:
 OUT TAST_OUT, AL
 IN AL, TAST_IN
 AND AL, 0Fh
 CMP AL, 0Fh
 JNE ENCODE ;кога ќе го најдеме вршиме кодирање на симболот
 ROL CL, 1
 MOV AL, CL
 JMP NEXT_ROW
ENCODE: ;дел кој врши конверзија
 MOV BX, 15 ;на кодот на тастерот во хексадецимален еквивалент
 IN AL, TAST_IN
TRY_NEXT:
 CMP AL, TABLE[BX]
 JZ DONE
 DEC BX
 JNS TRY_NEXT
 MOV AH, 01
 JMP EXIT
DONE: MOV AL, BL
 MOV AH, 00
EXIT: NOP
 NOP
 POP DX
 POP CX
 POP BX
 POPF
 RET
KEYBRD ENDP
PRIKAZ PROC NEAR ;процедура за приказ на хексадецималниот број на дисплејот
 PUSHF
 PUSH BX
 MOV BX, OFFSET HEX
 CMP AH, 00
 JNZ ERROR
 XLAT
 OUT DISPLEJ, AL
 JMP KRAJ
ERROR: MOV AL, 37h ;доколку настанало грешка се прикажува “H” (37h)
 OUT DISPLEJ, AL
KRAJ: NOP
 NOP
 POP BX
 POPF
 RET
PRIKAZ ENDP
CSEG ENDS
 END START 

```
 ![Screenshot (1)](https://github.com/Ristova123/Zadaca1-8086/blob/main/Zadaca1.1%208086%20code.png)
 ![Screenshot (2)](https://github.com/Ristova123/Zadaca1-8086/blob/main/Zadaca1.2%208086%20code.png)
 ![Screenshot (3)](https://github.com/Ristova123/Zadaca1-8086/blob/main/Zadaca1.3%208086%20code.png)
 
**Develop by:**

[Tamara Ristova ](https://github.com/Ristova123)


**Subject**

Microcomputer's systems

**Built With**

This project is built using the following tools:

- [8086 simulator](https://emu8086-microprocessor-emulator.en.softonic.com/?ex=RAMP-2046.0): Assembler and emulator for the Intel 8085 microprocessor.

**Getting Started**

To get a local copy up and running, follow these steps.

**Prerequisites**

In order to run this project you need:

A working computer
Connection to internet
Setup

**How to Run**

To run the program, you need an 8086 emulator or assembler. You can use emulators like DOSBox or TASM (Turbo Assembler). Here's how to run the program using 8086 simulator:

1. Download and install 8086 simulator from [here](https://emu8086-microprocessor-emulator.en.softonic.com/?ex=RAMP-2046.0).
2. Clone this repository to your local machine.
3. Open 8086 simulator and load the `Zadaca1 8086 code.asm` file.
4. Assemble the code by pressing the Assemble button.
5. Run the program by pressing the Run button or by pressing F10.
