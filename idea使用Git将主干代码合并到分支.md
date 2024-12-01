### 1、更新拉取分支dev代码

![](//upload-images.jianshu.io/upload_images/6366169-6735976b5c6e7975.png?imageMogr2/auto-orient/strip|imageView2/2/w/367/format/webp)

image.png

![](//upload-images.jianshu.io/upload_images/6366169-61a8a5414a66067a.png?imageMogr2/auto-orient/strip|imageView2/2/w/556/format/webp)

image.png

### 2、切换到主干分支，更新拉取主干master代码

![](//upload-images.jianshu.io/upload_images/6366169-1ed2c104436b0bcf.png?imageMogr2/auto-orient/strip|imageView2/2/w/308/format/webp)

image.png

### 3、因为是主干合并到分支，代码都更新后

#### 1、切换到dev分支（current）

#### 2、点击要合并的分支（from 分支）remote branch的master分支，出现2，3选项 。

![](//upload-images.jianshu.io/upload_images/6366169-c0de86c4b3e698bc.png?imageMogr2/auto-orient/strip|imageView2/2/w/640/format/webp)

image.png

#### 3、合并的时候，可以先选2，也可以选3，选2的话，将master分支上自己修改的代码合并到dev分支上

![](//upload-images.jianshu.io/upload_images/6366169-58ddef26a3ee8222.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

#### 选3直接合并的话如果有冲突要先解决冲突，合并完后，`本地dev最新合并后的代码要先在本地启动运行`，看是否报错，没有报错就进行第4步push到远程仓库

### 4、dev 分支运行没问题后，push合并后的代码到dev分支远程仓库

![](//upload-images.jianshu.io/upload_images/6366169-b7f89a4ff62f7e97.png?imageMogr2/auto-orient/strip|imageView2/2/w/1085/format/webp)

image.png

5、将分支代码合并到主干，反向操作分支就行。

本文转自 <https://www.jianshu.com/p/0fe1f83da33a>，如有侵权，请联系删除。