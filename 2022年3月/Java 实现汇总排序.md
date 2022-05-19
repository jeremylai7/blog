# Java 实现汇总排序

> 排序在系统中经常能用到，一般可以在数据库做排序，也可以在服务端做排序。在数据库一般使用 `order by` 排序。而服务端也是使用快排。本期使用汇总排序。

## 问题
统计销售数据，每个销售员都有对应的部门和销售量，现在要统计销售数据。

要求**部门总销量递减排序，相同部门的也按照递减排序**。

比如：

| 销售员 | 部门 | 销售额 |
| :-----:| :----: | :----: |
| A | 南部 | 100w |
| B | 南部 | 20W |
| C | 北部 | 30W |
| D | 北部 | 70W |
| E | 北部 | 40W |
| F | 东部 | 150W|

根据汇总排序:

| 部门 | 销售额 |
| :----: | :----: |
| 东部 | 150W |
| 北部 | 130W|
| 南部 | 120W|

然后根据先按照部门总和排序，相同部门按照递减排序:

| 销售员 | 部门 | 销售额 |
| :-----:| :----: | :----: |
| F | 东部 | 150W |
| D | 北部 | 70W |
| E | 北部 | 40W |
| C | 北部 | 30W |
| E | 南部 | 100W |
| F | 南部 | 20W|

## 解决方案

前期创建 `model`
```
public class SalesmanStatistic {


    public SalesmanStatistic(String salesman, String department, Integer amount) {
        this.salesman = salesman;
        this.department = department;
        this.amount = amount;
    }

    @Override
    public String toString() {
        return "{salesman='" + salesman + '\'' +
                ", department='" + department + '\'' +
                ", amount=" + amount +
                '}';
    }

    private String salesman;

    private String department;

    private Integer amount;
}
```

添加数据
```
        // 填充数据
        List<SalesmanStatistic> list = new ArrayList<>();
        SalesmanStatistic statistic1 = new SalesmanStatistic("A","南方",100);
        SalesmanStatistic statistic2 = new SalesmanStatistic("B","南方",20);
        SalesmanStatistic statistic3 = new SalesmanStatistic("C","北方",30);
        SalesmanStatistic statistic4 = new SalesmanStatistic("D","北方",70);
        SalesmanStatistic statistic5 = new SalesmanStatistic("E","北方",40);
        SalesmanStatistic statistic6 = new SalesmanStatistic("F","东方",150);
        list.add(statistic1);
        list.add(statistic2);
        list.add(statistic3);
        list.add(statistic4);
        list.add(statistic5);
        list.add(statistic6);
```

使用两个 `map`，`key` 都是存部门，第一个 `map` 的 `value` 销售额的总额，第二个是存同部门的列表。

```
        Map<String,Integer> sumMap = new HashMap<>();
        Map<String, List<SalesmanStatistic>> listMap = new HashMap<>();
        // 把数据放在 map 里面
        for (SalesmanStatistic statistic : list) {
            String key = statistic.getDepartment();
            Integer amount = statistic.getAmount();
            sumMap.put(key,sumMap.getOrDefault(key,0) + amount);
            List<SalesmanStatistic> subList = listMap.get(key);
            if (subList == null) {
                subList = new ArrayList<>();
            }
            subList.add(statistic);
            listMap.put(key,subList);
        }
```
首先进行总量 `sumMap` 排序,就是对 `sumMap` 的 `value` 进行排序：
```
        // 对总和 sumMap 排序
        List<Map.Entry<String,Integer>> sumMapList = new ArrayList<>(sumMap.entrySet());
        sumMapList.sort((o1, o2) -> o2.getValue().compareTo(o1.getValue()));
```
以上获取到总和 `sumMap` 的列表，从大到小排列，然后遍历每个数据，根据 `key` 匹配到 `listMap` 的 `key`。首先获取对 `listMap` 的 `value` 列表进行排序，然后把 `list`  添加到总的集合里面。
```
        List<SalesmanStatistic> list = new ArrayList<>();
        for (Map.Entry<String,Integer> entry : sumMapList) {
            List<SalesmanStatistic> list1 =  listMap.get(entry.getKey());
            list1.sort((o1, o2) -> o2.getAmount().compareTo(o1.getAmount()));
            list.addAll(list1);
        }
        list.stream().forEach(list3 -> System.out.println(list3.toString()));
```

打印输出结果：
```
{salesman='F', department='东方', amount=150}
{salesman='D', department='北方', amount=70}
{salesman='E', department='北方', amount=40}
{salesman='C', department='北方', amount=30}
{salesman='A', department='南方', amount=100}
{salesman='B', department='南方', amount=20}
```

## 总结
根据部门汇总和进行排序，然后每个部门也按照从大到小排序。这里使用到 `map` 的**键值对**属性。流程如下：
* 使用 `sumMap` 存储部门总数以及使用 listMap 存储部门信息。
* 对 `sumMap` 的 `value` 排序，把 map.entrySet 放在一个集合做排序。
*  根据排序后的 `sumMap` 的 `key` 找到 `listMap` 的 `value`。先对列表排序，最后放在集合中。
 
