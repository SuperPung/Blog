---
url: primary-practice-h58
title: 投票统计
date: 2021-05-06 21:12:08
categories: [技术]
tags: [程序设计实践]
---

Primary Practice h58

<!--more-->

{% note info %}

fileName 是一个投票的明细记录，里面逐行存放了

```
投票的时间（yyyy-MM-dd HH:mm:ss 格式） + \t + 投票的微信ID + \t + 候选人
```

存放按时间递增（但是可能出现同一秒出现若干条记录的情况）

现在需要完成投票统计的过程，具体要求如下：

- 1 个微信 ID 1 分钟内 最多投 1 票 多余的票数无效
- 1 个微信 ID 10 分钟内 最多只能投 5 票 多余的票无效
- 其中微信 ID 不固定，候选人姓名不固定
- 测试的时候要求 10 万行记录处理时间不超过 3 秒

{% endnote %}

很有趣的一道题，虽然最后的结果始终有 3%～5% 的误差（截止到 2021.5.6 晚 9:58，还是没有找到误差出现的地方）。

和之前分析的文件类似，每一行文本有相同的格式，此处我抽象为一个 Record 类。

假设已经有了 Record 类，根据要求，需要对每位投票者的投票操作进行分析，去除掉那些不符合要求的操作记录。所以投票者也应抽象为一 Voter 类。

# 0x00 Record

此 Record 类对应文件的一行，所以应有时间、投票者和候选人属性。但整体来看，每条记录可能有效也可能无效，所以再添加一有效性属性：

```java
    private Date date;
    private final String voterId;
    private final String candidate;
    private boolean valid;
```

构造方法只需要传入参数，初始化各个属性。难点在于时间 `Date` 类型的处理，需要引入 `DateFormat` 和 `SimpleDateFormat` 类。此时暂且将每一条记录都设为有效：

```java
    public Record(String date, String voterId, String candidate) {
        DateFormat fmt = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        try {
            this.date = fmt.parse(date);
        } catch (Exception e) {
            e.printStackTrace();
        }
        this.voterId = voterId;
        this.candidate = candidate;
        valid = true;
    }
```

定义各种 getter 和 setter：

```java
    public Date getDate() {
        return date;
    }

    public String getVoterId() {
        return voterId;
    }

    public String getCandidate() {
        return candidate;
    }

    public boolean isValid() {
        return valid;
    }

    public void setInvalid() {
        valid = false;
    }
```

最后，由于后续分析记录有效性的主要依据是时间，所以记录之间的先后顺序很重要。故 Record 类需要实现 `Comparable` 接口，重写 `compareTo` 方法：

```java
public class Record implements Comparable<Record> {
  	...
    @Override
    public int compareTo(Record o) {
        return date.compareTo(o.date);
    }
}
```

# 0x01 Voter

最主要的部分，本次实践的核心。

## 准备

作为一名投票者，区别身份的就是 ID，权利是一系列的投票记录：

```java
    private final String id;
    private final List<Record> records = new ArrayList<>();
    private static final int MAX_RECORDS_IN_TEN_MINUTES = 5; // 十分钟内最多投票的次数
```

{% note warning %}

可能会联想到，Record 类的属性中，关于投票者为什么只存一个 String 而不是 Voter？

因为 Voter 会包含 Record，避免套娃，减少冗余，而且 Record 只是一个中间类，目的是根据 Record 的 voterId 将其添加到对应 voter 的 records 中，所以只需要一个 String。

{% endnote %}

构造方法只传入 ID：

```java
    public Voter(String id) {
        this.id = id;
    }
```

文件中同一位投票者的投票记录不一定是连续的，所以投票记录需要逐条添加：

```java
    public void addRecord(Record record) {
        records.add(record);
    }
```

另外，从整体来看，投票者有若干个，可构成集合，为便于后续获取投票者的集合，此处需实现可序列化接口，重写哈希码和相等方法：

```java
public class Voter implements Serializable {
		...
    @Override
    public int hashCode() {
        return id.hashCode();
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Voter)) {
            return false;
        }
        return ((Voter) o).id.equals(id);
    }
}
```

## 去除无效记录

核心中的核心。

再次回顾一下，具体要求如下：

- 1 个微信 ID 1 分钟内 最多投 1 票 多余的票数无效
- 1 个微信 ID 10 分钟内 最多只能投 5 票 多余的票无效

为了判断，记录存储必须是按照时间递增的。1 分钟内最多 1 票，这个条件容易判断，只需要记录当前记录的时间，计算出 1 分钟之后的时间，再和下一条记录比对。但是要保证 10 分钟内最多 5 票，就有些困难。一方面，不能先只判断 1 分钟，因为两个条件互相影响，必须按时间顺序逐条判断；另一方面，判断 10 分钟不能按 0～10、10～20 判断，因为要保证任意 10 分钟内都最多只有 5 条，所以我利用 buffer 一次读入 5 条记录，这 5 条记录一定是满足 10 分钟条件的，在此基础上进行 1 分钟条件判断。

{% note primary %}
#### 2021.5.7 更新

之前读入 5 条记录后只判断了 1 分钟条件，忘记判断 10 分钟条件。即这 5 条记录中可能从中间某一条记录开始，时间已经在第一条的 10 分钟以外，此时这条记录之前的记录一定为有效，且 buffer 应后移相应记录条数。

相应地，移动后应重新判断 1 分钟条件和 10 分钟条件，故定义了 `isBufFinish` 控制循环。

且，5 条记录内，1 分钟条件应优先于 10 分钟条件，所以加了 `break`。

