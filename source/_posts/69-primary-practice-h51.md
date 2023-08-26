---
url: primary-practice-h51
title: 微信红包的模拟算法
date: 2021-05-01 20:35:22
categories: [技术]
tags: [程序设计实践]
---

Primary Practice h51

<!--more-->

{% note info %}

完成分红包金额的程序，调用方式如下：

```java
HongBao home = new HongBao();
```

返回每个人应得的的钱数，分红包规则参考微信的随机红包

此题目结果不要求精确匹配

{% endnote %}

模拟一个微信随机红包的算法，查阅了一些资料，得知微信随机红包可能遵循以下规律（可靠性未知，且过于简单化）：

1. 单个红包最低金额为 0.01 元；
2. 单个红包最高金额为剩余钱数对剩余份数的平均值的二倍。
3. 抢到红包的钱数总和等于发出的钱数。

根据此，可以写出抢到红包的钱数：

```java
	public double getOneRedPackage(double remainMoney, int remainCount) {// 剩余钱数、剩余份数
		double min = 0.01;// 最小值
		double max = remainMoney / remainCount * 2;// 最大值
		double money;// 抢到的钱数
		if (remainCount == 1) {
			money = remainMoney;// 就剩一个了，抢到的就是剩下的钱
		} else {
			money = Math.random() * max;// 如果剩多个，抢到的就是0～max之间的随机数
			money = Math.max(money, min);// 保证抢到的大于min
		}
		money = (double) Math.round(money * 100) / 100;// 保留两位，精确到分
		return money;
	}
```

其中，`Math.random()` 可以返回 0.0～1.0 之间的随机数。

根据以上算法，可以进一步写出分红包的算法：

{% note info %}

@param total  红包总金额，以元为单位，精确到分，系统测试的时候保证总金额至少够每人分得 1 分钱

@param personCount 分红包的总人数 > 0

@return 每个人分得的钱数

规则遵循微信分红包规则 例如：

要求 每人分得的钱数总和 = total

每个人分得钱数必须是正数，且不能少于 1 分

{% endnote %}

```java
	public double[] getHongbao(double total,int personCount) {
		if (total < personCount * 0.01) {
			return null;// 发出红包份数太多/总钱数太少
		}
		double[] result = new double[personCount];// 抢到金额数组
		int remainCount = personCount;// 剩余份数
		double remainMoney = total;// 剩余钱数
		int i;
		for (i = 0; i < personCount; i++) {
			double oneMoney;// 一个
			oneMoney = getOneRedPackage(remainMoney, remainCount);// 一个
			remainCount--;// 剩余份数减一
			remainMoney -= oneMoney;// 剩余钱数减少
			result[i] = oneMoney;
		}
		return result;
	}
```

{% note success %}

#### 2021.5.13 更新

分红包时最大值不能太大，否则剩下的连一分钱都分不到了。

所以对最大值需要增加判断，保证其他人至少要分到一分钱。

修改后的 `getOneRedPackage`：

```java
	public double getOneRedPackage(double remainMoney, int remainCount) {
		double min = 0.01;
		double max1 = remainMoney / remainCount * 2;
		double max2 = remainMoney - (remainCount - 1) * 0.01;
		double max = Math.min(max1, max2);
		double money;
		if (remainCount == 1) {
			money = remainMoney;
		} else {
			money = Math.random() * max;
			money = Math.max(money, min);
		}
		money = (double) Math.round(money * 100) / 100;
		return money;
	}
```

{% endnote %}

[Source code](https://github.com/SuperPung/Primary-Practice-Homeworks/tree/master/src/com/huawei/classroom/student/h51)
