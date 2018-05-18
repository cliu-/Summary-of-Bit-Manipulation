# Summary-of-Bit-Manipulation

## 位运算基础

基本的位操作符有与、或、异或、取反、左移、右移这6种，它们的运算规则如下所示：

| 符号 | 描述 | 运算规则                                                     |
| ---- | ---- | ------------------------------------------------------------ |
| `&`  | 与   | 两个位都为1时，结果才为1                                     |
| `丨` | 或   | 两个位都为0时，结果才为0                                     |
| `^`  | 异或 | 两个位相同为0，相异为1                                       |
| `~`  | 取反 | 0变1，1变0                                                   |
| `<<` | 左移 | 各二进位全部左移若干位，高位丢弃，低位补0                    |
| `>>` | 右移 | 各二进位全部右移若干位，对无符号数，高位补0，有符号数，各编译器处理方法不一样，有的补符号位（算术右移），有的补0（逻辑右移） |

基本技巧：

- `^`异或操作符可以删除两个二进制中相同的位，留下不同的位。Use `^` to remove even exactly same numbers and save the odd, or save the distinct bits and remove the same.

需要注意以下几点：

- 这6种操作符，只有~取反是单目操作符，其它5种都是双目操作符；

- 位操作只能用于整形数据，对float和double类型进行位操作会被编译器报错；

- `^`异或操作符

- 移位操作都是采取算术移位操作，算术移位是相对于逻辑移位，它们在左移操作中都一样，低位补0即可，但在**右移中逻辑移位的高位补0而算术移位的高位是补符号位**。如下面代码会输出-4和3；

  ```
  a, b = -15, 15
  puts a >> 2
  puts b >> 2
  ```

因为15=0000 1111(二进制)，右移二位，最高位由符号位填充将得到0000 0011即3。-15 = 1111 0001(二进制)，右移二位，最高位由符号位填充将得到1111 1100即-4。

- 位操作符的运算优先级比较低，因为尽量使用括号来确保运算顺序，否则很可能会得到莫明其妙的结果。比如要得到像1，3，5，9这些2^i+1的数字。写成a = 1 << i + 1是不对的，程序会先执行i + 1，再执行左移操作。应该写成a = (1 << i) + 1;
- 另外位操作还有一些复合操作符，如&=、|=、 ^=、<<=、>>=。

##常用位操作小技巧

下面对位操作的一些常见应用作个总结，有判断奇偶、交换两数、变换符号及求绝对值。这些小技巧应用易记，应当熟练掌握。

### 判断奇偶

只要根据最未位是0还是1来决定，为0就是偶数，为1就是奇数。因此可以用if ((a & 1) == 0)代替if (a % 2 == 0)来判断a是不是偶数。

下面程序输出0到100之间的所有奇数。

```python
def isOdd(n):
    return n&1
```

### 交换两数

一般交换函数的写法是：

```python
def swap(a, b):
    if a != b:
        c = a
        a = b
        b = c
```

利用python的语法糖，可以简单的写成：

```python
def swap(a, b):
    a, b = b, a
```

可以用位操作来实现交换两数而不用第三方变量：

```python
def swap(a, b):
    if a != b:
        a ^= b
        b ^= a
        a ^= b
```

可以这样理解：

1. a ^= b 即a = (a ^ b);
2. b ^= a 即b = b ^ (a ^ b)，由于^运算满足交换律，b ^ (a ^ b)=b ^ b ^ a。由于一个数和自己异或的结果为0并且任何数与0异或都会不变的，所以此时b被赋上了a的值;
3. a ^= b 就是a = a ^ b，由于前面二步可知a = (a ^ b)，b = a，所以a = a ^ b即a = (a ^ b) ^ a。故a会被赋上b的值。

再来个实例说明下以加深印象。a = 13, b = 6:
a的二进制为 13 = 8 + 4 + 1 = 1101(二进制)
b的二进制为 6 = 4 + 2 = 110(二进制)

1. a ^= b a = 1101 ^ 110 = 1011;
2. b ^= a b = 110 ^ 1011 = 1101; 即b == 13
3. a ^= b a = 1011 ^ 1101 = 110; 即a == 6

### 变换符号

变换符号就是正数变成负数，负数变成正数。
如对于-11和11，可以通过下面的变换方法将-11变成11:

```
1111 0101(二进制)
取反-> 0000 1010(二进制) 
加1-> 0000 1011(二进制)

```

同样可以这样的将11变成-11

```
0000 1011(二进制)
取反-> 1111 0100(二进制)
加1-> 1111 0101(二进制)

```

因此变换符号只需要取反后加1即可。完整代码如下：

```python
def sign_reversal(n):
    return ~n + 1
```

### 求绝对值

位操作也可以用来求绝对值，对于负数可以通过对其取反后加1来得到正数。对-6可以这样：

```
1111 1010(二进制)
取反->0000 0101(二进制)
加1-> 0000 0110(二进制)
```

来得到6。

因此先移位来取符号位，i = a >> 31;要注意如果a为正数，i等于0，为负数，i等于-1。然后对i进行判断——如果i等于0，直接返回；否之，返回~a + 1。完整代码如下：

