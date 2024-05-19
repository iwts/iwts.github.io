---
layout: post
title: "hdoj-5115 14年ICPC北京站D题 Dire Wolf"
author: "iwts li"
date: 2018-02-19 15:45:36
categories: 题解
header-style: text
tags:
  - ACM
  - 动态规划
  - 区间DP
description: "hdoj-5115 | 14年ICPC北京站D题 Dire Wolf 题解"
---

# 题目链接  

[hdoj-5115 Dire Wolf](http://acm.hdu.edu.cn/showproblem.php?pid=5115)

# 题目

Dire Wolf
---------

**Time Limit: 5000/5000 MS (Java/Others)    Memory Limit: 512000/512000 K (Java/Others)  
Total Submission(s): 3067    Accepted Submission(s): 1813  
**  
  

Problem Description

Dire wolves, also known as Dark wolves, are extraordinarily large and powerful wolves. Many, if not all, Dire Wolves appear to originate from Draenor.  
Dire wolves look like normal wolves, but these creatures are of nearly twice the size. These powerful beasts, 8 - 9 feet long and weighing 600 - 800 pounds, are the most well-known orc mounts. As tall as a man, these great wolves have long tusked jaws that look like they could snap an iron bar. They have burning red eyes. Dire wolves are mottled gray or black in color. Dire wolves thrive in the northern regions of Kalimdor and in Mulgore.  
Dire wolves are efficient pack hunters that kill anything they catch. They prefer to attack in packs, surrounding and flanking a foe when they can.  
— Wowpedia, Your wiki guide to the World of Warcra  
  
Matt, an adventurer from the Eastern Kingdoms, meets a pack of dire wolves. There are N wolves standing in a row (numbered with 1 to N from left to right). Matt has to defeat all of them to survive.  
  
Once Matt defeats a dire wolf, he will take some damage which is equal to the wolf’s current attack. As gregarious beasts, each dire wolf i can increase its adjacent wolves’ attack by b i. Thus, each dire wolf i’s current attack consists of two parts, its basic attack ai and the extra attack provided by the current adjacent wolves. The increase of attack is temporary. Once a wolf is defeated, its adjacent wolves will no longer get extra attack from it. However, these two wolves (if exist) will become adjacent to each other now.  
  
For example, suppose there are 3 dire wolves standing in a row, whose basic attacks ai are (3, 5, 7), respectively. The extra attacks b i they can provide are (8, 2, 0). Thus, the current attacks of them are (5, 13, 9). If Matt defeats the second wolf first, he will get 13 points of damage and the alive wolves’ current attacks become (3, 15).  
  
As an alert and resourceful adventurer, Matt can decide the order of the dire wolves he defeats. Therefore, he wants to know the least damage he has to take to defeat all the wolves.

  

Input

The first line contains only one integer T , which indicates the number of test cases. For each test case, the first line contains only one integer N (2 ≤ N ≤ 200).  
  
The second line contains N integers a i (0 ≤ a i ≤ 100000), denoting the basic attack of each dire wolf.  
  
The third line contains N integers b i (0 ≤ b i ≤ 50000), denoting the extra attack each dire wolf can provide.

  

Output

For each test case, output a single line “Case #x: y”, where x is the case number (starting from 1), y is the least damage Matt needs to take.  

  

Sample Input

2 3 3 5 7 8 2 0 10 1 3 5 7 9 2 4 6 8 10 9 4 1 2 1 2 1 4 5 1

  

Sample Output

Case #1: 17 Case #2: 74

_Hint_

In the ﬁrst sample, Matt defeats the dire wolves from left to right. He takes 5 + 5 + 7 = 17 points of damage which is the least damage he has to take.

  

Source

[2014ACM/ICPC亚洲区北京站-重现赛（感谢北师和上交）](http://acm.hdu.edu.cn/search.php?field=problem&key=2014ACM%2FICPC%D1%C7%D6%DE%C7%F8%B1%B1%BE%A9%D5%BE-%D6%D8%CF%D6%C8%FC%A3%A8%B8%D0%D0%BB%B1%B1%CA%A6%BA%CD%C9%CF%BD%BB%A3%A9&source=1&searchmode=source)

# 题意

描述了一种狼，这种狼攻击的时候会排成一排摆阵，每只狼有2个属性，攻击力与对周围狼的buff。

每只狼的buff影响了左边的狼与右边的狼。

单个狼的最终攻击力是该狼的攻击力+左边狼给加的buff+右边狼给加的buff。例如：（题目中的例子）

| | 第一只狼 | 第二只狼 | 第三只狼 |
| :----: | :----: | :----: | :----: |
| 狼的攻击力加成buff | 8 | 2 | 0 |
| 狼的基础攻击力 | 3 | 5 | 7 |

那么第二只狼的攻击力是：5+8+0 = 13

Matt要正面刚狼群，他每杀一匹狼，就要受到跟狼的总攻击力相同的伤害，求Matt以怎样的顺序杀狼才能受到最小的伤害。

# 题解

其实就是区间dp，我们可以定义状态dp\[i\]\[j\]表示Matt要杀完第i只到第j只狼所受的最低伤害，那么最终dp\[1\]\[n\]的值就是答案。

我们可以定义2个数组，wolves和wol\_ex，表示每只狼对应的基础攻击力与攻击力buff。

那么杀a狼时a狼的最终伤害是wolves\[a\]+wol\_ex\[a-1\]+wol_ex\[a+1\]。可以枚举i<=k<=j，表示在区间i、j内杀k狼的最小伤害。

3重for循环即可解决问题。

# C++ 代码 52ms

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<cmath>
#include<algorithm>
#include<map>
#include<utility>
#define TIME std::ios::sync_with_stdio(false)
#define LL long long
#define MAX 210
#define INF 0x3f3f3f3f

using namespace std;

int wolves[MAX];
int wol_ex[MAX];
int dp[MAX][MAX];

int Min( int a, int b ) {
	return a < b ? a : b;
}

int main() {
	TIME;
	int N, I;
	cin > > I;
	for ( int flag = 1; flag <= I; flag++) {
		cin > > N;
		for ( int i = 1; i <= N; i++) {
			cin > > wolves[i];
		}
		for ( int i = 1; i <= N; i++) {
			cin > > wol_ex[i];
		}
		memset(dp, 0, sizeof(dp ) );
		for (int i = 1; i <= N; i++) {
			dp[i][i] = wolves[i];
		}
		for (int i = N; i > 0; i-- ) {
			for (int j = i; j <= N; j++) {
				dp[i][j] = INF;
				for (int k = i; k <= j; k++) {
					int heat = wolves[k] + wol_ex[i-1] + wol_ex[j+1];
					dp[i][j] = Min(dp[i][j], dp[i][k-1] + dp[k+1][j] + heat);
				}
			}
		}
		cout << "Case #" << flag << ": ";
		cout << dp[1][N] << endl;
	}
	system("pause");
	return 0;
}
```

# 代码的简单解释

1. 由于求的是最小值，那么就初始化dp数组全部为0，i与j确定以后将dp\[i\]\[j\]的值设置为无限大即可。
2. dp\[i\]\[i\]是有意义的，表示区间只有1只狼，并且就杀这只狼。
3. 枚举k的时候分左右区间，i到k-1与k+1到j是左右区间，不能包括k，否则加上heat值，等于k狼被杀了2次。
4. k狼所受的buff应该是第i-1只狼与第j+1只狼，因为我们定义dp\[i\]\[j\]表示i、j只狼已经被杀了。