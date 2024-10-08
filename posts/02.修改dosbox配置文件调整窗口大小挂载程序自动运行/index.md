# 修改DOSBox配置文件（调整窗口大小，挂载程序自动运行）

## 打开DOSBox配置文件

DOSBox的配置文件名为`DOSBOX 版本号  Options`，对于有的版本配置文件好像名为`bc31.conf`。在DOSBOX的安装目录下，后缀为`.bat`。直接用记事本打开进行修改即可。
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZHJvaWRzYW5zZmFsbGJhY2s%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_7%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png)

## 调整窗口大小

在实际使用DOSBox的时候，通常会觉得DOSBox的窗口有点小：
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZHJvaWRzYW5zZmFsbGJhY2s%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png)

我们又不能直接调节。这个时候有两种方法调节：

* `Alt&#43;Enter`进入全屏模式。
	这种方法比较别扭，不过方便快捷。
	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZHJvaWRzYW5zZmFsbGJhY2s%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_15%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png)

* 利用配置文件修改。
	进入配置文件后定位到：
	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZHJvaWRzYW5zZmFsbGJhY2s%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_18%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png)
	第一个参数`fullscreen`代表是否填充屏幕，其是否进入全屏模式。如果你喜欢第一种方式，可以直接在这里修改，这样每次进入都是全屏模式了。
	第二个参数和第三个参数都是调节全屏模式下的特征，这里我们不再作介绍了。
	关键在于`windowresolution`和`output`，分别为窗口分辨率和输出。
	我们将分辨率调为`1280x1080`（分辨率可以根据自己实际需要进行调节）,同时需要将输出改为`opengl`,即开启渲染。
	保存重启DOSBox如下：
	![加粗样式](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZHJvaWRzYW5zZmFsbGJhY2s%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20240709224159947.png)

## 挂载程序自动运行

当我们没有打开DOSBox的时候进入的是Z盘，即DOSBox的虚拟Z盘。
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZHJvaWRzYW5zZmFsbGJhY2s%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20240709224200005.png)

但是，我们需要使用本地的`masm`文件夹用来执行汇编程序和使用debug等操作。这个时候我们通常的方法是可以将`masm`文件夹挂载为DOSBox的C盘，即输入`mount c d:\masm`即可将`d:\masm`挂载到C盘。通俗的理解，该文件夹之后就是DOSBox的C盘了。我们用dos命令`c:`进入C盘即可使用`masm`文件夹下的内容了。
所以以上操作为：

```c&#43;&#43;
mount c d:\masm
c:
```

那么如果我们每次打开都要进行这样的操作，那未免太累了。所以我们可将两条语句添加到配置文件的文件末尾处。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d8c0f9f2e77f409a9defad07b0f8353c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAdW5pcXVlX3B1cnN1aXQ=,size_20,color_FFFFFF,t_70,g_se,x_16)
保存重新打开DOSBox之后为：

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZHJvaWRzYW5zZmFsbGJhY2s%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20240709224200078.png)
当然，如果你想挂载其他的程序照此操作即可实现。



---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/02.%E4%BF%AE%E6%94%B9dosbox%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%B0%83%E6%95%B4%E7%AA%97%E5%8F%A3%E5%A4%A7%E5%B0%8F%E6%8C%82%E8%BD%BD%E7%A8%8B%E5%BA%8F%E8%87%AA%E5%8A%A8%E8%BF%90%E8%A1%8C/  