```python
def my_abs(n):
    i = n >> 31
    return n if i == 0 else (~n + 1)
```

现在再分析下。对于任何数，与0异或都会保持不变，与-1即0xFFFFFFFF异或就相当于取反。因此，n与i异或后再减i（因为i为0或-1，所以减i即是要么加0要么加1）也可以得到绝对值。所以可以对上面代码优化下：

```python
def my_abs(n):
  i = a >> 31
  return ((a ^ i) - i)
```

注意这种方法没用任何判断表达式.

### 计算二进制数中1的数量

给定一个十进制整数，计算这个整数对应的二进制数中1的数量。

Write a function that takes an unsigned integer and returns the number of ’1' bits it has (also known as the Hamming weight).

```python
def count_one(n):
    count = 0
    while n:
        n = n&(n-1)
        count+=1
    return count
```

### Power of Two

这道题让我们判断一个数是否为2的次方数，而且要求时间和空间复杂度都为常数。

Given an integer, write a function to determine if it is a power of two.

比较下x - 1和x的关系试试？以x=4为例。

```
0100 ==> 4
0011 ==> 3
```

两个数进行按位与就为0了！如果不是2的整数幂则无上述关系，反证法可证之。

```python
def is_power_of_two(n):
   return n > 0 and not (n & n-1)
```

Get the value of a power of two with integer `exponent` as the exponent.

```python
def power_of_two(exponent):
    return 1<<exponent
```

### Power of Four

Given an integer (signed 32 bits), write a function to check whether it is a power of 4.

```python
def is_power_of_four(n):
    return n > 0 and not n&&(n-1) and n==(n&0x55555555555)
```

### 两整数和 Sum of Two Integers

Calculate the sum of two integers *a* and *b*, but you are **not allowed** to use the operator `+` and `-`.

```python
def getSum(a, b):
    return a if b==0 else getSum(a^b, (a&b)<<1)
```

### Missing Number

Given an array containing *n* distinct numbers taken from `0, 1, 2, ..., n`, find the one that is missing from the array.

For example,
Given `nums = [0, 1, 3]` return `2`.

```python
def missingNum(nums):
    ret = 0
    for i, n in enumerate(nums):
        ret ^= i
        ret ^= n
    return ret^len(nums)
```

Of cause, for this particular problem, solve it by math is a better solution.

```python
def missingNum(nums):
    return len(nums)*((len(nums)+1)//2) - sum(nums)
```

### Single Number
#### Variation I
一个数组中除了一个数字出现过一次外，其余的数字都出现了两次，找出那个只出现一次的数字。

Given an array of integers, every element appears *twice* except for one. Find that single one.

注意点：

- 算法时间杂度要求为O(n)
- 空间复杂度为O(1)

例子:

输入: nums = [1, 2, 3, 4, 3, 2, 1]
输出: 4

非常常见的一道算法题，将所有数字进行异或操作即可。对于异或操作明确以下三点：

- 一个整数与自己异或的结果是0
- 一个整数与0异或的结果是自己
- 异或操作满足交换律，即a^b=b^a

所以对所有数字进行异或操作后剩下的就是那个只出现一次的数字。

```python
import functools
def singleNum(nums):
    return functools.reduce(lambda x, y: x^y, nums)
```

#### Variation II
Given an array of numbers `nums`, in which exactly two elements appear only once and all the other elements appear exactly twice. Find the two elements that appear only once.
**Example:**
```
Input:  [1,2,1,3,2,5]
Output: [3,5]
```
**Solution:**
```python
from functools import reduce
def singleNumber(nums):
    diff=reduce(lambda x, y: x^y, nums) # first pass
    res,diff=[0]*2,diff&-diff
    for n in nums: # second pass
        if n&diff==0:
            res[0]^=n
        else:
            res[1]^=n
    return res
```
**Complexity:**
Time: O (n)
Space: O (1)

**Explaination:**
we need to use XOR to solve this problem. But this time, we need to do it in two passes:
In the first pass, we XOR all elements in the array, and get the XOR of the two numbers we need to find. Note that since the two numbers are distinct, so there must be a set bit (that is, the bit with value '1') in the XOR result. Find
out an arbitrary set bit (for example, the rightmost set bit).
In the second pass, we divide all numbers into two groups, one with the aforementioned bit set, another with the aforementinoed bit unset. Two different numbers we need to find must fall into thte two distrinct groups. XOR numbers in each group, we can find a number in either group.

### Gray Code

The **reflected binary code** (**RBC**), also known just as **reflected binary** (**RB**) or **Gray code** after [Frank Gray](https://en.wikipedia.org/wiki/Frank_Gray_(researcher)), is an ordering of the [binary numeral system](https://en.wikipedia.org/wiki/Binary_numeral_system) such that two successive values differ in only one [bit](https://en.wikipedia.org/wiki/Bit) (binary digit).

Get the gray code for a specified decimal unsigned number:

```python
def grayCode(num):
    return num ^ (num >> 1)
```


