# UIAutomator

## 简介

Android所提供的一套黑盒UI自动化测试框架

优点：

支持跨进程测试

不依赖于源码，基于黑盒进行

基于控件交互，支持Android源生控件解析，比坐标交互兼容性更强

事件等待优秀

缺点：

仅支持API Level 17及以上

不支持web测试

## API

### UiAutomatorTestCase

作为所有测试用例的超类，UiAutomatorTestCase继承于junit.framework.TestCase，遵循setUp、test、tearDown的测试流程，支持断言使用，负责基础的框架支持，包含执行过程的参数获取、实例获取及断言使用。

#### 参数获取

~~~java
//得到在执行UIAutomator时，通过-e <key> <value>传递的参数
Bundle getParams()
~~~

#### 实例获取

~~~java
//等价于UiDevice.getInstance()，取得当前设备的实例
UiDevice getUiDevice();
~~~

#### 流程执行

~~~java
//测试前环境准备，一般被用户覆写，在每个testCase前执行，将测试时的初始化准备放在该方法内执行
void setup();

//休眠指定时间
void sleep(long ms);

//测试后的收尾工作，一般被用户覆写，在每个testCase后执行，用于处理执行结果、保存数据、还原测试前环境
void tearDown()
~~~

#### 断言支持

~~~java
//如果断言失败，则认为case执行失败，case执行中断并进入tearDown
void assertXXX()
~~~

### UiDevice

UiDevice用来与测试设备进行交互，获取设备信息、发送操作指令及保存截图布局等状态

#### 事件操作相关

~~~Java
//单击Back键、Home键、Menu键等，这里仅以Back为例
boolean pressBack();

//向设备发送事件keyCode，可以直接实现上面那些方法的内容
boolean pressKeyCode(int keyCode)
~~~

#### 屏幕操作相关

向设备发送屏幕操作事件，包含点击、拖曳、修改设备屏幕状态（亮灭屏、屏幕方向）

~~~java
//单击屏幕坐标点(x,y)
boolean click(int x,int y);

//从start向end拖拽，步长为steps
boolean drag(int startX,int startY,int endX,int endY,int steps);

//swipe是没有目标的滑动
boolean swipe(int startX,int startY,int endX,int endY,int steps);

//按住不动顺序完成点间的滑动，步长为segmentSteps
boolean swipe(Point[] segments,int segmentSteps);

//设备灭屏，进入休眠状态
void sleep();

//唤醒设备，一般和isScreenOn来配合使用
void wakeUp();

//调整屏幕旋转90度
void setOrientationLeft();
void setOrientationRight();
//恢复自然角度
void setOrientationNatural();
~~~

#### 快捷开关相关

~~~java
//打开多任务切换界面
boolean pressRecentApps();

//打开通知栏
boolean openNotification();
~~~

#### 设备截图

~~~java
//截取当前屏幕截图，保存至指定文件
boolean takeScreenshot(File storePath);
//scale为缩放比例，quality为截图质量
boolean takeScreenshot(FIle storePath,float scale,int quality);
~~~

#### 属性获取

~~~java
//得到当前屏幕的高度与宽度
int getDisplayHeight();
int getDisplayWidth();

//获取当前Activity名
String getCurrentActivityName();

//获取当前应用包名
void getCurrentPackageName();

//判断当前屏幕是否是亮起状态
boolean isScreenOn();
~~~

#### 视图相关

~~~java
//获取当前窗口视图的布局
void dumpWIndowHierarchy(File dest);
void dumpWIndowHierarchy(String fileName);
void dumpWIndowHierarchy(OutputStream out);

//获取与清除上一次设置的text内容
String getLastTraversedText();
void clearLastTraversedText();
~~~

#### 事件等待

~~~java
//无限时等待当前应用空闲
void waitForIdle();
//等待当前应用空闲，若超时则不再等待
void waitForIdle(long timeout);

//等待指定报名的任意窗口更新，若超时则不再等待
boolean waitForWindowUpdate(String packageName,long timeout);
~~~

### UiSelecotr

UiSelector用来描述目标控件的特征，所有方法调用后返回的都是UiSelector，所以支持链式调用填充多个属性

~~~java
//查找控件类型为Button，且text为确定的控件
UiObject uiobject = mDevice.findObject(new UiSelector().text("确定").className(Button.class));
~~~

常用的一些属性有

~~~java
//指定目标控件的类型为type
className(Class<T> type);
className(className(String className));

//指定目标控件的文案为text
text(String text);

//指定目标控件的ID为id
resourceId(String id);

