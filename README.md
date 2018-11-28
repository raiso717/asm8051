# asm8051 實作系列

## project001
int0_int1_timer: 實作兩個中斷點，以及計時。
程式會先在P1系列，從0~7依序閃爍，若有中斷按鈕(P3.2andP3.3)，則前四個燈同時閃爍2or3次，後四個燈同時閃爍2or3次，之後回到主程式。

## project002
timeradvance: 高階控制delay之數。
正轉，f = 4。
int0:正轉，f = 8。DELAY 2S。
int1:反轉，f = 2。DELAY 1S。
