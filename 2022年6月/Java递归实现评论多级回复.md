> 最近工作需要做一个评论功能，除了展示评论之外，还需要展示评论回复，评论的回复的回复，这里就用到了**递归**实现评论的**多级回复**。

## 评论实体
数据库存储字段： `id` 评论id、`parent_id` 回复评论id、`message` 消息。其中如果评论不是回复评论，`parent_id` 为`-1`。

创建一个评论实体 `Comment`：
```
public class Comment {

    /**
     * id
     */
    private Integer id;

    /**
     * 父类id
     */
    private Integer parentId;

    /**
     * 消息
     */
    private String message;
}
```

查询到所有的评论数据。方便展示树形数据，对`Comment`添加回复列表

`List<ViewComment> children`

`ViewComment`结构如下：
```
// 展示树形数据
public class ViewComment {

    /**
     * id
     */
    private Integer id;

    /**
     * 父类id
     */
    private Integer parentId;

    /**
     * 消息
     */
    private String message;

    /**
     * 回复列表
     */
    private List<ViewComment> children = new ArrayList<>();
}
```

## 添加非回复评论
非回复评论的`parent_id`为`-1`，先找到非回复评论：
```
List<ViewComment> viewCommentList = new ArrayList<>();
// 添加模拟数据
Comment comment1 = new Comment(1,-1,"留言1");
Comment comment2 = new Comment(2,-1,"留言2");
Comment comment3 = new Comment(3,1,"留言3，回复留言1");
Comment comment4 = new Comment(4,1,"留言4，回复留言1");
Comment comment5 = new Comment(5,2,"留言5，回复留言2");
Comment comment6 = new Comment(6,3,"留言6，回复留言3");

//添加非回复评论
for (Comment comment : commentList) {
    if (comment.getParentId() == -1) {
        ViewComment viewComment = new ViewComment();
        BeanUtils.copyProperties(comment,viewComment);
        viewCommentList.add(viewComment);
    }
}
```

## 递归添加回复评论
遍历每条非回复评论，递归添加回复评论：
```
for(ViewComment viewComment : viewCommentList) {
    add(viewComment,commentList);
}


private void add(ViewComment rootViewComment, List<Comment> commentList) {
    for (Comment comment : commentList) {
        // 找到匹配的 parentId  
        if (rootViewComment.getId().equals(comment.getParentId())) {
            ViewComment viewComment = new ViewComment();
            BeanUtils.copyProperties(comment,viewComment);
            rootViewComment.getChildren().add(viewComment);
            //递归调用 
            add(viewComment,commentList);
        }
    }
}
```

* 遍历每条非回复评论。
* 非回复评论`id`匹配到评论的`parentId`,添加到该评论的`children`列表中。
* 递归调用。

## 结果展示：
![](https://files.mdnice.com/user/29864/e2bd84c2-5c3e-4e88-939e-1928800f1df3.png)

## github 源码
[https://github.com/jeremylai7/java-codes/tree/master/basis/src/main/java/recurve](https://github.com/jeremylai7/java-codes/tree/master/basis/src/main/java/recurve)
