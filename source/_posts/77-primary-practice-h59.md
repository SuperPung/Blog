---
url: primary-practice-h59
title: 化学反应分析
date: 2021-05-06 21:12:11
categories: [技术]
tags: [程序设计实践]
---

Primary Practice h59

<!--more-->

{% note info %}

根据 reactionFile 给出的一系列反应， 判断一个体系中根据 init 物质，判断出最后可能都存在什么物质

本文件中存放了若干的化学反应方程式（总数量不会超过 1000 个）

本文件中存放了一系列的化学反应 #表示注释

化学反应以 = 分为了左侧和右侧；不同化合物之间至少有一个空格

A + B = C + D 意味着体系中如果有了 A B 就可以生成 C D，同样如果有 C D 也可以生成 A B

所有反应，反应物前系数均为 1

{% endnote %}

1. 从文件中读取反应方程式，用~~哈希映射~~ `Reaction` 类存储：反应物存储为一个集合、生成物存储为一个集合，二者一一对应（可逆反应，反应物和生成物可互换）；
2. 遍历方程式映射，如果 result 含有所有的反应物**且不含有所有的生成物（不含、含一部分）**，则将生成物加入 result；依据生成物加反应物同理；
3. 若遍历一次，result 未添加新的物质，说明反应达到平衡。

{% note success %}

#### 2021.5.13 更新

文件读取方程式不能用哈希映射存储，因为可能存在反应物相同而生成物不同的情况。

所以新建了 `Reaction` 类。

{% endnote %}

修改后：

```java
package com.huawei.classroom.student.h59;

import java.util.Set;

/**
 * @author super
 */
public class Reaction {
    private Set<String> reactant;
    private Set<String> product;

    public Reaction(Set<String> reactant, Set<String> product) {
        this.reactant = reactant;
        this.product = product;
    }


    public Set<String> getReactant() {
        return reactant;
    }

    public Set<String> getProduct() {
        return product;
    }
}
```

```java
package com.huawei.classroom.student.h59;

import java.io.*;
import java.util.*;

/**
 * @author super
 */
public class ReactionTools {

	/**
	 * 根据reactionFile给出的一系列反应， 判断一个体系中根据init物质，判断出最后可能都存在什么物质
	 * @param reactionFile 体系中初始反应物
	 * @param initComponents 体系中初始反应物
	 * @return 最后体系中存在的全部物质
	 */
	public Set<String> findAllComponents(String reactionFile,Set<String> initComponents){
		List<Reaction> reactions = readLines(reactionFile);
		Set<String> result = new HashSet<>(initComponents);
		int newAddCount = initComponents.size();
		while (newAddCount != 0) {
			newAddCount = 0;
			for (Reaction reaction : reactions) {
				// contain or not contain, that is a question
				if (result.containsAll(reaction.getReactant()) && !result.containsAll(reaction.getProduct())) {
					result.addAll(reaction.getProduct());
					newAddCount++;
				} else if (result.containsAll(reaction.getProduct()) && !result.containsAll(reaction.getReactant())) {
					result.addAll(reaction.getReactant());
					newAddCount++;
				}
			}
		}
		return result;
	}

	public List<Reaction> readLines(String filename) {
		String line;
		Reader reader = null;
		List<Reaction> result = new ArrayList<>();
		try {
			reader = new FileReader(filename);
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}
		if (reader == null) {
			return null;
		}
		LineNumberReader lineReader = new LineNumberReader(reader);

		try {
			while (true) {
				line = lineReader.readLine();
				if (line == null) {
					break;
				}
				if (line.trim().length() == 0 || line.startsWith("#")) {
					continue;
				}
				String[] reaction = line.split("=");
				String left = reaction[0];
				String right = reaction[1];
				String[] lefts = left.split("\\ \\+\\ ");// regular
				Set<String> leftSet = new HashSet<>();
				for (String s : lefts) {
					leftSet.add(s.trim());
				}
				String[] rights = right.split("\\ \\+\\ ");
				Set<String> rightSet = new HashSet<>();
				for (String s : rights) {
					rightSet.add(s.trim());
				}
				result.add(new Reaction(leftSet, rightSet));
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
		return result;
	}
}
```

{% note no-icon 修改前 %}

```java
package com.huawei.classroom.student.h59;

import java.io.*;
import java.util.*;

/**
 * @author super
 */
public class ReactionTools {

	/**
	 * 根据reactionFile给出的一系列反应， 判断一个体系中根据init物质，判断出最后可能都存在什么物质
	 * @param reactionFile 体系中初始反应物
	 * @param initComponents 体系中初始反应物
	 * @return 最后体系中存在的全部物质
	 */
	public Set<String> findAllComponents(String reactionFile,Set<String> initComponents){
		Map<Set<String>, Set<String>> reactions = readLines(reactionFile);
		Set<String> result = new HashSet<>(initComponents);
		Set<Map.Entry<Set<String>, Set<String>>> reactionsEntrySet = reactions.entrySet();
		int newAddCount = initComponents.size();
		while (newAddCount != 0) {
			newAddCount = 0;
			for (Map.Entry<Set<String>, Set<String>> entry : reactionsEntrySet) {
				// contain or not contain, that is a question
				if (result.containsAll(entry.getKey()) && !result.containsAll(entry.getValue())) {
					result.addAll(entry.getValue());
					newAddCount++;
				} else if (result.containsAll(entry.getValue()) && !result.containsAll(entry.getKey())) {
					result.addAll(entry.getKey());
					newAddCount++;
				}
			}
		}
		return result;
	}

	public Map<Set<String>, Set<String>> readLines(String filename) {
		String line;
		Reader reader = null;
		Map<Set<String>, Set<String>> result = new HashMap<>();
		try {
			reader = new FileReader(filename);
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}
		if (reader == null) {
			return null;
		}
		LineNumberReader lineReader = new LineNumberReader(reader);

		try {
			while (true) {
				line = lineReader.readLine();
				if (line == null) {
					break;
				}
				if (line.trim().length() == 0 || line.startsWith("#")) {
					continue;
				}
				String[] reaction = line.split("=");
				String left = reaction[0];
				String right = reaction[1];
				String[] lefts = left.split("\\ \\+\\ ");// regular
				Set<String> leftSet = new HashSet<>();
				for (String s : lefts) {
					leftSet.add(s.trim());
				}
				String[] rights = right.split("\\ \\+\\ ");
				Set<String> rightSet = new HashSet<>();
				for (String s : rights) {
					rightSet.add(s.trim());
				}
				result.put(leftSet, rightSet);
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
		return result;
	}
}
```

{% endnote %}

[Source code](https://github.com/SuperPung/Primary-Practice-Homeworks/tree/master/src/com/huawei/classroom/student/h59)
