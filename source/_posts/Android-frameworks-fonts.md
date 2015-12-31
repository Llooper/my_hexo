title: Android-framesworks-fonts
comments: true
date: 2015-12-31 10:13:42
tags:
	- Android
	- AOSP
	- frameworks

categories: AOSP
original: false
updated:
---

### 一、概述
近期在对Android系统源码进行一些个性化的修改，主要集中在`frameworks`层。本篇旨在将自添加或者已存在的字体包设置为系统默认的字体，满足产品的需求。   
这次定制的系统版本为Android4.4，主要为4.4添加宋体验证修改效果，不喜宋体的略过。网络查找的资料提及4.X与5.X版本的系统在设置默认字体方面有一些区别，望想定制5.X及以上系统的朋友自行补充。

<!--more-->

### 二、实现

#### 1.分析
Android4.4版本系统字体配置文件主要有三个：
><p>/system/etc/system_fonts.xml</p>
><p>/system/etc/fallback_fonts.xml</p>
><p>/vendor/etc/fallback_fonts.xml</p>   

第一个文件用于定义系统默认字体，文本中的第一个字体系列就是系统默认的英文字体，后面的字体是某些语言或者某些应用下是使用。   
第二个文件是用来定义系统默认字体找不到的字符的扩展字体库，按照从上到下的顺序进行索引。   
最后到了一个比较大的文件，DroidSansFallback.ttf，来寻找中日韩以及其他特殊字符。   
第三个文件为第三方定制产商添加字体的配置文件，Google为我们良好定制预留了接口。

#### 2.实战
Android源码字体目录：
<b>`/frameworks/base/data/fonts/`</b>   
针对上述分析，得出将自定义字体设置为Android系统默认的可修改方案：

1)暴力方式：将添加字体名字修改为DroidSansFallback.ttf，放置到系统源码`fonts`目录，替换掉原生DroidSansFallback.ttf。这种方式比较暴力，不建议采用，因为有可能导致某些字显示不正常或者不出现。   

2）修改`fonts`目录下的 `system_fonts.xml`文件。既然在4.x版本下，system_fonts.xml是系统默认字体的配置文件，且字体调用顺序从上到下，将自添加的字体包添加到该文件的第一的位置即为第一默认调用。 

```xml  
<family>  
	<fileset>  
		<file>MyFont.ttf</file>  
	</fileset>  
</family>  
```

同时还需将添加的字体包copy到生成的系统的/system/fonts/目录：修改`fonts`目录下的fonts.mk文件

```makefile
	PRODUCT_COPY_FILES := \  
		...
		frameworks/base/data/fonts/MyFont.ttf:system/fonts/MyFont.ttf 
```

3)遵循Google预留接口进行自定制，对`vendor_fonts.xml`进行修改。对于厂商定制ROM备用字体文件，android官方指导规范的方法是，修改`/frameworks/base/data/fonts/vendor_fonts.xml`文件，并在`fonts.mk`文件中添加代码段，使此文件在构建过程中拷贝并重命名为`/vendor/etc/fallback_fonts.xml`文件。  
								 
`vendor_fonts.xml`修改:

```makefile
<familyset>  
	<family order="0">  
		<fileset>  
			<file>MyFont.ttf</file>  
		</fileset>  
	</family>   
</familyset> 
```
其中family order="0"表示vendor字体插入`system_fonts.xml`的第一个位置。   
`fonts.mk`添加的代码段：

```makefile
PRODUCT_COPY_FILES := \  
	...
	frameworks/base/data/fonts/vendor_fonts.xml:$(TARGET_COPY_OUT_VENDOR)/etc/fallback_fonts.xml \
	frameworks/base/data/fonts/MyFont.ttf:system/fonts/MyFont.ttf

```
---

####补充
`system_fonts.xml`配置了系统的内置字体，第一个family节点为系统默认字体。nameset节点的各个name子节点定义可用的字体名称，fileset节点的file子节点分别对应normal、bold、italic、bold-italic四种字体样式，如果file节点个数少于四个，相应字体样式会对应已有兄弟file节点的字体文件。   
`fallback_fonts.xml`配置了系统备用字体，只有在系统内置字体中找不到相应字符时，才会到备用字体中去寻找，family节点的顺序对应搜索顺序，搜索匹配规则采用BCP47的定义。   
`vendor/etc/fallback_fonts.xml`是为了规范厂商定制默认字体，加载备用字体配置时，会将此文件中定义的各个family插入到`system/etc/fallback_fonts.xml`中，插入位置由family节点order属性指定，如果没有order属性，默认会插入到最后。
*SourceHanSansCN 思源字体已经加入系统，默认是Regular，如果设计要求使用 Medium或者Light的，直接调用下面的API就可以了
`Typeface.createFromFile("/system/fonts/xxx.ttf");`*

####效果展示
修改前：   
![P image](http://7xp9uy.com1.z0.glb.clouddn.com/FC2DE68F-9822-428F-84C2-658CE675F0B5.jpg)

修改后：   
![L image](http://7xp9uy.com1.z0.glb.clouddn.com/6B9083EB-EBA9-4328-BD42-7F66E1ADDAFC.png)
