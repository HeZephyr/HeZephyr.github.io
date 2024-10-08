# 解决Linux系统centos7的开机报错：Welcome to Emergency Mode

开机后报错如下：
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/031ebd8bcd3c4a969d0a383834b72cb6.png)
报这个错误多数情况下是因为/etc/fstab文件的错误。注意一下是不是加载了外部硬盘、存储器或者是网络共享空间，在重启时没有加载上导致的。
我就是因为在上次编辑了/etc/fstab文件想实现自动挂载，但重启并没有挂载成功导致的。

所以我们需要恢复/etc/fstab文件，处理方法如下：

1. 先输入密码登录root账户；
2. 输入`vim /etc/fstab`编辑，注释或者修改自己增加的内容；
	![img](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_19%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20240709224518670.png)
	这里我挂载defaults写成default了，所以会出错，所以我这里直接更改后就没有问题了。

3. 保存并退出；
4. 输入`reboot`重启即可恢复正常。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/03.%E8%A7%A3%E5%86%B3linux%E7%B3%BB%E7%BB%9Fcentos7%E7%9A%84%E5%BC%80%E6%9C%BA%E6%8A%A5%E9%94%99welcome-to-emergency-mode/  

