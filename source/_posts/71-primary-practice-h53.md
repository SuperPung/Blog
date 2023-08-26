---
url: primary-practice-h53
title: 兔子繁殖问题
date: 2021-05-01 20:35:30
categories: [技术]
tags: [程序设计实践]
---

Primary Practice h53

<!--more-->

{% note info %}

1 对兔子出生以后经过 180 天可以生出一窝（2 对）兔子，以后每隔 90 天繁殖一次生出一窝（2 对）兔子

每对兔子的寿命是 700 天

@param startCount 第 0 天 开始的时候初始的兔子对数

@param days 经过的天数

@return 目前系统中存活的兔子的对数

{% endnote %}

一道很有趣的题，不是简单的递归问题。无需找规律，只需模拟出真实的兔子繁殖。

可以发现，需要抽象出兔子类（个体）、兔子窝类（集体），才能完整模拟兔子繁殖。

# 0x00 Rabbit

根据要求，兔子的属性有它的“年龄”（以天计）、“寿命”（以天计，700 天）、生存状况（生存天数达到寿命则死亡）：

```java
    private int dayAge;
    private boolean dead;
    private final int lifeDayTime = 700;
```

兔子刚出生时，“年龄”为 0：

```java
    public Rabbit() {
        dayAge = 0;
        dead = false;
    }
```

获取兔子的“年龄”：

```java
    public int getDayAge() {
        return dayAge;
    }
```

兔子经过一天的成长（只有此时会发生兔子死亡，所以需要判断）：

```java
    public void growOneDay() {
        if (isDead()) {
            return;
        }
        dayAge++;
        if (dayAge > lifeDayTime) {
            dead = true;
        }
    }
```

获取兔子的生存状况：

```java
    public boolean isDead() {
        return dead;
    }
```

# 0x01 RabbitNest

上面的 Rabbit 类代表兔子个体，此 RabbitNest 类代表兔子集体：

```java
    private final List<Rabbit> rabbits = new ArrayList<>();
```

兔子窝开始的时候，需要几只初始兔子：

```java
    public RabbitNest(int startCount) {
        for (int i = 0; i < startCount; i++) {
            Rabbit rabbit = new Rabbit();
            rabbits.add(rabbit);
        }
    }
```

经过一天，兔子们长大了：

```java
    public void rabbitsGrow() {
        for (Rabbit rabbit : rabbits) {
            rabbit.growOneDay();
        }
    }
```

经过一天，有的兔子达到的寿命，离开了：

```java
    public void rabbitsDead() {
        rabbits.removeIf(Rabbit::isDead);
    }
```

经过一天，有的兔子生小兔子了：

```java
    public void rabbitsBearLittleRabbits() {
        List<Rabbit> rabbitsBornOneDay = new ArrayList<>();// 这一天生的兔子
        for (Rabbit rabbit : rabbits) {// 这一天存活的兔子，不包括新生的兔子
            int dayAge = rabbit.getDayAge();// 某兔子的年龄
            boolean isRabbitCanBear = dayAge == 180 || (dayAge > 180 && (dayAge - 180) % 90 == 0);// 可以生小兔子的条件
            if (isRabbitCanBear && !rabbit.isDead()) {// 可生且未死亡
                Rabbit rabbit1 = new Rabbit();// 生了一对
                Rabbit rabbit2 = new Rabbit();// 又生了一对
                rabbitsBornOneDay.add(rabbit1);
                rabbitsBornOneDay.add(rabbit2);// 加入到这一天新生的兔子中
            }
        }
        rabbits.addAll(rabbitsBornOneDay);// 这一天新生的兔子加入到兔子集体
    }
```

经过一天，兔子们主要经历以上三个阶段：成长、死亡、新生：

```java
    public void oneDayPass() {
        rabbitsGrow();
        rabbitsDead();
        rabbitsBearLittleRabbits();
    }
```

获取兔子窝中兔子数量：

```java
    public int getLivingRabbits() {
        return rabbits.size();
    }
```

# 0x02 RabbitCount

创建了兔子和兔子窝，想要知道兔子的数量也就简单了：只需创建一个兔子窝，经历相应的天数，再获取其中兔子的数量：

```java
	public int getLivingRabbit(int startCount,int days) {
		RabbitNest rabbitNest = new RabbitNest(startCount);
		for (int i = 0; i < days; i++) {
			rabbitNest.oneDayPass();
		}
		return rabbitNest.getLivingRabbits();
	}
```