//目标控件是否可以滚动，注意如ListView只有一页的内容，那么也会为false
scrollable(boolean val)
~~~



### UiObject

UiObject抽象的程度比较高，所有Android基础控件都可以用UiObject来表示，在自动化过程中用以完成信息获取及控件交互。

#### 属性获取

~~~java
//判断控件是否存在
boolean exists();

//获取控件的完整边框信息
Rect getBounds();
//获取控件的可见边框信息
Rect getVisibleBounds();

boolean isFocusable();
boolean isSelected();
String getText();
~~~

#### 控件获取

~~~java
//获取子控件个数
int getChildCount();

//根据selector获取子控件
UiObject getChild(UiSelector selector);

//在父节点下根据selector获取对应控件
UiObject getFromParent(UiSelector selector);
~~~

#### 控件操作

~~~java
//点击控件中间、右下角、左上角
boolean click();
boolean clickBottomRight();
boolean clickTopLeft();

boolean longClick();
boolean longClickBottomRight();

//强烈建议使用的方法
//点击控件并且等待至有新窗口出现，如果超时时间内有新窗口出现则继续往下走，如果没有，则一直等到超时后继续往下走
boolean clickAndWaitForNewWindow(long timeout);

//从控件中间向下滑动
boolean swipeDown(int steps);

//设置控件文案
boolean setTExt(String text);
~~~

#### 事件等待（UiCollection）

### UiCollection

UiCollection继承于UiObject，用于表示符合同一UiSelector的控件集合，通常用于集合的遍历使用，较于UiObject多了四个方法，用于获取集合元素个数及指定元素。

### UiScrollable

UiScrollable继承于UiCollection，用来表示可滚动控件。通常用于屏幕之外的控件检索，封装了滚动操作及自动滚动查找控件的操作，在实际的自动化过程中还是比较实用的。

#### 状态相关

~~~java
//获取、设置检索最大滚动次数
int getMaxSearchSwipes();
void setMaxSearchSwipes(int swipes);

//设置水平、垂直列表
void setAsHorizontalList();
void setAsVerticalList();

//以默认速度向前、向后滑动
void scrollBackward();
void scrollForward();
//以默认速度滑动至开始、结束，最大的滑动次数为maxSwipes
boolean scrollToBeginning(int maxSwipes);
boolean scrollToEnd(int maxSwipes);

//滚动至描述为text、匹配selector、文案为text的控件，如果没有，则停留到列表尾部
boolean scrollDescriptionIntoView(String text);
boolean scrollIntoView(UiSelector selector);
boolean scrollTextIntoView(String text);
~~~

#### 控件获取

~~~java
//查找描述为text且匹配childPattern的控件，allowScrollSearch表示是否自动滚动查询，为false表示只查找当前页面
UiObject getChildByDescription(UiSelector childPattern,String text,boolean allowScrollSearch);

//查找文案为text的控件
UiObject getChildByText(UiSelector childPattern,String text,boolean allowScrollSearch);

//查找实例编号为instance且匹配childPattern的控件
UiObject getChildByInstance(UiSelector childPattern,int instance);
~~~

### Configurator

Configurator是UIAutomator的配置类，主要用于获取设置测试过程中的超时等待参数，通过修改参数可以调整操作速率，更加贴近想要模拟的用户操作场景。

下面就举几个典型的例子

~~~java
//设定通用动作的超时时间
Configurator setActionAcknowledgmentTimeout(long timeout);
//设定键盘输入的超时时间
Configurator setKeyInjectionDelay(long delay);
//设定滚动动作的超时时间
Configurator setScrollAcknowledgmentTimeout(long timeout);
~~~

### UiWatcher

UiWatcher是interface作为界面观察者处理测试过程所有的弹窗中断，接口仅存在一个方法即checkForCondition，实现后调用UiDevice进行注册，其生命周期是从注册的那一刻开始，直至测试结束或者调用UiDevice进行移除。如果用户想防止突然弹出的Crash弹框、闹钟、电话等阻塞测试的执行，UiWatcher还是非常实用的选择。

~~~java
//用于处理不可预料的弹框或者页面导致测试终端。如果回调中找到了匹配的情况并予以处理，则返回true；如果未有任何匹配，则返回false
boolean checkForCondition()
~~~

### 测试用例的编写

1、每个用例都继承于UiAutomatorTestCase，可见域为public。

2、setUp（）于每个测试用例前执行，适于用作环境准备。

3、tearDown（）于每个测试用例后执行，适用于数据收集及环境恢复。

4、一个类可以有多个测试用例，用例之间不建议耦合，执行期间为无序执行。



