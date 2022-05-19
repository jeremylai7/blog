# 深入理解Redis 数据结构—简单动态字符串sds

Redis是用ANSI C语言编写的，它是一个高性能的key-value数据库，它可以作用在数据库、缓存和消息中间件。其中 Redis 键值对中的键都是 string 类型，而键值对中的值也是有 string 类型，在 Redis 中 string 类型运用还是很广泛的。本文主要介绍 string 的数据结构—— **简单动态字符串（Simple Dynamic String）** 简称sds。

## sds 实现
sds 的数据结构:
```
struct sdshdr {

     //buf 已占用的长度
     int len;

     // buf 剩余的可用的长度
     int free;
   
     // 保存字符串数据的地方
     char buf[]; 
}
```
结构 sdshdr 保存了 len、free 和 buf 三个属性，分别记录字符的已使用的长度，未使用的长度，以及实际保存字符串的数组。
以下是一个新建的，保存 hello world 字符串的 sdshdr 结构：
```
struct sdshdr {
    len = 5;
    free = 0;
    buf = "hello\0"; 
}
```
* free 属性值为0，表示这个sds没有分配未使用的空间。
* len 属性值为5，表示这个sds保存了一个五字节长的字符串。
* buf 属性是一个 char 类型的数组，数组的前五个字节分别保存了 'h'、'e'、'l'、'l'、'o' 五个字符，而最后一个字节保存了空字符'\0'。

sds 遵守 C 字符串以空字符串结尾的惯例，保存的空字符串一个字节空间不计算在 sds 的 len 属性里面。添加空字符串到字符串末尾等操作，都是由 sds 函数自动完成的，所以这个空字符对于使用者来说完全是透明的。

>通过 len 属性，可以实现时间复杂度 O(1) 的长度计算。另外通过对 buf 分配一些额外的空间，并使用 free 记录未使用空间的长度，sdshdr 可以减少内存的重新分配。这是 sds 相对 c 字符串的一个优势。

## 为何 Redis 不用 C 语言表示字符串
Redis 是使用 C 语言开发的，而在使用最多的字符串上，Redis 没有使用 C 语言传统的字符串表示，而且使用自己构建的**简单动态字符串（sds）**。
在 C 语言中，字符串可以用一个 \0 结尾的 char 数组表示。比如 hello world 在 C 语言中就可以表示为"hello world\0"。数组一般初始化以后长度就已经固定了，不能支持字符串**追加append**和**长度计算**操作：
* 每次计算字符串长度都要遍历一遍数组，所以时间复杂度是O(N)
* 对字符串每次进行追加操作，需要对字符串进行一次**内存分配**

## sds 优化追加字符操作
Redis 作为数据库，对于查询速度要求严格，数据修改也比较频繁，如果每次修改字符串都需要执行一次内存分配的话，都会占用大量的时间。所以 Redis 选择了 sds 而不是 C 字符串，sds 可以减少追加字符的内存分配。通过举例来说明，执行以下操作时，sds 内部的变化：
```
redis> set msg "hello world"
OK

redis> append msg " again"
(integer)18

redis> get msg
"hello world again"
```
首先 set 命令创建并保存hello world 到一个 sdshdr 中，这个 sdshdr 的值如下：
```
struct sdshdr {
     len = 11;
     free = 0;
     buf = "hello world\0";
}
```

当执行 append 命令时，相对应的 sdshdr 被更新，字符串 " again" 会被追加到原来的 "hello world" 之后：
```
struct sdshdr {
     len = 17;
     free = 17;
     buf = "hello world again\0                ";
}
```

当调用 set 命令创建 sdshdr 时，Redis 没有给 sdshdr 分配多余的空间，free 属性为0。而在执行 append 操作之后，Redis 为 buf 分配了多于所需空间一倍的大小。

在执行 append 命令之后，保存 "hello world again" 共需要17 + 1 个字节，但是程序为 sdshdr 分配了 17 + 17 + 1 = 35 个字节，而后续如果在对 sdshdr 进行追加操作，只要追加的长度不超过 free 属性值，那么就不需要对 buf 进行内存重分配。

比如执行以后命令并不会引起 buf 的内存重分配，因为新追加的字符串长度小于17：
```
redis> append msg  " again"
(integer) 23
```
对应的 sdshdr 结构如下：
```
struct sdshdr {
     len = 23;
     free = 11;
     buf = "hello world again again\0               ";
}
```

redis 内存分配可以查看[源码](https://github.com/redis/redis/blob/2.6/src/sds.c) sds.s/sdsMakeRoomFor，sdsMakeRoomFor 函数描述了内存分配的策略，下面的该函数的伪代码：
```
// sdshdr:追加前的字符
// addlen：追加字符串
sds sdsMakeRoomFor(sdshdr, addlen) {
   
    // 多余空间大于追加空间，无序再分配内存，直接返回
    if (free >= addlen) return s;
    // 计算新字符的长度
    newlen = (len+addlen); 
   // 如果新字符的长度小于 SDS_MAX_PREALLOC，就分配两倍新字符空间
  //  如果新字符的长度大于 SDS_MAX_PREALLOC，就分配新字符空间 + SDS_MAX_PREALLOC 空间
    if (newlen < SDS_MAX_PREALLOC)
        newlen *= 2;
    else
        newlen += SDS_MAX_PREALLOC;
    // 分配内存
    newsh = zrealloc(sh, sizeof(struct sdshdr)+newlen+1);
    // 更新 free 属性
    newsh.free = newlen - len;
    return newsh;
}
```
> 而对于字符的缩短操作，Redis 保存缩短后的字符串，此时并不会进行内存重分配，而是使用 free 属性记录缩短的字符长度。

## 总结
Redis 的 string 类型为何使用sds而不是 C 字符串，因为sds有两点优势：
* 计算字符长度，C 字符串复杂度O（n），而 sds 复杂度为 O（1）
* 字符追加操作，C 字符串每次都需要对内存进行重分配，而 sds 每次会进行动态扩容，当添加字符小于空闲字符时，不会对内容进行分配，减少系统等待时间

## 参考
[Redis 设计与实现](https://book.douban.com/subject/25900156/)

