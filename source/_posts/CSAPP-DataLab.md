---
title: CSAPP-DataLab
date: 2024-06-25 20:25:24
tags: CSAPP-Lab
---







**整篇Lab的核心思想就是以小见大，通过对较低位数数字的思考，从而推广到32位，64位的位数。以及位数的运算一些题目和逻辑运算有着异曲同工之处。可以很好的进行类比。**

### **bitXor**

思考：异或的逻辑运算式：x ^ y = (x & (~y)) | ((~x) & y)。通过逻辑运算式子的变式。主要是**双重否定律和德摩尔根定律**的运用。

```c
/*
 * bitXor - x^y using only ~ and &
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
    // (x & (~y)) | ((~x) & y) = ~~((x & (^y)) | ((~x) & y)) 
    return ~(~((~x) & y) & ~((~y) & x)); 
    
}
```



### tmin

这个没有什么技术含量，只要记住tmin和tmax的含义就完全行。

```c
/*
 * tmin - return minimum two's complement integer
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void)a {
	// tmin = 10000000
    return (1 << 31);
}
```

### isTmax

思考：Tmax = 0111(四位中)。Tmax + 1 = 1000 = Tmin。 ~Tmin =  Tmax = 0111.

但还需要考虑的是，一个特殊的情况，1111 + 1 = 0000 ~0000 = 1111，所以-1也是满足这个特殊的条件的，需要在最后把他给排除掉。

这里的不同是，1111 + 1  = 0 而`Tmax`+ 1 != 0，可以根据这个差别来进行对二者的区分。

```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */

int isTmax(int x) {
    int y = ~(x + 1);
    return !(~(x + 1) ^ x) & !!((x + 1) ^ 0);// 注意这里要对 (x + 1) ^ 0x0,进行规格化（！！）转变为判断符号。
}
```

### allOddBits

思路：这个题刚开始理解错了意思，题意是这个数字所有的奇数位都要是1才行，所以不能存在一个奇数位不是1，因此：

对于四位中1010这样的数字则是满足条件的，所以在32位中`0xaaaaaaaa`才是满足条件的，所以整个题的思路就是判断这个数和0xaaaaaaaa的奇数位的相等性，偶数位的状况是需要被我们忽略并舍弃的。

**所以，需要一个掩码来对使得x忽略掉所有的偶数位。就是使偶数位全部为0，从而不影响之后的判断。**

在这里，更重要的则是，获取0xaaaaaaaa这个数字的方法和蕴含的思想。

```c
/*
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
    // get the 0xaaaaaaaa
    // 这个掩码需要做到和x运算之后使偶数位全都为0
    int a = 0xa; // 0x1010
    a = (a << 4) | a; // 0x 10101010
    a = (a << 8) | a; // 0x 1010101010101010
    a = (a << 16) | a; // 0xaaaaaaaa
	int maskCode = a;
    // 消除掉偶数位的影响之后，只需要判断是否等于掩码值
    return !((maskCode & a) ^ maskCode);
	
} 
```

### isAsciiDigit

思路：首先我们需要找到AsciiDigit的二进制位数表示的通用性和规律。

'0' = 0x30 = 0011 0000  '9' = 0011 1001

1. 该数字的前24位都需要是0。

2. 数字的倒数最后一字节的前半部分  == 0011(3)
3. 最后四位 <= 1001

```c
/*
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
    // int cond1 = !((x >> 8) ^ 0);
    int cond2 = !((x >> 4) ^ 0x3);
    int cond3 = !((((~(x & 0xf) + 1) + 9) >> 31) ^ 0x0); // get the last 4 bits and calculate the 9- last-4-bits
    // get the signal bits judge if it is positive number
    return cond2 & cond3;
}
```

### conditional

条件式：这道题的突破口在于掩码部分，同时也非常的有技巧性。

1.  先对x进行规格化
2.  规格化之后得到的是 1 or 0, 利用 `0xf & x = x`的性质，进行位移运算得到`0xfffffff` or 0.再进行掩码操作 

```c
/*
 * conditional - same as x ? y : z
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
// find a universal way to behave the y & z
int conditional(int x, int y, int z) {
    int a = !!x; // 首先先进行规格化
   	int mask = a << 31 >> 31; // 1111 1111 or 0000 0000
    return ((~mask) & z) | (mask & y);
  
}
```

### isLessOrEqual

简单明了，返回 x - y的符号位.

```c
/*
 * isLessOrEqual - if x <= y  then return 1, else return 0
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
	return !((((~x) + 1) + y) >> 31) & 0x01) | !((((~x) + 1) + y) >> 31) ^ 0x0);
}


```

### logicalNeg

```	c
/*
 * logicalNeg - implement the ! operator, using all of
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4
 */
int logicalNeg(int x){
    int negX = ~x + 1;
    int signX = (negX | x) >> 31;
    return signX + 1;
}
```



### howManyBits

1. X == 0 or x != 0

返回能表示x的最小的位数值，要注意的是：1111 = -1 111 = -1 11 = -1 1 = -1。

这说明了对于**表示一个负数**来说，其中的除去符号位以外的最高有效位之前的所有连续的1都是多余的，这些1都是为了满足能够使用规定的位数（如32位），所以，一个x的最小的二进制表示位数：sign + 数字。如`1110 1101` = 10 **1**101。而在这里只需要计算x的位数，所以获取到一个拥有和x的至少的有效位数一样的**数字**（作为中介值），进行计算。

对于一个**正数**来说，只用考虑符号位之后第一个为1的位。

```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
// eg: x(-) = 1110 1100 in fact x = 10 1100(the least bits to behave)
// ~x = 0001 0011 
// signX = x >> 31  = 1111 1111
// signX & ~x = 000 10011 num(10011) = num(10 1100) - signX
// x(+) = 0111 0110 in fact x = 1 0110(the least bitnum)
// signX = x >> 31 = 0
// ~signX = 1111 1111 
// ~signX & x = x 
int howManyBits(int x) {
    int isZero = !x; // x == 0 return 1
    // 0000 0000 | 0000 0000
    // 1. get rid of the redundant 1
    int flag = x >> 31;
    int mask = !!x  << 31 >> 31;
    
    int a = (flag & (~x)) | ((~flag) & x); // get the right word size FOR NEGATIVE 
    int bit_16 , bit_8, bit_4, bit_2, bit_1, bit_0;
    // 3. deconomy
    // every half for these bits number to detect if there is number '1' on every half 
    bit_16 = !((!!(a >> 16)) ^ 0x1) << 4; // if is yes turn bit_16 to 16 means which means num += 16
    a >>= bit_16; // bit_16 = 16 | 0
    
    bit_8 = !((!!(a >> 8)) ^ 0x1) << 3; // if is yes turn bit_8 to 8 means which means num += 8
    a >>= bit_8;
    
    bit_4 = !((!!(a >> 4)) ^ 0x1) << 2; // if is yes turn bit_4 to 4 means which means num += 4
    
    a >>= bit_4;
    bit_2 = !((!!(a >> 2)) ^ 0x1) << 1; // if is yes turn bit_2 to 2 means which means num += 2
    a >>= bit_2;
    
    bit_1 = !(!!(a >> 1)) ^ 0x01);
    a >>= bit_1;
    bit_0 = a; // the last bit 1 | 0
    int res = bit_16 + bit_8 + bit_4 + bit_2 + bit_1 + bit_0;
	
    res = res + 1; // the signal bit
    return (mask & res) | isZero; //there is the +0 -0 ?
    
    
    
     
}
```