在AndroidStudio中使用

1、在对应module中添加依赖

~~~xml
//引入uiautomator
androidTestCompile 'com.android.support.test.uiautomator:uiautomator-v18:2.1.3'
~~~

2、获得应用中对应的控件id

由于用的是Android Studio的虚拟机，所以没有去用更方便的uiAutomatorViewer工具。这里直接使用的是在adb shell下调用。这里由于没有指定file的路径，所以默认会保存/sdcard/window_dump.xml下。这里就得到了当前ui的xml文件。

~~~java
130|generic_x86:/storage $ uiautomator dump
UI hierchary dumped to: /sdcard/window_dump.xml
~~~

通过adb pull命令从手机端传到电脑上

~~~java
F:\code\MyTest>adb pull /sdcard/window_dump.xml E:\BiZhi\
/sdcard/window_dump.xml: 1 file pulled. 0.1 MB/s (14540 bytes in 0.118s)
~~~

接着我们可以通过搜索text的方式，来得到对应控件的对应数据。这里截取了我们所需要的计算机的结果展示控件

~~~xml
<node 
      index="2" 
      text="677" 
      resource-id="com.android.calculator2:id/result" 
      class="android.widget.TextView" 
      package="com.android.calculator2" 
      content-desc="" 
      checkable="false" 
      checked="false" 
      clickable="false" 
      enabled="true" 
      focusable="false" 
      focused="false" 
      scrollable="false" 
      long-clickable="false" 
      password="false" 
      selected="false" 
      bounds="[0,463][1080,693]" />
</node>
~~~



3、新建一个module，可以不需要新建Activity，在module下的src.androidTest.java.*你的包名*下，新建一个Java类，如UiAutomatorTest，继承InstrumentationTestCase。（这里没有继承UiAutomatorTestCase的原因是：UiAutomatorTestCase 会提示已经过时）

~~~java
public class UiAutomatorTest extends InstrumentationTestCase {
    private UiDevice mDevice;

    @Override
    protected void setUp() throws Exception {
        super.setUp();
      //这里getName可以打印出调用的testCalculator
      //setUp of testCalculator
      	Log.i("uiAutomator", "setUp of " + getName());
        mDevice = UiDevice.getInstance(getInstrumentation());
    }

    public void testCalculator() throws UiObjectNotFoundException {
      //先打开计算器应用
        UiObject uiObject = mDevice.findObject(new UiSelector().text("Calculator"));
        uiObject.clickAndWaitForNewWindow();

        UiObject one = mDevice.findObject(new UiSelector().text("1").clickable(true));
        UiObject plus = mDevice.findObject(new UiSelector().text("+").clickable(true));
        UiObject equal = mDevice.findObject(new UiSelector().text("=").clickable(true));
        one.click();
        plus.click();
        one.click();
        equal.click();

        UiObject result = mDevice.findObject(new UiSelector().resourceId("com.android.calculator2:id/result"));
        String resultText = result.getText();
      //断言，如果与期望结果不同，则会报错
        assertEquals("测试计算的结果", "2", resultText);
    }

    @Override
    protected void tearDown() throws Exception {
      //tearDown类似finally的用法，无论用例执行中断与否，最后都会执行tearDown
        super.tearDown();
    }
}

~~~

如果期望结果改成"3"，则会报出如下错误

~~~java
junit.framework.ComparisonFailure: 测试计算的结果 expected:<[3]> but was:<[2]>
at junit.framework.Assert.assertEquals(Assert.java:85)
at wzt.uiautomatordemo.UiAutomatorTest.testCalculator(UiAutomatorTest.java:38)
at java.lang.reflect.Method.invoke(Native Method)
~~~



### 启动方式

可以直接在AndroidStudio中启动

或者编译成jar包，通过adb shell去启动，启动命令如下：

~~~java
//adb shell uiautomator runtest <jar> -c <test_class_or_method> [options]
//[options]用于指定测试过程中的特殊参数，比如后台挂起测试--nohup、向测试过程传递参数-e <key> <value>，传递的参数可以在UiAutomatorTestCase中使用getParams()获得，可以多项指定，还有是否处于debug模式、以及dump当前页面的xml至文件中等功能
//即运行A.jar中的com.tencent.uiAutotest.C类的testC_e测试用例，如果不指定用例名，则默认执行全部测试用例
adb shell uiautomator runtest A.jar –c com.tencent.uiAutotest.C#testC_e
~~~

### 技巧

巧用Runtime，通过指令快速打开对应的页面

巧妙使用settings数据库来获取系统属性