---
url: primary-practice-h52
title: 质因数分解
date: 2021-05-01 20:35:28
categories: [技术]
tags: [程序设计实践]
---

Primary Practice h52

<!--more-->

{% note info %}

将 num 进行质因数分解，将分解到的质因数放到 Set 里面返回

{% endnote %}

如果每次对 num 分解都从最小的质数开始，那么分解的结果一定都是质数。

```java
	public Set<Integer> decompose(int num) {
		Set<Integer> result = new HashSet<>();// 分解的结果
		if (num < 0) {
			return null;// 负数不分解
		} else if (num == 1) {
			result.add(1);// 1就是1
		} else {
			int i = 2;// 先从最小质数开始
			while (i <= num) {
				if (num % i == 0) {
					result.add(i);// 整除，i是num因数
					num = num / i;// 取剩余部分，下次继续分解
					i = 2;// 下次分解一定要从头开始
				} else {
					i++;// 不整除，只自增1即可，不必直接到下一个质数（都不能被2整除，怎么可能被4整除）
				}
			}
		}
		return result;
	}
```
