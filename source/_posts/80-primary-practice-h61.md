---
url: primary-practice-h61
title: 疾病传播模拟
date: 2021-05-22 20:35:30
categories: [技术]
tags: [程序设计实践]
---

Primary Practice h61

<!--more-->

{% note info %}

#### Param

| 方法                                                         | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `getImmuEffect()` / `setImmuEffect(double immuEffect)`       | 疫苗免疫率，可降低传染率                                     |
| `getImmuRate()` / `setImmuRate(double immuRate)`             | 人群中注射疫苗的人的比例，随机                               |
| `getCityPopulation()` / `setCityPopulation(int cityPopulation)` | 城市人口数                                                   |
| `getFamilySize()` / `setFamilySize(int familySize)`          | 每个家庭人口数（按序）                                       |
| `getCompanySize()` / `setCompanySize(int companySize)`       | 每个公司人口数（随机）                                       |
| `getSpreadRateFamily()` / `setSpreadRateFamily(double spreadRateFamily)` | 家庭传染率（晚上）                                           |
| `getSpreadRateCompany(double spreadRateCompany)`             | 公司传染率（白天）                                           |
| `getLatentPeriod()` / `setLatentPeriod(int latentPeriod)`    | 潜伏期，此时病人具有传染性，但仍正常上班生活，度过潜伏期后排队治疗 |
| `getHealingRateHome()` / `setHealingRateHome(double healingRateHome)` | 在家每天自愈概率                                             |
| `getDeathRateHome()` / `setDeathRateHome(double deathRateHome)` | 在家每天死亡概率                                             |
| `getHealingRateHospital()` / `setHealingRateHospital(double healingRateHospital)` | 在医院每天治愈概率                                           |
| `getDeathRateHospital()` / `setDeathRateHospital(double deathRateHospital)` | 在医院每天死亡概率                                           |
| `getHospitalSize()` / `setHospitalSize(int hospitalSize)`    | 医院床位数                                                   |
| `getInitPatients()` / `setInitPatients(List<Integer> initPatients)` | 初始病人编号                                                 |

#### SimResult

| 方法                                          | 含义           |
| --------------------------------------------- | -------------- |
| `getLatents()` / `setLatents(int latents)`    | 潜伏期状态人数 |
| `getDeaths()` / `setDeaths(int deaths)`       | 死亡人数       |
| `getCured()` / `setCured(int cured)`          | 自愈+痊愈人数  |
| `getPatients()` / `setPatients(int patients)` | 患病人数       |

{% endnote %}

和 [h53](https://superpung.com/primary-practice-h53) 问题类似，确保你已经理解 h53。

{% note warning %}

#### 本文尚未完成，但 h62 已经完成

{% endnote %}

考虑一个人的状态，有：健康->潜伏期->患病->自愈、治愈/死亡。

开始的思路是新建一个 `Citizen` 类，保存一个人的状态和活动。但模拟的时候需要另外定义许多组合，比如家庭、公司、医院等，代码复杂而且算法效率低。运行一次需要 70 多秒，结果还不对。考虑到可能是公司成员设置问题，影响效率、疫苗注射给谁，随机问题不能每次循环十万次、可不可以只保存不健康的人，每天不再循环健康者……改进后仍无果，遂改用老师的思路。

发现我改进后和老师的思路类似，只不过缺少抽象。为了减少模拟过程中的计算，应将家庭和公司抽象为地点类、医院抽象为医院类。

由于许多地方用到概率，直接在工具类中定义，提高可读性、简洁度。

家庭和公司包含成员“市民”，同时市民也包含属性“属于哪个家庭/公司”。

定义 `boolean` 变量时，应使用过去分词或形容词，调用它的方法名为 `is...`。

潜伏期默认为 -1 更好。

市民不用写多个“spendOneDay”，写一个分多情况就可以。

判断一个人是否得病，可以抽象为一个方法，属于一个市民。

潜伏期在家或在公司都会传染其他人，而且过了潜伏期，如果在家等待床位，也会传染其他人。所以判断家庭和公司内是否有传染源，需要不同的判断方式。

抽象的好处，住院的病人和等待住院的病人都可以放在 `Hospital` 类中。

疫苗注射人数可以不准确，直接对一个人按概率注射。

果然设置公司用到了 `shuffle`。

之前模拟一天的市民感染需要住院、患者治愈或自愈需要出院移动到 `Hospital` 中。

以前从个体考虑，很复杂；现在从整体考虑，并加以抽象，更简单。

传染给公司其他人，不再从传染源考虑，而是从公司考虑，每天查看公司里面是否有传染源，家庭同理；之前每个人判断他的同事，现在公司抽象为一个带有 id 的类。一个人每天如果健康则判断周围环境有无传染源、如果处于潜伏期则潜伏期增长一天并判断是否超过一定天数患病（患病则直接在家等待）、如果在家或在医院等待则判断是否自愈/治愈或死亡。

上层类调用下层类的实例，可以将自身传参过去，这样下层类就可以访问上层类了。
