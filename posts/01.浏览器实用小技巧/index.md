# 浏览器实用小技巧

## 下载网页中的内嵌 PDF
在一些网页中，PDF文件是以内嵌形式呈现的，直接下载链接可能隐藏在网络请求中。以下是找到并下载这些内嵌PDF文件的方法：
1. 打开开发者工具（Fn&#43;F12）
2. 选中Network栏目后再选择XHR，这会筛选出所有XMLHttpRequest，通常内嵌PDF的请求会出现在这里。
3. Ctrl&#43;R（刷新）重新加载所有网络请求
4. 在 Network 面板中，找到PDF文件的请求。通常，这些请求会有一个文件名以 `.pdf` 结尾。
5. 右键点击该请求，然后选择 Open in new tab。这样会在新的标签页中打开PDF文件，您可以直接获取下载链接并保存文件。

## 解决网站复制问题
不能复制的根本原因是网站使用了JavaScript代码来阻止用户选择和复制内容。我们只需要禁用JavaScript就可以。
1. 打开开发者工具（Fn&#43;F12）	
2. Settings-&gt;Preferences
3. 划到最后有个`Disable JavaScript`选项



---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://lruihao.cn/posts/01.%E6%B5%8F%E8%A7%88%E5%99%A8%E5%AE%9E%E7%94%A8%E5%B0%8F%E6%8A%80%E5%B7%A7/  

