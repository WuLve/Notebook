# RSA非对称加密

## 1.RSA生成密钥

```c
//用于计算RSA密匙
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>
#include <stdbool.h>

bool is_prime_number(int);			 //判断输入的P和Q是不是素数
bool gcd(int, int);					 //判断两个数是不是互素。
void auto_build_prime_number(int *); //生成素数值
void auto_build_e(int *, int *);	 //生成E的值
int extend(int, int);				 //求E关于模(p-1)(q-1)的逆元D：即私钥
int gcdEx(int, int, int *, int *);	 //拓展欧几里得算法

int main()
{
	//设置UTF-8编码
	// system("chcp 65001");
	// system("cls");

	while (1)
	{
		int Q, P, E, D, N, ora, tep = 0;
		// 1.输入素数p,q
		printf("请输入P(输入0自动取值):", P);
		while (tep == 0)
		{
			scanf("%d", &P);
			auto_build_prime_number(&P);
			tep = is_prime_number(P);
			if (tep == 0)
				printf("P不是素数，请重新输入P！\n");
		}
		tep = 0;
		printf("请输入Q(输入0自动取值):", Q);
		while (tep == 0)
		{
			scanf("%d", &Q);
			auto_build_prime_number(&Q);
			tep = is_prime_number(Q);
			if (tep == 0)
				printf("Q不是素数，请重新输入Q！\n");
		}
		printf("------------------------------------\n");
		// 2.计算p,q的乘积,n=p*q
		N = P * Q;
		// 3.计算n的欧拉函数,φ(n)=(p-1)*(q-1)
		ora = (P - 1) * (Q - 1);
		printf("ora=(%d-1)*(%d-1)=%d\n", P, Q, ora);
		// 4.选⼀个与φ(n)互质的整数e,1<e<φ(n)
		tep = 0;
		printf("要求:使得(E,ora)=1\n请输入E(输入0自动取值):");
		while (tep == 0)
		{
			scanf("%d", &E);
			auto_build_e(&E, &ora);
			tep = gcd(E, ora);
			if (tep == 0)
				printf("请重新输入一个指数e，使得(e,ora)=1，自动生成输入0");
		}
		// 5.计算出e对于φ(n)的模反元素d de mod φ(n)=1
		D = extend(E, ora);
		// 6.输出密钥
		printf("------------------------------------\n");
		printf("公匙 E N : %d-%d \n", E, N);
		printf("私匙 D N : %d-%d \n", D, N);

		system("pause");
	}
	return 0;
}

//判断输入的p和q是不是素数
bool is_prime_number(int s)
{
	for (int i = 2; i < s; i++)
	{
		if (s % i == 0)
			return false;
	}
	return true;
}

//判断两个数是不是互质。
bool gcd(int p, int q)
{
	int temp1, temp2; // q=temp2*p+temp1 ;
	if (q < p)
	{
		temp1 = p;
		p = q;
		q = temp1;
	}
	temp1 = q % p, temp2 = q / p;
	while (temp1 != 0)
	{

		q = p;
		p = temp1;
		temp1 = q % p;
		temp2 = q / p;
	}
	if (temp1 == 0 && temp2 == q)
	{
		printf("符合条件！\n");
		return true;
	}
	else
	{
		printf("不符合条件！请重新输入：\n");
		return false;
	}
}

//生成素数的值
void auto_build_prime_number(int *num)
{
	if (*num == 0)
	{
		srand(time(0));			  //产生随机数种子
		*num = rand() % 100 + 60; // rand()产生一个随机数
		while (is_prime_number(*num) == 0)
		{							  // num不为质数重新选值
			*num = rand() % 100 + 60; // rand()产生一个随机数
		}
	}
}

//生成E的值
void auto_build_e(int *E, int *ora)
{
	if (*E == 0)
	{
		int s1, t1;
		srand(time(0));			//产生随机数种子
		*E = rand() % *ora + 1; // rand()%ora 产生一个0~(ora-1)的随机数，+1保证a最小只能是1
		while (gcdEx(*E, *ora, &s1, &t1) != 1)
		{
			*E = rand() % *ora + 1;
		};
	}
}

//求e关于模(p-1)(q-1)的逆元d：即私钥
int extend(int E, int ora)
{
	int D;
	for (D = 0; D < ora; D++)
	{
		if (E * D % ora == 1)
			return D;
	}
}

//扩展欧几里得算法
int gcdEx(int a, int b, int *x, int *y)
{
	if (b == 0)
	{
		*x = 1, *y = 0;
		return a;
	}
	else
	{
		int r = gcdEx(b, a % b, x, y);
		int t = *x;
		*x = *y;
		*y = t - a / b * *y;
		return r;
	}
}
```

