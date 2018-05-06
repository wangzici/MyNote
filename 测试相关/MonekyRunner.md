# MonekyRunner

## 介绍

### 简介

提供了一系列api，可以完成模拟事件与截图操作

### api分类

①MonkeyRunner连接设备或模拟器

②MonkeyDevice操作设备或模拟器（如安装、卸载应用，发送模拟事件）

③MonkeyImage（完成图像对比与保存的操作）

### 执行脚本方式

```shell
#cmd中执行，monkeyrunner工具的位置在sdk/tools/monkeyruuner
monkeyrunner demo.py
```

##MonkeyRunner API

### alert

警告框

~~~python
void alert(string message,string title,string okTitle)
~~~

示例：

~~~python
#!/usr/bin/pyton
#-*- UTF-8 -*-

#这一行表示从com.android.monkeyrunner中导入MonkeyRunner模块
from com.android.monkeyrunner import MonkeyRunner
MonkeyRunner.alert('Hello mook friends','This is title','OK')
~~~

### waitForConnection

等待设备连接，有多个device id，则需要指明具体哪一个设备

~~~python
#timeout的时间单位是s
waitForConnection(float timeout,string deviceid)
~~~

## MonkeyDevice API

### drag

在页面进行拖动操作

~~~python
#start七点位置
#end终点位置
#duration手势持续的时间
#steps 插值点的步数，默认10
drag(tuple start,tuple end,float duration,integer steps)
~~~

### press

按键

~~~python
#keycode为Down、UP、DOWN_AND_UP
press(string keycode,dictionary type)
~~~

### startActivity

启动应用

~~~python
#注意只有一个参数
startActivity(package + '/' + activity)
~~~

### touch

点击

~~~python
#type为Down、UP、DOWN_AND_UP
touch(integer x,integer y,integer type)
~~~

### type

输入

~~~python
type(string message)
~~~

### takeSnapShot

截屏

~~~python
MonkeyImage takeSnapshot()
~~~

## MonkeyImage API

### sameAs

图像对比

~~~python
boolean sameAs(MonkeyImage other,float percent)
~~~

### writeToFile

保存图像文件

~~~python
#format:即指定如jpeg、png类型的图片
void writeToFile(String path,String format)
~~~

## 用例

### 实现在搜索框中输入查询词，并截图

~~~python
#!/usr/bin/pyton
#-*- UTF-8 -*-

#这一行表示从com.android.monkeyrunner中导入MonkeyRunner模块
from com.android.monkeyrunner import MonkeyRunner,MonkeyDevice,MonkeyImage
#连接设备，超时时间3秒
device = MonkeyRunner.waitForConnection(3,"192.168.56.101:5555")
#启动App
device.startActivity("com.exmaple.minibrowser2/com.example.minibrowser2.MainActivity")
#等待2秒
MonkeyRunner.sleep(2)
#点击搜索框，位置为(100,100)
device.touch(100,100,"DOWN_AND_UP")
MonkeyRunner.sleep(1)
#输入查询词
device.type('test')
MonkeyRunner.sleep(1)
#点击回车键
device.press('KEYCODE_ENTER','DOWN_AND_UP')
MonkeyRunner.sleep(1)
#点击搜索按钮
device.touch(400,100,"DOWN_AND_UP")
MonkeyRunner.sleep(6)
#截图
image = device.takeSnapshot()
image.writeToFile('./test.png','png')
#点击清除按钮
device.touch(300,100,"DOWN_AND_UP")
MonkeyRunner.sleep(3)

MonkeyRunner.alert('Hello mook friends','This is title','OK')
~~~

