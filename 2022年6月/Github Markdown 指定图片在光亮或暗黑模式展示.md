# Github Markdown 指定图片在光亮或暗黑模式展示

`Github` 根据系统配置不同的主题模式：

![file](http://image.openwrite.cn/27106_08BE1B6C3F8B4D26914B4D185D9AAED0)

如果想要在光亮模式和暗黑模式显示不同的主题的图片，比如以下就是同一个图片在暗黑模式和光亮模式下展示:

![光亮模式](http://image.openwrite.cn/27106_FE1D5A57545745358EE1B58EAAF3244E)
![暗黑模式](http://image.openwrite.cn/27106_9C23E03AB93B4F6596400B086546E0D1)


## 解决方案

在markdon 的图片链接后添加`#gh-dark-mode-only` 或者 `#gh-light-mode-only` 参数。

暗黑模式添加参数 
`#gh-dark-mode-only`

光亮模式添加参数
`#gh-light-mode-only`

比如：

```
![](./profile-3d-contrib/profile-green.svg#gh-light-mode-only)
![](./profile-3d-contrib/profile-night-green.svg#gh-dark-mode-only)
```

引用好配置之后，markdown 就可以根据不同的主题展示对应的图片了。

## 总结

* 暗黑模式 `![](./profile-3d-contrib/profile-green.svg#gh-light-mode-only)`
* 光亮模式 `![](./profile-3d-contrib/profile-night-green.svg#gh-dark-mode-only)`


## 参考

[Github doc Specifying the theme an image is shown to](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#specifying-the-theme-an-image-is-shown-to)
