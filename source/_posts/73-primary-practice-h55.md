---
url: primary-practice-h55
title: 李白杜甫诗词分析
date: 2021-05-01 20:35:37
categories: [技术]
tags: [程序设计实践]
---

Primary Practice h55

<!--more-->

{% note info %}

用计算机来证明：为什么说李白是浪漫主义诗人、杜甫是现实主义诗人？

分析不同诗人使用一些汉字时候的特点。

1. 分析：不同诗人使用一个汉字的时候，将这些汉字组成什么词汇使用在诗句里面；
2. 按这些词汇出现的频率高低排序；
3. 只要是两个汉字连起来就视为一个词。

{% endnote %}

[上一次](https://superpung.com/oop-h13/) 分析比较了红楼梦的前八十回和后四十回，这一次分析比较李白和杜甫的诗词。

主体代码几乎和 h13 相同。

借用了 h13 的 `getTopNWords` 方法，属性有：

```java
	private String[] verses = new String[0];
	private Set<String> charSet = new HashSet<>();
```

改写 `getTopNWords` 方法：

- 返回值改为 `List<Map.Entry<String, Integer>>` 便于得到频率值；
- 对切割后子串的判断条件改为 `!charSet.contains(str.substring(0, 1)) && !charSet.contains(str.substring(1))`，即两个字都不在要判断的字集里时跳过。

```java
	public List<Map.Entry<String, Integer>> getTopNWords(int n){
		int i, j;
		Map<String, Integer> map = new HashMap<>();
		List<Map.Entry<String, Integer>> mapList;
		List<Map.Entry<String, Integer>> ans = new ArrayList<>();
		for (i = 1; i < this.verses.length; i++){
			String content = this.verses[i];
			for (j = 0; j < content.length() - 1; j++) {
				String str = content.substring(j, j + 2);
				if (!charSet.contains(str.substring(0, 1)) && !charSet.contains(str.substring(1))) {
					continue;
				}
				int count;
				count = map.getOrDefault(str, 0);
				map.put(str, count + 1);
			}
		}

		mapList = new ArrayList<>(map.entrySet());
		mapList.sort((o1, o2) -> o2.getValue().compareTo(o1.getValue()));

		for (i = 0; i < n; i++) {
			ans.add(mapList.get(i));
		}

		return ans;
	}
```

最终 `analysis` 方法只需处理文件读入和判断的字并打印结果：

```java
	public void analysis(String pathFilename, String chars) {
		String content = readFromTxt(pathFilename);
		List<Map.Entry<String, Integer>> result = new ArrayList<>();
		charSet = new HashSet<>(Arrays.asList(chars.split(";")));
		verses = content.split("[，。]");
		result = getTopNWords(10);
		for (Map.Entry<String, Integer> res : result) {
			System.out.println(res.getKey() + "\t" + res.getValue());
		}
	}
```

最后分析不同词汇（不完整）。

#### "春;夏;秋;冬;暑;寒;风;雨;雪;霜;露"

| 李白                                                         | 杜甫                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 春风	72<br/>风吹	51<br/>秋月	40<br/>清风	28<br/>秋风	25<br/>东风	24<br/>白雪	21<br/>长风	21<br/>秋浦	21<br/>秋霜	20 | 风尘	48<br/>秋风	30<br/>风吹	28<br/>天寒	27<br/>清秋	25<br/>春色	20<br/>春风	19<br/>风雨	17<br/>高秋	16<br/>云雨	16 |

这些是几个和季节有关的词汇，李白和杜甫在高频词上有所不同，李白喜欢“春风”、“风吹”、“秋月”，杜甫喜欢“风尘”、“秋风”。

#### "醉;酒;饮;杯"

| 李白                                                         | 杜甫                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 美酒	27<br/>一杯	21<br/>杯酒	14<br/>对酒	13<br/>三杯	11<br/>斗酒	11<br/>杯中	8<br/>饮酒	8<br/>置酒	8<br/>衔杯	7 | 酒酣	10<br/>杯酒	7<br/>醉眠	6<br/>嗜酒	6<br/>痛饮	6<br/>酒杯	5<br/>对酒	5<br/>杯中	5<br/>酒肉	5<br/>春酒	5 |

李白喝酒比杜甫多，而且酒量也比杜甫大。

#### "东;西;南;北"

| 李白                                                         | 杜甫                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 东风	24<br/>东海	23<br/>东山	20<br/>东流	19<br/>南山	19<br/>西来	16<br/>东南	12<br/>北斗	12<br/>西施	10<br/>北海	10 | 西南	15<br/>北风	14<br/>北斗	14<br/>东西	13<br/>东流	12<br/>南国	12<br/>西江	11<br/>南征	11<br/>西戎	9<br/>瀼西	9 |

李白更喜欢东风，杜甫总是想到北风。

#### "云;日;月;山;河"

| 李白                                                         | 杜甫                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 白日	62<br/>明月	59<br/>青云	46<br/>浮云	44<br/>秋月	40<br/>白云	34<br/>落日	32<br/>今日	28<br/>五月	26<br/>黄河	26 | 今日	33<br/>落日	29<br/>白日	27<br/>浮云	23<br/>江山	22<br/>巫山	20<br/>日月	20<br/>终日	18<br/>他日	18<br/>日暮	18 |

除去“白日”、“今日”等与自然景色无关的词汇，李白更喜欢“明月”，而杜甫更喜欢“落日”。

#### "水;天;玉;心"

| 李白                                                         | 杜甫                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 青天	54<br/>天地	53<br/>流水	34<br/>天上	31<br/>天子	28<br/>白玉	27<br/>绿水	26<br/>天下	22<br/>海水	17<br/>汉水	16 | 天下	35<br/>天地	32<br/>天子	29<br/>天寒	27<br/>江水	16<br/>寸心	13<br/>白水	13<br/>秋水	12<br/>天意	12<br/>水中	11 |

可见杜甫更加挂念天下苍生，忧国忧民。

#### "歌;愁"

| 李白                                                         | 杜甫                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 歌舞	9<br/>愁不	9<br/>人愁	9<br/>吴歌	9<br/>行歌	8<br/>歌白	8<br/>长歌	8<br/>愁杀	7<br/>歌钟	7<br/>棹歌	7 | 愁思	13<br/>穷愁	11<br/>客愁	10<br/>长歌	9<br/>高歌	8<br/>歌兮	7<br/>悲歌	6<br/>兮歌	6<br/>狂歌	5<br/>愁绝	5 |

可以看出李白更加浪漫，喜欢歌舞；而杜甫则悲怆多愁，连“歌”都是“悲歌”。

#### 总结

由于不同的出身、经历，李白和杜甫两人的诗歌创作风格有所不同。李白和杜甫彼此深情凝望，又相互辉映。一个浪漫豪放，一个忧国忧民；一个富有激情，一个臻于深刻。李白的创作尚意韵，是神来之笔；杜甫的创作尚法度，是典范圭臬。李白和杜甫代表两种精神人格、两种风格境界。二者并立，成为两座高峰、两座丰碑，他们风格不同，却伟大相通。

{% cq %}

李富于才，杜深于学。富于才者豪于情，深于学者笃于性。

诗原本性情。以性情为诗，所以凌驾一代，妙绝千古。

{% endcq %}
