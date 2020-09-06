@[TOC](大整数/高精度板子（已更新除法）)
# 前言
有些题的数据范围很大，以至于开了long long还是会爆......
为此我写了大整数，大致思想是开long long数组存储每九位数字（多了在处理乘法的时候会爆），然后用一个布尔值来判断正负。
目前只支持赋值，加法，减法，乘法，除法，比较。
~~感觉除法太难，暂时没想出来怎么写，可能以后用到了会补上吧（orz）。~~ 

**除法已于9月6日0点补上。**~~（睡觉前灵光一闪，想到二分法可以解决除法，爬起来写了）~~ 

# 使用方法
声明：
```c
bignum a,b,c;
```
初始化：
```c
a.clear();
b.clear();
```
赋值：
```c
a = 123; //赋值int类型
a = "123"; //赋值字符串类型(char *)
a = b; //赋值bignum类型
```
运算（支持 + - *）：
```c
c = a + b; //bignum + bignum
c = a + 1; //bignum + int
c += a; //bignum += bignum
c += 1; //bignum += int
c = -a; //bignum的负数
```
比较：
```c
if (a < c)
{
	//...
}
if (a >= c)
{
	//...
}
```
# 实现思路
闲来无事，讲一讲写大整数时的思路吧。
## 大整数存储
就是用一个数组存一个较大的数字，为了效率，我开的是longlong数组，每一位可以存1e9的数字，存1e9可以方便借位，而且1e9乘1e9也不会爆long long。
每一位数字乘以1e9^下标的和即为大整数本身，用a[i]表示大数a的每一位数字，则：
$$
a = \sum\limits_{i=1}^n a[i] × 1e(9i)
$$
## 乘法实现
若用a[i]表示大数a的每一位数字，b[i]表示大数b的每一位数字，则易得：
$$
a*b=\sum\limits_{i}\sum\limits_{j} a[i] × b[i]×1e(9(i+j))
$$
也就是先把$$a = 123,456,789$$看成$$a = 123 × 1e6 + 456 × 1e3 + 789$$
然后一个O(n^2)级别的遍历相乘得到结果。
（本来想用分治的，结果复杂度也是n^2，算了算了）
## 除法实现
若求$$ c = \frac{a}{b}$$，则可以用二分法找到 **c** 使得$$a=bc$$，易知除法的时间复杂度是O(n^2logm)。

