# Robotium

## 介绍

### 简介

优点：

支持黑盒与白盒测试

支持Native应用与hybird应用

测试代码运行在被测应用所在的线程，控件识别与模拟UI都可以快速执行

缺点：

由于是基于Instrumentation的事件发送，因此**无法跨应用**

代码运行在被测进程，会导致进程的内存、CPU占用测量会有误差

### 创建方法

1、添加robotium包的依赖

~~~xml
androidTestImplementation 'com.jayway.android.robotium:robotium-solo:5.6.3'
~~~

2、创建测试类

在app->src->androidTest->java->对应的包下，新建一个测试类如MyFirstTest，继承父类ActivityInstrumentationTestCase2，同时实现构造方法，重写setUp方法与tearDown方法，构造方法传入需要启动的activity的class对象，setUp方法初始化Solo对象，tearDown方法释放Solo的资源

~~~java
public class MyFirstTest extends ActivityInstrumentationTestCase2 {
    private static final String LAUNCHER_ACTIVITY_FULL_CLASSNAME = "wzt.mytest.MainActivity";
    private static Class launcherActivityClass;
    private Solo solo;

    static {
        try {
            launcherActivityClass = Class.forName(LAUNCHER_ACTIVITY_FULL_CLASSNAME);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }

    public MyFirstTest() {
        super(launcherActivityClass);
    }

    @Override
    protected void setUp() throws Exception {
        super.setUp();
        solo = new Solo(getInstrumentation(), getActivity());
    }

    @Override
    protected void tearDown() throws Exception {
        solo.finishOpenedActivities();
      //注意此处super方法在后调用
        super.tearDown();
    }

  //测试方法以Test开头
    public void testEditText() throws Exception {
        solo.sleep(1000);
        EditText editText = solo.getEditText(0);
        solo.enterText(editText, "this is my test");
        solo.sleep(5000);
    }
}

~~~



### 执行脚本方式

直接在Android Studio中运行

## 基本操作

### 控件

如果没有查找到对应的控件，则会抛出Throwable异常，我们可以catch到

#### Native控件

##### 获取

1、通过String型ID获取控件

~~~java
ImageView mIcon = (ImageView) solo.getView("mypic");
RelativeLayout rel = (RelativeLayout) solo.getView("example1");
LinearLayout lin = (LinearLayout) solo.getView("example2");
~~~

2、通过索引获取控件

~~~java
//返回界面中第一个类型为Button的控件，其它还有getEditText（int index）、getText（int index）等
Button loginBtn = (Button) solo.getButton(0);
~~~

3、通过文本获取控件

~~~java
//返回界面中文本为‘登录’类型为Button的控件，其它还有getText（String text）、getEditText（String text）
Button loginBtn = (Button) solo.getButton("登录");
~~~

4、根据控件类型进行过滤

~~~java
//获取当前界面或弹框中所有控件类型为TextView的控件
ArrayList<TextView> allTextViews = solo.getCurrentViews(TextView.class);

//获取指定父控件下所有控件类型为TextView的控件
RelativeLayout rel = (RelativeLayout) solo.getView("example1");
ArrayList<TextView> allTextViews = solo.getCurrentViews(TextView.class, rel);
~~~

**获取控件常见问题：**

1、把string的id转化成int型的id

~~~java
//此方法来自于源码
public int getId(String id,String packageName){
  Context targetContext = instrumentation.getTargetContext().getApplicationContext();
  int viewId = targetContext.getResources().getIdentifier(id, "id", packageName);
  LogUtils.logD("CopyOfAssistantTabActivityTest", "viewId:" + viewId);
  if(viewId == 0){
  	viewId = targetContext.getResources().getIdentifier(id, "id", "android");
  }
  return viewId;
}
~~~

2、遍历点击ListView

对于ListView而言，通过getFirstVisiblePosition（）和getLastVisiblePosition（）可以获取ListView在屏幕中第一个可见子控件及最后一个可见子控件在列表中的位置。当遍历至当前最后一个子控件时，通过solo.scrollListToLine（listView，lastPosition）方法将列表滑至lastPosition所在的位置，即实现翻屏的效果。当遍历至每个child子控件时，可以通过该子控件的布局结构来判断该子控件是否为要查找的控件。另外，需要注意的是，正如前文所介绍的，scrollListToLine（listView，lastPosition）方法并不会直接产生上滑手势，因此如果列表需要产生上滑动作才能加载更多的话，则还需要配合使用drag方法进行上拉加载更多。

~~~java
public RelativeLayout findCardByType(int maxCount) {
  // 获取当前界面中的ListView
  ListView listView = getCurrentListView();
  int firstPosition = 0;
  int lastPosition = 0;
  RelativeLayout relativeLayout = null;
  int currentPosition = 1;
  labelAll:
  for (int i = 0; i < length; i++) {
    firstPosition = listView.getFirstVisiblePosition();
    lastPosition = listView.getLastVisiblePosition();
    for (int j = 1; j <= lastPosition - firstPosition; j++) {
      currentPosition++;
      if (currentPosition >= maxCount) {
        break labelAll;
      }
      // 判断该节点是否为relativeLayout
      if (listView.getChildAt(j) instanceof RelativeLayout) {
        relativeLayout = (RelativeLayout) listView.getChildAt(j);
        // 这里可以对该relativeLayount进行判断，例如获取该relativeLayout中的子控件，如果有标题则判断标题等
        if (isSatisfied(relativeLayout)) {
            break labelAll;
        }
        relativeLayout = null;
      }
    }
    solo.scrollListToLine(listView, lastPosition);
    if (lastPosition >= listView.getCount()) {
      // 当需要上拉加载更多时，调用drag实现的方法进行上拉加载更多
      dragUpToShowAll(listView);
    }
    sleeper.sleep();
  }
  sleeper.sleep();
  return relativeLayout;
}
~~~



##### 操作

1、点击与长按

~~~java
//直接点击控件,Robotium还提供了点击文本、点击图片的API，例如clickOnText（String text）、click-OnButton（String text）
Button loginBtn = (Button) solo.getView("loginBtn");
solo.clickOnView(loginBtn);
//长按
solo.clickLongOnView(loginBtn)；
~~~

2、操作输入框

测试框架中主要提供了enterText（EditTexteditText，String text）和typeText（EditTexteditText，String text）两种方法，前者直接对EditText文本框进行赋值，不会有文本输入的展示过程，而后者则会一个文本一个文本地输入，更贴近真实用户的操作。

~~~java
EditText userET = (EditText) solo.getView("example_et_id");
solo.enterText(userET, "my_user_name") //直接对文本框赋值
solo.typeText(userET, "my_user_name") //会展示输入的过程
~~~

3、滑动、滚动

在滑动方面，测试框架主要提供了两类支持，一类是根据坐标进行滑动从而可以模拟各类手势操作，另一类则是根据控件来直接进行滚动操作。注意第二类只是调用控件自身的API，因此不会触发相应的监听器

~~~java
//第一类：根据坐标进行滑动，这里的意思是从(100,100)移动到(500,500)，第五个参数为步长,意思是把这次移动平均分成步长的个数，然后会依次滑动。
solo.drag(100,500,100,500,1);

//第二类：直接根据控件进行滚动
//滚动至顶部/底部
solo.scrollToTop();
solo.scrollToDown();
//向下滚动一屏/向上滚动一屏
solo.scrollUp();
solo.scrollDown();
//滚动列表absListView至Line行
scrollListToLine(AbsListView absListView,int line)
~~~

4、搜索与等待

~~~java
//休眠指定的时间，单位毫秒
solo.sleep(int time);
//从当前界面搜索指定文本
boolean hasText = solo.searchText(String text);
~~~

Robotium还提供了Condition模式即waitForCondition（Condition condition，int timeout）方法，使用该方法时，实现Condition接口并重写isSatisfied（）方法，isSatisfied（）为true时将跳出等待。通过这种模式我们可以自定义实现更多类型的等待操作

~~~java
public void waitForAppInstalled(final String appName, int timeout) {
  waitForCondition(new Condition() {
    @Override
    public boolean isSatisfied() {
      //此处的sleeper其实就是solo属性里的一个Sleeper对象，sleepMini其实就是调用Thread.sleep(500)，使当前测试的线程暂停500ms
    sleeper.sleepMini();
    return checker.isAppInstalled(appName);
    }
  }, timeout);
}
~~~

5、截图及其它

~~~java
//截图，图片名称为指定的name参数，图片默认路径为/sdcard/Robotium-Screenshots
takeScreenshot(String name)
~~~



​	除了常规的操作外，Robotium测试框架还提供了发送模拟按键sendKey（int key）、设置屏幕是横屏还是竖屏setActivityOrientation（int orientation）、模拟点击返回键goBack（）、跳转至指定Activity的方法goBackToActivity（String name）、收起输入法hideSoftKeyboard（）、关闭所有已打开的Activity的方法finishOpenedActivities（）等。通过组合利用这些常用操作，基本就可以完成在Android端的UI自动化操作了。

## 断言

### JUnit中的断言

~~~java
//断言传入的condition参数应该为True，否则将抛出一个带有message提示的Throwable异常
Button loginBtn = (Button) solo.getView("loginBtn");
assertTrue("‘登录’按钮应该要显示在界面", loginBtn.isShown());

//直接使用例失败，并抛出一个带有message提示的Throwable异常
if(isBadHappened()){
	fail("this should no happened");
}
~~~

而如果出现某种场景，我们希望直接使用例通过而不再执行，则此时在用例脚本中直接使用return即可。

### Robotium中的断言

~~~java
//获取当前的Activity名
String currentActivity = solo.getCurrentActivity().getClass().getSimpleName();
//判断当前的界面是否是期望的expectedActivity，如果不是，以第一个参数为message抛出异常
solo.assertCurrentActivity("expected xxxActivity" + " but was " + currentActivity, expectedActivity);
~~~

~~~java
//判断当前是否处于内存紧张的状态
assertMemoryNotLow（）
~~~

### Android中的断言

基本原理其实就是使用Android提供的SDK来获取相关的信息，从而进行断言的操作，如下面的代码，就是根据Android的系统api来实现判断某应用是否是系统应用

~~~java
/**
* 根据
packageName判断该应用是否是系统应用
* @param packageName 应用的包名
* @return true，系统应用；false，非系统应用
*/
public boolean isSystemApp(String packageName){
  PackageManager pm = getInstrumentation().getTargetContext().getApplicationContext().getPackageManager();
  ApplicationInfo applicationInfo = null;
  try {
    applicationInfo = pm.getApplicationInfo(packageName, PackageManager.GET_UNINSTALLED_PACKAGES);
    if(applicationInfo !=null && (applicationInfo.flags & ApplicationInfo.FLAG_SYSTEM) ==1){
    LogUtils.logD(TAG, "applicationInfo flag:" + (applicationInfo.flags & ApplicationInfo.FLAG_SYSTEM));
    return true;
  	}
  } catch (NameNotFoundException e) {
  }
  return false;
}
~~~

