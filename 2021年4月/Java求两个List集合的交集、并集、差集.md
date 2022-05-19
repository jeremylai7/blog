# Java求两个List集合的交集、并集、差集

在项目中经常会求解集合的交集、并集、差集，这里做个记录。首先创建两个集合list1、list2以及添加元素。
```
List<String> list1 = new ArrayList<>();
list1.add("a");
list1.add("b");
List<String> list2 = new ArrayList<>();
list2.add("b");
list2.add("c");
```
#### 交集
```
list1.retainAll(list2);
```
#### 并集（去重）
```
list1.removeAll(list2);
list1.addAll(list2);
```
#### 并集（不去重）
```
list1.addAll(list2);
```
#### 差集 list1有的，list2没有
```
list1.removeAll(list2);
```