其实还可以有另一种写法，就是将ab对齐后再进行除法的计算，回头有空了写写看。
# 实现代码
```c
#include<iostream>
#include<algorithm>
#include<cstring>
#include<cstdio>
#include<cmath>
using namespace std;
const int maxn2 = 100;
const long long max_digit = 1e9;
struct big_num
{
	long long digits[maxn2];
	bool minus = 0;
	void clear(void)
	{
		this->minus = 0;
		memset(this->digits, 0, sizeof(this->digits));
	}
	big_num operator=(const big_num b)
	{
		for (int i = 0; i < maxn2; i++)
			this->digits[i] = b.digits[i];
		this->minus = b.minus;
		return *this;
	}
	big_num operator=(const int b)
	{
		this->clear();
		this->minus = b < 0;
		if (b < 0)
			this->digits[0] = -b;
		else
			this->digits[0] = b;
		return *this;
	}
	big_num operator=(const char* b)
	{
		int len = strlen(b), st = 0;
		this->clear();
		if (b[0] == '-')
		{
			st = 1;
			this->minus = 1;
		}
		for (int i = st; i < len; i++)
		{
			this->digits[(len - i - 1 + st) / 9] *= 10;
			this->digits[(len - i - 1 + st) / 9] += b[i] - '0';
		}
		return *this;
	}
	bool operator<(const big_num b)
	{
		if (this->minus != b.minus) return this->minus > b.minus;
		for (int i = maxn2 - 1; i >= 0; i--)
			if (this->digits[i] != b.digits[i])
				return (this->digits[i] < b.digits[i]) ^ minus;
		return false;
	}
	bool operator>(const big_num b)
	{
		if (this->minus != b.minus) return this->minus < b.minus;
		for (int i = maxn2 - 1; i >= 0; i--)
			if (this->digits[i] != b.digits[i])
				return (this->digits[i] > b.digits[i]) ^ this->minus;
		return false;
	}
	bool operator==(const big_num b)
	{
		if (this->minus != b.minus) return false;
		for (int i = maxn2 - 1; i >= 0; i--)
			if (this->digits[i] != b.digits[i])
				return false;
		return true;
	}
	bool operator<=(const big_num b)
	{
		return *this < b || *this == b;
	}
	bool operator>=(const big_num b)
	{
		return *this > b || *this == b;
	}
	const big_num operator-() const
	{
		big_num temp = *this;
		temp.minus ^= 1;
		return temp;
	}
	big_num operator+(const big_num b)const
	{
		big_num temp = *this;
		if (temp.minus == b.minus)
		{
			for (int i = 0; i < maxn2; i++)
			{
				temp.digits[i] += b.digits[i];
			}
			for (int i = 1; i < maxn2; i++)
			{
				temp.digits[i] += temp.digits[i - 1] / max_digit;
				temp.digits[i - 1] %= max_digit;
			}
			return temp;
		}
		else
		{
			return temp - (-b);
		}
	}
	big_num operator+(const int b)const
	{
		big_num temp = *this;
		big_num b2;
		b2 = b;
		return temp + b2;
	}
	big_num operator-(const big_num b) const
	{
		big_num temp = *this;
		if (temp.minus == b.minus)
		{
			if (temp >= b && temp.minus == 0 || temp <= b && temp.minus == 1)
			{
				for (int i = 0; i < maxn2; i++)
				{
					temp.digits[i] -= b.digits[i];
				}
				for (int i = 1; i < maxn2; i++)
				{
					if (temp.digits[i - 1] < 0)
					{
						temp.digits[i - 1] += max_digit;
						temp.digits[i]--;
					}
				}
				return temp;
			}
			else
			{
				return -(b - temp);
			}
		}
		else
		{
			return temp + (-b);
		}
	}
	big_num operator-(const int b) const
	{
		big_num temp = *this;
		big_num b2;
		b2 = abs(b);
		b2.minus = b < 0;
		return temp - b2;
	}
	big_num operator*(const big_num b)const
	{
		big_num temp;
		temp.clear();
		temp.minus = this->minus ^ b.minus;
		for (int i = maxn2 - 1; i >= 0; i--)
		{
			for (int i2 = maxn2 - i - 1; i2 >= 0; i2--)
			{
				temp.digits[i + i2] += this->digits[i] * b.digits[i2];
				if (temp.digits[i + i2] >= max_digit && i + i2 + 1 < maxn2)
				{
					temp.digits[i + i2 + 1] += temp.digits[i + i2] / max_digit;
				}
				temp.digits[i + i2] %= max_digit;
			}
		}
		for (int i = 1; i < maxn2; i++)
		{
			temp.digits[i] += temp.digits[i - 1] / max_digit;
			temp.digits[i - 1] %= max_digit;
		}
		return temp;
	}
	big_num operator*(const int b)const
	{
		big_num temp = *this;
		temp.minus ^= (b < 0);
		for (int i = 0; i < maxn2; i++)
		{
			temp.digits[i] *= b;
		}
		for (int i = 1; i < maxn2; i++)
		{
			temp.digits[i] += temp.digits[i - 1] / max_digit;
			temp.digits[i - 1] %= max_digit;
		}
		return temp;
	}
	big_num operator/(const big_num b) const
	{
		int d = 0;
		big_num temp = *this;
		big_num l, r, mid;
		l.clear();
		r.clear();
		mid.clear();
		for (int i = 0; i < maxn2; i++)
		{
			if (b.digits[i] != 0)
				d = max(d, i);
		}
		l = 0;
		r.digits[maxn2 - 1 - d] = 1;
		while (l < r)
		{
			mid = (l + r + 1) / 2;
			if (mid * b > temp)
				r = mid - 1;
			else
				l = mid;
		}
		return l;
	}
	big_num operator/(const int b)const
	{
		big_num temp = *this;
		for (int i = maxn2 - 1; i >= 1; i--)
		{
			temp.digits[i - 1] += (temp.digits[i] % b) * max_digit;
			temp.digits[i] /= b;
		}
		temp.digits[0] /= b;
		return temp;
	}
	big_num operator+=(const big_num b)
	{
		*this = *this + b;
		return *this;
	}
	big_num operator-=(const big_num b)
	{
		*this = *this - b;
		return *this;
	}
	big_num operator*=(const big_num b)
	{
		*this = *this * b;
		return *this;
	}
	big_num operator/=(const big_num b)
	{
		*this = *this / b;
		return *this;
	}
	big_num operator+=(const int b)
	{
		*this = *this + b;
		return *this;
	}
	big_num operator-=(const int b)
	{
		*this = *this - b;
		return *this;
	}
	big_num operator*=(const int b)
	{
		*this = *this * b;
		return *this;
	}
	big_num operator/=(const int b)
	{
		*this = *this / b;
		return *this;
	}
	void show(void)
	{
		bool is_begun = 0;
		for (int i = maxn2 - 1; i >= 0; i--)
		{

			if (is_begun)
				printf("%0.9lld", this->digits[i]);
			if (this->digits[i] != 0 && !is_begun)
			{
				if (this->minus == 1)
					putchar('-');
				is_begun = 1;
				printf("%lld", this->digits[i]);
			}
		}
		if (is_begun == 0) putchar('0');
		putchar('\n');
	}
};
```
# 部分习题
[洛谷1005 - 矩阵取数游戏](https://www.luogu.com.cn/problem/P1005)
[洛谷1797 - 克鲁斯的加减法](https://www.luogu.com.cn/problem/P1797)