修改前：

```
1007 ms cost!
wang 的误差为：5.09%
zhao 的误差为：4.23%
zhang 的误差为：3.36%
li 的误差为：4.29%
```

修改后：

```
901 ms cost!
wang 的误差为：4.65%
zhao 的误差为：2.7%
zhang 的误差为：4.57%
li 的误差为：4.05%
```
{% endnote %}

buffer 内部判断完成，需将 buffer 整体后移。若 buffer 之后的记录还在 10 分钟内，一定为无效。直到找到下一条 10 分钟之外的记录。每次 buffer 只移动一条记录，加 `break`。

```java
    public void removeInvalidRecords() {
        // 按时间排序
        records.sort(Record::compareTo);
        // 容量为5的buffer
        List<Record> buffer = new LinkedList<>();
        int next = 0;
        for (int i = 0; i < Math.min(MAX_RECORDS_IN_TEN_MINUTES, records.size()); i++) {
            buffer.add(records.get(next++));
        }
        // 每条记录判断
        do {
            int i, j;
            Date curDate = buffer.get(0).getDate();
            Date oneMinLater = new Date(curDate.getTime() + 60000);
            Date tenMinLater = new Date(curDate.getTime() + 600000);
            boolean isBufFinish = false;
            while (!isBufFinish) {
                // 去除不满足1分钟条件的记录
                for (i = 1; i < buffer.size(); i++) {
                    Date nextDate = buffer.get(i).getDate();
                    if (nextDate.compareTo(oneMinLater) < 0) {
                        buffer.get(i).setInvalid();
                        buffer.remove(i--);
                        if (next < records.size()) {
                            buffer.add(records.get(next++));
                        }
                    } else {
                        oneMinLater = new Date(nextDate.getTime() + 60000);
                    }
                }
                // （5.7新增）判断buffer内是否超过10分钟，不设无效，只移动
                for (i = 1; i < buffer.size(); i++) {
                    Date nextDate = buffer.get(i).getDate();
                    if (nextDate.compareTo(tenMinLater) >= 0) {
                        tenMinLater = new Date(nextDate.getTime() + 600000);
                        for (j = 0; j < i; j++) {
                            buffer.remove(j);
                            buffer.add(records.get(next++));
                        }
                        isBufFinish = false;
                        break;
                    } else {
                        isBufFinish = true;
                    }
                }

            }
            // 判断buffer后的下一条记录是否超过10分钟
            while (next < records.size()) {
                Record nextRecord = records.get(next);
                if (nextRecord.getDate().compareTo(tenMinLater) < 0) {
                    nextRecord.setInvalid();
                    next++;
                } else {
                    buffer.remove(0);
                    buffer.add(nextRecord);
                    next++;
                    break;
                }
            }
        } while (next < records.size());
    }
```

至此，最核心的算法已完成。但 5.7 更新后仍有 4.7% 以下的误差，一定是有边界情况未考虑，待更新。

## 获取有效记录

用哈希映射存储，遍历有效记录。

```java
    public Map<String, Integer> getCandidates() {
        Map<String, Integer> candidates = new HashMap<>();
        for (Record record : records) {
            if (!record.isValid()) {
                continue;
            }
            String candidate = record.getCandidate();
            int count = candidates.getOrDefault(candidate, 0);
            candidates.put(candidate, count + 1);
        }
        return candidates;
    }
```

# 0x02 VoteRecord

顶层类，按行读取文件，每行存一 Record 实例：

```java
	public List<Record> readLines(String filename) {
		String line = "";
		Reader reader = null;
		List<Record> result = new ArrayList<>();
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
				String[] recordItems = line.split("\t");
				String date = recordItems[0];
				String voterId = recordItems[1];
				String candidate = recordItems[2];
				Record record = new Record(date, voterId, candidate);
				result.add(record);
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
		return result;
	}
```

主体 `calcRecording`，将读取到的 Record 列表存储，存所有 voter 的 id，存所有 voter，存所有 voterId 和 voter 的对应关系，存所有的候选人和其所得票数：

```java
	public Map<String,Integer> calcRecording(String fileName){
		List<Record> records = readLines(fileName);
		Set<String> voterIds = new HashSet<>();
		Set<Voter> voters = new HashSet<>();
		Map<String, Voter> voterMap = new HashMap<>();
		Map<String, Integer> candidates = new HashMap<>();

		for (Record record : records) {
			voterIds.add(record.getVoterId());
		}

		for (String voterId : voterIds) {
			Voter voter = new Voter(voterId);
			voters.add(voter);
			voterMap.put(voterId, voter);
		}

		for (Record record : records) {
			Voter voter = voterMap.get(record.getVoterId());
			voter.addRecord(record);
		}

		for (Voter voter : voters) {
			voter.removeInvalidRecords();
			Map<String, Integer> oneCandidates = voter.getCandidates();
//			System.out.println(voter.getRecords().size());
			Set<Map.Entry<String, Integer>> oneCandidatesNameCount = oneCandidates.entrySet();
			for (Map.Entry<String, Integer> entry : oneCandidatesNameCount) {
				String candidate = entry.getKey();
				int count = entry.getValue();
				if (candidates.containsKey(candidate)) {
					count += candidates.get(candidate);
				}
				candidates.put(candidate, count);
			}
		}

		return candidates;
	}
```


{% note warning %}
#### 误差待消除

{% endnote %}

[Source code](https://github.com/SuperPung/Primary-Practice-Homeworks/tree/master/src/com/huawei/classroom/student/h58)
