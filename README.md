# asm8051 實作系列

## project001
int0_int1_timer: 實作兩個中斷點，以及計時。
程式會先在P1系列，從0~7依序閃爍，若有中斷按鈕(P3.2andP3.3)，則前四個燈同時閃爍2or3次，後四個燈同時閃爍2or3次，之後回到主程式。

## project002
timeradvance: 高階控制delay之數。
正轉，f = 4。
int0:正轉，f = 8。DELAY 2S。
int1:反轉，f = 2。DELAY 1S。

## project003
dispfunc: 高階控disp之術。
因為我們要顯示count2，在這裡他像是C的全域變數，所以可以直接寫disp，然後顯示當前的count2。



## 兩個中斷點的程式架構
```asm=
	ORG 0H; 一開張會從這開始
	AJMP MAIN
	ORG 03H; 按了中斷點0(P3.2接地)會從這開始
	JMP INIT0
	ORG 13H; 按了中斷點1(P3.3接地)會從這開始
	JMP INIT1
;----------------------------
MAIN:
    MOV	IE,#10000101B; 三個1分別為 啟動中斷功能 啟動中斷點1 啟動中斷點0
	...
LOOP
	...
	AJMP LOOP

;----------------------------
INIT0:
	
	...
	
	RETI

;-----------------------------
INIT1:
	
	...
	
	RETI

;----------------------------
```
Note: 標籤(MAIN or INIT0)，有無分號似乎沒關係。

### EXAMPLE
```asm=
	ORG 0H
	AJMP MAIN
	ORG 03H
	JMP INIT0
	ORG 13H
	JMP INIT1
;-----------------------------------
MAIN:
	MOV IE,#10000101B
	MOV A,#11111110B
LOOP
	MOV P1,A
	ACALL DELAY1
	RL A
	AJMP LOOP

;-----------------------------------
INIT0:
	PUSH A
	PUSH COUNT3
	MOV R0,#11110000B
	MOV COUNT2,#02
C0
	MOV COUNT1,#02
A0
	MOV P1,R0
	CALL DELAY1
	MOV P1,#FFH
	CALL DELAY1
	DJNZ COUNT1,A0
	CALL DELAY1

	MOV A,R0
	SWAP A
	MOV R0,A
	DJNZ COUNT2,C0
	POP COUNT3
	POP A
	RETI

;-----------------------------------
INIT1:
	PUSH A
	PUSH COUNT3
	MOV R0,#11110000B
	MOV COUNT2,#02
C1
	MOV COUNT1,#03
A1
	MOV P1,R0
	CALL DELAY1
	MOV P1,#FFH
	CALL DELAY1
	DJNZ COUNT1,A1
	CALL DELAY1

	MOV A,R0
	SWAP A
	MOV R0,A
	DJNZ COUNT2,C1
	POP COUNT3
	POP A
	RETI


;-----------------------------------
DELAY1:
	MOV COUNT3,#5
	MOV TMOD,#00000001B
TIMER
	MOV TH0,#3CH
	MOV TL0,#B0H
	SETB TR0
WAIT 
	JB TF0,OK
	AJMP WAIT
OK
	CLR TF0
	DJNZ COUNT3,TIMER
	RET

;-----------------------------------
```

## 如何使用7段顯示器
> 高階控disp之術。 因為我們要顯示count2，在這裡他像是C語言的全域變數，所以可以直接寫disp()，而不用disp(count2)，然後顯示當前的count2。
```asm=
    COUNT2 REG 09
	ORG 0H
	AJMP MAIN
;-----------------------------
MAIN:  
    MOV	IE,#10000101B
LOOP
    MOV COUNT2,#8
    ACALL DISP
    ACALL DELAY1
    MOV COUNT2,#7
    ACALL DISP
    ACALL DELAY1
    AJMP LOOP

;-----------------------------
DISP:
	MOV DPTR,#DATA
	MOV A,COUNT2
	MOVC A,@A+DPTR
	MOV P2,A; 把on/off的狀態寫入
	RET

;-----------------------------
DELAY1:
	MOV COUNT3,#32
	MOV TMOD,#00000001B
TIMER1
	MOV TH0,#85H
	MOV TL0,#EEH
	SETB TR0
WAIT1
	JB TF0,OK1
	AJMP WAIT1
OK1
	CLR TF0
	DJNZ COUNT3,TIMER1
	RET

;-----------------------------------
DATA
DB 11000000B
DB 11111001B
DB 10100100B
DB 10110000B
DB 10011001B
DB 10010010B
DB 10000011B
DB 11111000B
DB 10000000B
DB 10010000B

```