## 2.RSA加密

```c
//用于计算RSA加密
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define LENGTH 1024

void encrypt(int, int, int *, int *);

int main(int argc, char *argv[])
{
	//设置UTF-8编码
	// system("chcp 65001");
	// system("cls");

	//字符明文->数字明文->数字密文
	char char_plaintext[LENGTH] = {0}; //字符明文
	int int_plaintext[LENGTH] = {0};   //数字明文
	int int_ciphertext[LENGTH] = {0};  //数字密文
	unsigned char tem[LENGTH] = {0};   //临时数组
	int E, N;						   // RSA公匙
	// E = 2825, N = 7811;			   // RSA公匙
	printf("--------------RSA  加密-------------\n");
	printf("请输入RSA公匙:(E.g:2825-7811)\n");
	scanf("%d-%d",&E,&N);

	//-------------------------------------------------------------------
	// 1. 输入字符明文
	printf("------------------------------------\n");
	printf("请输入符号明文：");
	scanf("%s",char_plaintext);
	// gets(char_plaintext);

	//-------------------------------------------------------------------
	// 2. 字符明文 -> 数字明文,采用默认的编码,Linux为UTF-8,Windows为GBK
	memcpy(tem, char_plaintext, sizeof(unsigned char) * LENGTH); //将char转为unsigned char
	for (int i = 0; i < LENGTH; i++)
	{
		int_plaintext[i] = (int)tem[i];
	}

	//-------------------------------------------------------------------
	// 3. 加密函数 , 数字明文 -> 数字密文.
	encrypt(E, N, int_plaintext, int_ciphertext);

	//-------------------------------------------------------------------
	// 4. 打印数字密文
	printf("\n加密密文为: \n");
	for (int i = 0; i < LENGTH; i++)
		printf("%d ", int_ciphertext[i]);
	printf("\n");

	system("pause");
	return 0;
}

//加密计算函数
void encrypt(int E, int N, int *int_plaintext, int *int_ciphertext)
{
	//开始加密
	int num = 1; // ciphertext为加密后的数字密文
	for (int i = 0; i < LENGTH; i++)
	{

		for (int j = 0; j < E; j++)
		{
			num = num * int_plaintext[i] % N;
		}
		int_ciphertext[i] = num;
		num = 1;
	}
}
```

## 3.RSA解密

```c
//用于RSA解密
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define LENGTH 1024

void decrypt(int, int, int *, int *);

int main()
{
	//设置UTF-8编码
	// system("chcp 65001");
	// system("cls");

	//数字密文->数字明文->字符明文
	char char_plaintext[LENGTH] = {0}; //字符明文
	int int_plaintext[LENGTH] = {0};   //数字明文
	int int_ciphertext[LENGTH] = {0};  //数字密文
	unsigned char tem[LENGTH] = {0};   //临时数组
	int D, N;						   // RSA私匙
	// D = 4409, N = 7811;			   // RSA私匙
	printf("--------------RSA  解密-------------\n");
	printf("请输入RSA私匙:(E.g:4409-7811)\n");
	scanf("%d-%d", &D, &N);

	//-------------------------------------------------------------------
	// 1. 输入数字密文
	printf("------------------------------------\n");
	printf("请输入密文:\n");
	int num;
	int num2 = 0;
	while (1)
	{
		scanf("%d", &num);
		char c = getchar();
		int_ciphertext[num2++] = num;
		if (c == '\n')
		{
			break;
		}
	}

	//-------------------------------------------------------------------
	// 2. 解密函数 , 数字密文 -> 数字明文.
	decrypt(D, N, int_ciphertext, int_plaintext);

	//-------------------------------------------------------------------
	// 3. 数字明文 -> 字符明文.
	for (int i = 0; i < LENGTH; i++)
		char_plaintext[i] = (char)int_plaintext[i];
	//-------------------------------------------------------------------
	// 4.打印
	printf("\n解密后的符号明文为:\n");
	puts(char_plaintext);
	printf("\n");

	system("pause");
	return 0;
}

//加密计算函数
void decrypt(int D, int N, int *int_ciphertext, int *int_plaintext)
{
	int num = 1;
	for (int i = 0; i < LENGTH; i++)
	{

		for (int j = 0; j < D; j++)
		{
			num = num * int_ciphertext[i] % N;
		}
		int_plaintext[i] = num;
		num = 1;
	}
}
```

