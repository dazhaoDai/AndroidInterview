[![](https://badge.juejin.im/entry/598d4d526fb9a03c467c0bdc/likes.svg?style=flat-square)](https://juejin.im/entry/598d4d526fb9a03c467c0bdc/detail)
### 关于Android触摸事件机制
Android触摸事件机制，开发中都是老生常谈，但是惭愧的是，这么久开发，依然对Android触摸事件的具体流程，懵懵懂懂，趁着项目上线间隙，来重新研究一下。

### 准备
为了尽可能简单并清晰的展示Android触摸事件的 分发--拦截--消费过程，将根据Activity、ViewGroup以及View的特点，做了一点准备工作。

**触摸事件**:
- actionDown
- actionMove
- actionUp

**对于Android事件处理分为两类**：

**对于Activity和View**： 只有两种事件：
- 分发: dispatchTouchEvent函数
- 消费: onTouchEvent函数和OnTouchListener函数

**对于ViewGroup**： 全部三种事件：
- 分发: dispatchTouchEvent函数
- 拦截：onInterceptTouchEvent函数
- 消费: onTouchEvent函数和OnTouchListener函数

所以，重写一个典型的ViewGroup：RelativeLayout和一个典型的View：TextView，针对他们的触摸事件进行改写，来得出触摸事件具体的流程。

TouchRelativeLayout
```
public class TouchRelativeLayout extends RelativeLayout {
    public TouchRelativeLayout(Context context) {
        super(context);
    }

    public TouchRelativeLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public TouchRelativeLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        LogUtils.e("ViewGroup-------dispatchTouchEvent-------", ev.getAction() + "");
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        LogUtils.e("ViewGroup-------onInterceptTouchEvent-------", ev.getAction() + "");
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        LogUtils.e("ViewGroup-------onTouchEvent-------", event.getAction() + "");
        return super.onTouchEvent(event);
    }
}
```
TouchTextView
```
public class TouchTextView extends TextView {
    public TouchTextView(Context context) {
        super(context);
    }

    public TouchTextView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    public TouchTextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        LogUtils.e("View---------dispatchTouchEvent-------", ev.getAction() + "");
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        LogUtils.e("View---------onTouchEvent-------", event.getAction() + "");
        return super.dispatchTouchEvent(event);
    }
}
```

代码里不进行任何操作，只打印出当前控件的触摸事件。
继续新建一个Activity，为触摸事件提供入口：

Activity的布局
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    android:orientation="vertical">

    <com.ddz.lifestyle.baseview.customview.TouchRelativeLayout
        android:id="@+id/rl_touch"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_centerInParent="true"
        android:background="@android:color/holo_blue_light">

        <com.ddz.lifestyle.baseview.customview.TouchTextView
            android:id="@+id/tv_touch"
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:layout_centerInParent="true"
            android:background="@color/recy_bg" />
    </com.ddz.lifestyle.baseview.customview.TouchRelativeLayout>
</RelativeLayout>
```
忽略外层的RelativeLayout，实际就是一个Activity中，一个ViewGroup包裹一个View，Activity中不进行操作，只打印出触摸事件:
```
   @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        LogUtils.e("activity-------dispatchTouchEvent-------", ev.getAction() + "");
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        LogUtils.e("activity-------onTouchEvent-------", event.getAction() + "");
        return super.onTouchEvent(event);
    }
    
 ```
 运行程序，打开要触摸的Activity，显示效果，蓝色区域为重写的TouchRelativeLayout，其中的灰色区域就是重写的TouchTextView
 
 ![image](http://upload-images.jianshu.io/upload_images/2789715-adbcbe220fe66cad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 开始分析：
**1、默认不改写任何事件**

- 1、触摸A区域（按下、抬起）
```
08-10 14:58:08.131 11780-11780/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 14:58:08.132 11780-11780/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 14:58:08.697 11780-11780/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 14:58:08.697 11780-11780/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
根据日志分析：记录下了Activity的分发dispatchTouchEvent和onTouchEvent事件，0为按下动作，1为抬起动作。

得出结论：默认Activity将触摸事件分发下去，并且没有子View消费情况下， 自己消费；

- 2、触摸B区域（按下、移动、抬起）
```
08-10 15:11:43.128 11780-11780/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 15:11:43.129 11780-11780/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 15:11:43.129 11780-11780/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 15:11:43.129 11780-11780/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 15:11:43.129 11780-11780/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 15:11:43.519 11780-11780/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 15:11:43.519 11780-11780/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 15:11:43.536 11780-11780/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 15:11:43.536 11780-11780/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 15:11:43.553 11780-11780/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 15:11:43.553 11780-11780/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 15:11:43.569 11780-11780/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 15:11:43.570 11780-11780/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 15:11:43.875 11780-11780/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 15:11:43.875 11780-11780/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：

1、按下后，Activity将事件分发给ViewGroup，ViewGroup返回默认super.dispatchTouchEvent(ev)将事件分发下去并传递给ViewGroup的onInterceptTouchEvent函数；

2、onInterceptTouchEvent返回默认的super.onInterceptTouchEvent(ev)，事件传递给ViewGroup的onTouchEvent方法

3、onTouchEvent依然返回默认的super.onTouchEvent(event)，不进行消费，将事件返回给上一层的Activity，Activity调用onTouchEvent，处理了按下事件。

4、手指移动，Activity的dispatchTouchEvent方法将事件分发。因为按下事件中，已经得知Activity中的ViewGroup不对触摸事件进行任何操作，并将事件返回给Activity，所以聪明的Activity这时候将分发后的事件自己消费了

5、Activity的onTouchEvent对移动事件自行处理

6、手指抬起：同样，Activity的dispatchTouchEvent方法将事件分发，自己的onTouchEvent进行消费

- 3、触摸C区域（按下、移动、抬起）
```
08-10 15:50:20.933 20620-20620/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 15:50:20.933 20620-20620/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 15:50:20.934 20620-20620/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 15:50:20.934 20620-20620/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 0
08-10 15:50:20.934 20620-20620/com.ddz.lifestyle E/View---------onTouchEvent-------: 0
08-10 15:50:20.943 20620-20620/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 15:50:20.944 20620-20620/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 15:50:21.441 20620-20620/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 15:50:21.441 20620-20620/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 15:50:21.458 20620-20620/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 15:50:21.458 20620-20620/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 15:50:21.475 20620-20620/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 15:50:21.475 20620-20620/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 15:50:21.492 20620-20620/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 15:50:21.492 20620-20620/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 15:50:21.786 20620-20620/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 15:50:21.786 20620-20620/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 15:50:21.787 20620-20620/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 15:50:21.787 20620-20620/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：结果和在B区域触摸结果类似

1、按下时，Activity使用dispatchTouchEvent方法将事件分发给ViewGroup

2、ViewGroup使用dispatchTouchEvent方法默认将事件分发给自己的onInterceptTouchEvent方法，onInterceptTouchEvent方法默认不拦截，将事件传递给其中的View

3、View的dispatchTouchEvent方法默认将事件分发给自己的onTouchEvent方法，onTouchEvent返回默认super.onTouchEvent(event)，将事件回传给ViewGroup的onTouchEvent方法

4、ViewGroup的onTouchEvent也默认将事件传递给Activity的onTouchEvent方法

5、按下事件只好由Activity的onTouchEvent方法自己处理

6、手指移动，Activity已经知道其中的ViewGroup及View都不处理触摸事件，所以使用自己的dispatchTouchEvent方法将移动事件传递给自己的onTouchEvent方法消费

7、手指抬起，同第6步一样，自己分发自己处理。


**2、改写ViewGroup的dispatchTouchEvent方法，返回false**

- 1、触摸B区域（按下、移动、抬起）
```
08-10 16:22:05.686 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 16:22:05.686 3933-3933/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 16:22:05.686 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 16:22:07.268 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:22:07.268 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 16:22:07.352 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:22:07.352 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 16:22:07.436 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:22:07.436 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 16:22:07.486 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:22:07.486 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 16:22:07.873 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:22:07.873 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 16:22:07.890 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:22:07.890 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 16:22:07.892 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 16:22:07.892 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：

1、按下时，Activity使用dispatchTouchEvent将事件分发给ViewGroup

2、因为ViewGroup的dispatchTouchEvent返回false，即事件不再继续向下分发，事件被返回给Activity的onTouchEvent进行消费

3、手指移动：Activity既然使用dispatchTouchEvent将事件分发，但从按下事件的反馈中，Activity已经得知ViewGroup不再分发事件，并且将事件返回，所以Activity还是自己用onTouchEvent将事件消费了

4、手指抬起：同手指移动一样，自己分发自己消费

- 2、触摸C区域（按下、移动、抬起）
```
08-10 16:31:17.409 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 16:31:17.409 3933-3933/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 16:31:17.409 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 16:31:17.912 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:31:17.912 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 16:31:17.929 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:31:17.929 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 16:31:17.963 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:31:17.963 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 16:31:18.194 3933-3933/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 16:31:18.194 3933-3933/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：

1、同触摸B区域一的结果，说明ViewGroupd的ispatchTouchEvent返回false，即不再分发事件，其中任何View得不到触摸事件

**3、改写ViewGroup的dispatchTouchEvent方法，返回true**

- 1、触摸B区域（按下、移动、抬起）
```
08-10 16:43:37.080 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 16:43:37.080 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 16:43:37.226 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:43:37.226 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 16:43:37.243 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:43:37.243 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 16:43:37.260 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:43:37.411 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 16:43:37.444 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:43:37.445 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 16:43:37.464 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 16:43:37.464 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 1
```
分析：
1、按下时，Activity使用dispatchTouchEvent方法将事件分发；
2、ViewGroup接收到事件，但是ViewGroup的dispatchTouchEvent返回true，即ViewGroup的dispatchTouchEvent将事件消费，不再分发。
3、手指移动、手指抬起：同按下状态一下，Activity使用dispatchTouchEvent分发的事件被ViewGroup的dispatchTouchEvent方法消费了。

- 2、触摸C区域（按下、移动、抬起）
```
08-10 16:43:37.080 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 16:43:37.080 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 16:43:37.226 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:43:37.226 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 16:43:37.243 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:43:37.243 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 16:43:37.260 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:43:37.411 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 16:43:37.444 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 16:43:37.445 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 16:43:37.464 3206-3206/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 16:43:37.464 3206-3206/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 1
```
分析：View接收不到任何事件，因为ViewGroup的dispatchTouchEvent方法返回true，事件被ViewGroup的dispatchTouchEvent方法消费掉，不再向下传递了

**4、改写ViewGroup的onInterceptTouchEvent方法，返回true**

- 1、触摸B区域（按下、移动、抬起）
```
08-10 17:01:03.576 16009-16009/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 17:01:03.576 16009-16009/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 17:01:03.576 16009-16009/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 17:01:03.576 16009-16009/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 17:01:03.576 16009-16009/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 17:01:04.148 16009-16009/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:01:04.148 16009-16009/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:01:04.165 16009-16009/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:01:04.165 16009-16009/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:01:04.448 16009-16009/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:01:04.448 16009-16009/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:01:04.448 16009-16009/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 17:01:04.448 16009-16009/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：
1、手指按下：Activity的dispatchTouchEvent将事件分发给ViewGroup
2、ViewGroup默认的dispatchTouchEvent方法将事件分发，传递给onInterceptTouchEvent方法，onInterceptTouchEvent方法返回true，将事件拦截，供给自己的onTouchEvent方法消费，
3、ViewGroup的onTouchEvent方法默认不消费，将事件返回给Activity的onTouchEvent方法，最终按下事件由Activity的onTouchEvent自己消费
4、手指移动：Activity辛辛苦苦分发的按下事件，ViewGroup不领情，那么这次移动事件，Activity使用dispatchTouchEvent将事件分发给自己的onTouchEvent，自己消费掉
5、手指抬起：同手指移动一样，Activity自己分发自己消费

- 2、触摸C区域（按下、移动、抬起）
```
08-10 17:06:27.835 16009-16009/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 17:06:27.835 16009-16009/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 17:06:27.835 16009-16009/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 17:06:27.835 16009-16009/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 17:06:27.835 16009-16009/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 17:06:27.998 16009-16009/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:06:27.999 16009-16009/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:06:28.015 16009-16009/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:06:28.015 16009-16009/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:06:28.300 16009-16009/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:06:28.300 16009-16009/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:06:28.357 16009-16009/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 17:06:28.357 16009-16009/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：可以看出，和B区域事件一样，ViewGroup的onInterceptTouchEvent将事件拦截后自己又不消费，返回给Activity，并且ViewGroup中的View接受不到事件消息，因为ViewGroup已经拦截了事件。

**5、改写ViewGroup的onInterceptTouchEvent方法，返回false**

- 1、触摸B区域（按下、移动、抬起）
```
08-10 17:13:47.725 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 17:13:47.725 19111-19111/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 17:13:47.725 19111-19111/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 17:13:47.725 19111-19111/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 17:13:47.725 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 17:13:47.786 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:13:47.786 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:13:47.803 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:13:47.803 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:13:48.005 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:13:48.005 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:13:48.009 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 17:13:48.009 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：
1、手指按下：Activity的dispatchTouchEvent将事件分发给ViewGroup
2、ViewGroup默认的dispatchTouchEvent方法将事件分发，传递给onInterceptTouchEvent方法，onInterceptTouchEvent方法返回false，不再拦截，但是ViewGroup触摸区域没有View，所以事件传递给自己onTouchEvent方法
3、ViewGroup的onTouchEvent方法默认不消费，将事件返回给Activity的onTouchEvent方法，最终按下事件由Activity的onTouchEvent自己消费
4、手指移动：Activity辛辛苦苦分发的按下事件，ViewGroup不领情，那么这次移动事件，Activity使用dispatchTouchEvent将事件分发给自己的onTouchEvent，自己消费掉
5、手指抬起：同手指移动一样，Activity自己分发自己消费

- 2、触摸C区域（按下、移动、抬起）
```
08-10 17:20:04.767 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 17:20:04.767 19111-19111/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 17:20:04.767 19111-19111/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 17:20:04.767 19111-19111/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 0
08-10 17:20:04.767 19111-19111/com.ddz.lifestyle E/View---------onTouchEvent-------: 0
08-10 17:20:04.768 19111-19111/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 17:20:04.768 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 17:20:04.896 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:20:04.896 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:20:05.047 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:20:05.047 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:20:05.064 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:20:05.064 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:20:05.081 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:20:05.081 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:20:05.095 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:20:05.095 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:20:05.095 19111-19111/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 17:20:05.095 19111-19111/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：
1、手指按下：这次流程依然是Activity的dispatchTouchEvent分发事件给ViewGroup
2、VIewGroup的dispatchTouchEvent方法返回默认的super.dispatchTouchEvent(ev)，所以事件继续分发，传递给onInterceptTouchEvent；
3、这时候onInterceptTouchEvent已经返回false，不再拦截，事件终于被传递给ViewGroup中的View了；
4、View接受到事件之后，dispatchTouchEvent方法也是返回默认super.dispatchTouchEvent(ev)，事件传递给自己的onTouchEvent方法
5、View的onTouchEvent方法不去消费，事件又被返回给ViewGroup的onTouchEvent方法；
6、但是ViewGroup的onTouchEvent方法也不消费，返回给Activity的onTouchEvent方法
7、Activity的onTouchEvent方法自己消费按下事件（Activity：都不消费，下次不给你传了）；
8、Activity说到做到，手指移动和手指抬起事件：Activity使用dispatchTouchEvent方法将事件分发给自己的onTouchEvent方法消费，自己发给自己消费，（再也不给下面的ViewGroup和View这两个什么都不干的人，哈哈哈）

**6、改写ViewGroup的onTouchEvent方法，返回true**

- 1、触摸B区域（按下、移动、抬起）
```
08-10 17:32:42.158 12826-12826/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 17:32:42.158 12826-12826/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 17:32:42.158 12826-12826/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 17:32:42.158 12826-12826/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 17:32:42.194 12826-12826/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:32:42.194 12826-12826/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 17:32:42.194 12826-12826/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 2
08-10 17:32:42.227 12826-12826/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:32:42.227 12826-12826/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 17:32:42.227 12826-12826/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 2
08-10 17:32:42.428 12826-12826/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:32:42.428 12826-12826/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 17:32:42.428 12826-12826/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 2
08-10 17:32:42.442 12826-12826/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 17:32:42.442 12826-12826/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 1
08-10 17:32:42.442 12826-12826/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 1
```
分析：1、手指按下：Activity的dispatchTouchEvent将事件分发给ViewGroup
2、ViewGroup默认的dispatchTouchEvent方法将事件分发，传递给onInterceptTouchEvent方法
3、ViewGroup的onInterceptTouchEvent方法默认返回super.onInterceptTouchEvent(ev)，不拦截，但是ViewGroup触摸区域没有View，所以事件传递给自己onTouchEvent方法
3、ViewGroup的onTouchEvent方法返回true， 将按下事件消费掉了，不再返回
4、手指移动：Activity的dispatchTouchEvent方法依然分发事件给ViewGroup；
5、ViewGroup在手指按下事件中已经知道了自己的onInterceptTouchEvent方法默认不拦截，所以移动事件就不再问onInterceptTouchEvent方法了，直接传递给onTouchEvent方法
6、ViewGroup的onTouchEvent方法将事件消费
7、手指提起：同手指移动一模一样的流程，最终事件由ViewGroup的onTouchEvent方法消费掉

- 2、触摸C区域（按下、移动、抬起）
```
08-10 17:39:21.134 12826-12826/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 17:39:21.135 12826-12826/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 17:39:21.135 12826-12826/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 17:39:21.135 12826-12826/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 0
08-10 17:39:21.135 12826-12826/com.ddz.lifestyle E/View---------onTouchEvent-------: 0
08-10 17:39:21.140 12826-12826/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 17:39:21.261 12826-12826/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:39:21.261 12826-12826/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 17:39:21.261 12826-12826/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 2
08-10 17:39:21.277 12826-12826/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:39:21.278 12826-12826/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 17:39:21.278 12826-12826/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 2
08-10 17:39:21.428 12826-12826/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:39:21.428 12826-12826/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 17:39:21.428 12826-12826/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 2
08-10 17:39:21.435 12826-12826/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 17:39:21.435 12826-12826/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 1
08-10 17:39:21.435 12826-12826/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 1
```
分析：
1、手指按下：Activity的dispatchTouchEvent将事件分发给ViewGroup
2、ViewGroup默认的dispatchTouchEvent方法将事件分发，传递给onInterceptTouchEvent方法
3、ViewGroup的onInterceptTouchEvent方法默认返回super.onInterceptTouchEvent(ev)，不拦截，这时候ViewGroup中有了View，事件传递给View的dispatchTouchEvent
4、View的dispatchTouchEvent将事件分发给自己的onTouchEvent方法
5、View的onTouchEvent不想消费事件，将事件返回给ViewGroup的onTouchEvent方法
6、ViewGroup的onTouchEvent方法返回true，将事件拿来自己消费掉了，按下事件结束了
7、手指移动：Activity的dispatchTouchEvent方法依然分发事件给ViewGroup；
8、ViewGroup在手指按下事件中已经知道了自己的onInterceptTouchEvent方法默认不拦截，所以移动事件就不再问onInterceptTouchEvent方法了，而且按下事件的时候，ViewGroup中的View不消费事件，所以这次也不再传递给View了，直接传递给ViewGroup的onTouchEvent
9、ViewGroup的onTouchEvent方法返回true，所以消费了移动事件
10、手指抬起同手指移动一样

**7、改写ViewGroup的onTouchEvent方法，返回false**

- 1、触摸B区域（按下、移动、抬起）
```
08-10 17:49:17.141 29978-29978/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 17:49:17.142 29978-29978/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 17:49:17.142 29978-29978/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 17:49:17.142 29978-29978/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 17:49:17.142 29978-29978/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 17:49:17.245 29978-29978/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:49:17.245 29978-29978/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:49:17.295 29978-29978/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:49:17.295 29978-29978/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:49:17.409 29978-29978/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:49:17.409 29978-29978/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:49:17.409 29978-29978/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 17:49:17.409 29978-29978/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：
1、手指按下：Activity的dispatchTouchEvent将事件分发给ViewGroup
2、ViewGroup默认的dispatchTouchEvent方法将事件分发，传递给onInterceptTouchEvent方法；
3、ViewGroup的onInterceptTouchEvent方法返回默认的super.onInterceptTouchEvent(ev)，即不拦截，但是ViewGroup触摸区域没有View，所以事件传递给自己onTouchEvent方法
3、ViewGroup的onTouchEvent方法返回false，不消费事件，将事件返回给Activity的onTouchEvent方法，最终按下事件由Activity的onTouchEvent自己消费
4、手指移动：Activity辛辛苦苦分发的按下事件，ViewGroup不领情，那么这次移动事件，Activity使用dispatchTouchEvent将事件分发给自己的onTouchEvent，自己消费掉
5、手指抬起：同手指移动一样，Activity自己分发自己消费

- 2、触摸C区域（按下、移动、抬起）
```
08-10 17:52:32.229 29978-29978/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 17:52:32.230 29978-29978/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 17:52:32.230 29978-29978/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 17:52:32.230 29978-29978/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 0
08-10 17:52:32.230 29978-29978/com.ddz.lifestyle E/View---------onTouchEvent-------: 0
08-10 17:52:32.230 29978-29978/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 17:52:32.230 29978-29978/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 17:52:32.331 29978-29978/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:52:32.331 29978-29978/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:52:32.348 29978-29978/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:52:32.348 29978-29978/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:52:32.735 29978-29978/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 17:52:32.735 29978-29978/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 17:52:32.740 29978-29978/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 17:52:32.741 29978-29978/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：
1、手指按下：这次流程依然是Activity的dispatchTouchEvent分发事件给ViewGroup
2、VIewGroup的dispatchTouchEvent方法返回默认的super.dispatchTouchEvent(ev)，所以事件继续分发，传递给onInterceptTouchEvent；
3、这时候onInterceptTouchEvent也默认不拦截，事件终于被传递给ViewGroup中的View了；
4、View接受到事件之后，dispatchTouchEvent方法也是返回默认super.dispatchTouchEvent(ev)，事件传递给自己的onTouchEvent方法
5、View的onTouchEvent方法默认不消费，事件又被返回给ViewGroup的onTouchEvent方法；
6、但是ViewGroup的onTouchEvent方法返回false，不消费，事件又返回给Activity的onTouchEvent方法
7、Activity的onTouchEvent方法自己消费按下事件（Activity：都不消费，下次不给你传了）；
8、Activity说到做到，手指移动和手指抬起事件：Activity使用dispatchTouchEvent方法将事件分发给自己的onTouchEvent方法消费，自己发给自己消费，（再也不给下面的ViewGroup和View这两个什么都不干的人，哈哈哈）

**8、改写View的dispatchTouchEvent方法，返回false**

- 1、触摸C区域（按下、移动、抬起）
```
08-10 19:20:06.394 29197-29197/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 19:20:06.394 29197-29197/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 19:20:06.394 29197-29197/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 19:20:06.394 29197-29197/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 0
08-10 19:20:06.394 29197-29197/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 19:20:06.395 29197-29197/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 19:20:06.529 29197-29197/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 19:20:06.529 29197-29197/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 19:20:06.563 29197-29197/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 19:20:06.563 29197-29197/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 19:20:06.579 29197-29197/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 19:20:06.580 29197-29197/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 19:20:06.634 29197-29197/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 19:20:06.634 29197-29197/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：
1、手指按下：Activity的dispatchTouchEvent分发事件给ViewGroup；
2、VIewGroup的dispatchTouchEvent方法返回默认的super.dispatchTouchEvent(ev)，所以事件继续分发，传递给onInterceptTouchEvent；
3、ViewGroup的onInterceptTouchEvent也默认不拦截，事件终于被传递给ViewGroup中的View了；
4、View接受到事件之后，dispatchTouchEvent方法返回false，事件不再分发，返回给ViewGroup的onTouchEvent；
5、ViewGroup的onTouchEvent方法默认也不消费，事件再次返回给Activity的onTouchEvent
6、最终，按下事件由Activity的onTouchEvent方法消费掉。
7、由按下事件可知：事件从View-->ViewGroup-->View，事件最终返回给Activity，所以手指移动、抬起事件，不会再继续向下分发，直接有Activity的dispatchTouchEvent传递给自己的onTouchEvent事件消费

**9、改写View的dispatchTouchEvent方法，返回true**

- 1、触摸C区域（按下、移动、抬起）
```
08-10 19:29:34.275 22834-22834/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 19:29:34.275 22834-22834/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 19:29:34.275 22834-22834/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 19:29:34.275 22834-22834/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 0
08-10 19:29:34.621 22834-22834/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 19:29:34.621 22834-22834/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 19:29:34.621 22834-22834/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 2
08-10 19:29:34.621 22834-22834/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 2
08-10 19:29:34.637 22834-22834/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 19:29:34.637 22834-22834/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-10 19:29:34.637 22834-22834/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 2
08-10 19:29:34.638 22834-22834/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 2
08-10 19:29:34.642 22834-22834/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 19:29:34.642 22834-22834/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 1
08-10 19:29:34.642 22834-22834/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 1
08-10 19:29:34.642 22834-22834/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 1
```
分析：
1、手指按下：Activity的dispatchTouchEvent分发事件给ViewGroup；
2、VIewGroup的dispatchTouchEvent方法返回默认的super.dispatchTouchEvent(ev)，所以事件继续分发，传递给onInterceptTouchEvent；
3、ViewGroup的onInterceptTouchEvent默认不拦截，事件终于被传递给ViewGroup中的View了；
4、View接受到事件之后，dispatchTouchEvent方法返回true，事件不再分发，直接被View的dispatchTouchEvent消费，不再传递
5、手指移动、手指抬起事件同上，全部被View的dispatchTouchEvent方法消费

**10、改写View的onTouchEvent方法，返回false**

- 1、触摸C区域（按下、移动、抬起）
```
08-10 21:18:33.082 8039-8039/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-10 21:18:33.082 8039-8039/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-10 21:18:33.082 8039-8039/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-10 21:18:33.082 8039-8039/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 0
08-10 21:18:33.082 8039-8039/com.ddz.lifestyle E/View---------onTouchEvent-------: 0
08-10 21:18:33.082 8039-8039/com.ddz.lifestyle E/ViewGroup-------onTouchEvent-------: 0
08-10 21:18:33.082 8039-8039/com.ddz.lifestyle E/activity-------onTouchEvent-------: 0
08-10 21:18:33.108 8039-8039/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 21:18:33.108 8039-8039/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 21:18:33.125 8039-8039/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 21:18:33.125 8039-8039/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 21:18:33.141 8039-8039/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-10 21:18:33.141 8039-8039/com.ddz.lifestyle E/activity-------onTouchEvent-------: 2
08-10 21:18:33.248 8039-8039/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-10 21:18:33.248 8039-8039/com.ddz.lifestyle E/activity-------onTouchEvent-------: 1
```
分析：
1、手指按下：这次流程依然是Activity的dispatchTouchEvent分发事件给ViewGroup
2、VIewGroup的dispatchTouchEvent方法返回默认的super.dispatchTouchEvent(ev)，所以事件继续分发，传递给onInterceptTouchEvent；
3、这时候onInterceptTouchEvent也默认不拦截，事件终于被传递给ViewGroup中的View了；
4、View接受到事件之后，dispatchTouchEvent方法也是返回默认super.dispatchTouchEvent(ev)，事件传递给自己的onTouchEvent方法
5、View的onTouchEvent方法返回false，不消费事件，事件又被返回给ViewGroup的onTouchEvent方法；
6、但是ViewGroup的onTouchEvent方法默认不消费，事件又返回给Activity的onTouchEvent方法
7、最后Activity的onTouchEvent方法自己消费按下事件（Activity：都不消费，下次不给你传了）；
8、手指移动：Activity已经知道ViewGroup和View都不消费事件，所以手指移动事件就自己使用dispatchTouchEvent传递给自己的onTouchEvent方法消费掉
9、手指抬起：同手指移动

**11、改写View的onTouchEvent方法，返回true**

- 1、触摸C区域（按下、移动、抬起）
```
08-11 13:33:27.684 11900-11900/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 0
08-11 13:33:27.684 11900-11900/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 0
08-11 13:33:27.684 11900-11900/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 0
08-11 13:33:27.684 11900-11900/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 0
08-11 13:33:27.685 11900-11900/com.ddz.lifestyle E/View---------onTouchEvent-------: 0
08-11 13:33:27.761 11900-11900/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-11 13:33:27.761 11900-11900/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-11 13:33:27.761 11900-11900/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 2
08-11 13:33:27.761 11900-11900/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 2
08-11 13:33:27.761 11900-11900/com.ddz.lifestyle E/View---------onTouchEvent-------: 2
08-11 13:33:27.962 11900-11900/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 2
08-11 13:33:27.962 11900-11900/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 2
08-11 13:33:27.962 11900-11900/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 2
08-11 13:33:27.962 11900-11900/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 2
08-11 13:33:27.962 11900-11900/com.ddz.lifestyle E/View---------onTouchEvent-------: 2
08-11 13:33:27.993 11900-11900/com.ddz.lifestyle E/activity-------dispatchTouchEvent-------: 1
08-11 13:33:27.994 11900-11900/com.ddz.lifestyle E/ViewGroup-------dispatchTouchEvent-------: 1
08-11 13:33:27.994 11900-11900/com.ddz.lifestyle E/ViewGroup-------onInterceptTouchEvent-------: 1
08-11 13:33:27.994 11900-11900/com.ddz.lifestyle E/View---------dispatchTouchEvent-------: 1
08-11 13:33:27.994 11900-11900/com.ddz.lifestyle E/View---------onTouchEvent-------: 1
```
分析：
1、手指按下：这次流程依然是Activity的dispatchTouchEvent分发事件给ViewGroup
2、VIewGroup的dispatchTouchEvent方法返回默认的super.dispatchTouchEvent(ev)，所以事件继续分发，传递给onInterceptTouchEvent；
3、这时候onInterceptTouchEvent也默认不拦截，事件终于被传递给ViewGroup中的View了；
4、View接受到事件之后，dispatchTouchEvent方法也是返回默认super.dispatchTouchEvent(ev)，事件传递给自己的onTouchEvent方法
5、View的onTouchEvent方法返回true，消费事件按下事件结束
6、手指移动：Activity已经知道ViewGroup中的View的onTouchEvent方法要消费事件，手指移动事件就重复按下事件的流程，一直传递给View的onTouchEvent方法，直到事件被消费；
7、手指抬起：同手指移动一样

###总结
####ViewGroup：
**dispatchTouchEvent 分发**
ViewGroup接收到事件之后，根据dispatchTouchEvent决定是否分发下去
- 1、默认返回 super.dispatchTouchEvent(ev)方法，即默认分发事件
- 2、如果返回false；事件将不再分发，直接返回给上一层的onTouch方法，并且后面的事件将不再分发给当前ViewGroup，上层直接自己分发并消费掉
- 3、如果返回true，事件将不再分发并且由ViewGroup的dispatchTouchEvent 消费掉，后面的触摸事件也会同样被ViewGroup的dispatchTouchEvent 消费

**onInterceptTouchEvent 拦截**
ViewGroup的dispatchTouchEvent如果默认将事件分发下去，传递给onInterceptTouchEvent方法，由onInterceptTouchEvent方法决定是否拦截
- 1、默认返回super.onInterceptTouchEvent(ev)  即不拦截，事件继续传递
- 2、改为返回true，事件直接传递给ViewGroup的onTouchEvent方法消费，不再传递给ViewGroup中的View
- 3、如果返回false，事件将传递给ViewGroup中的View去处理

**onTouchEvent 消费**
如果ViewGroup中的onInterceptTouchEvent 默认不拦截事件，这时根据ViewGroup的onTouchEvent返回值来判断
- 1、默认返回super.onTouchEvent(event)，事件将传递给ViewGroup中的View进行处理，如果ViewGroup中View不消费事件，事件将会返回给ViewGroup的onTouchEvent方法处理，ViewGroup的onTouchEvent方法默认不处理，返回给上层的onTouchEvent方法处理
- 2 、如果返回true，，事件也会传递给ViewGroup中的View进行处理，如果ViewGroup中View不消费事件，事件将会返回给ViewGroup的onTouchEvent方法，这时ViewGroup的onTouchEvent直接将事件消费掉，不返回上层的onTouchEvent方法了
- 3、如果返回false，事件将默认由ViewGroup传递给View，View不处理又返回给ViewGroup的onTouchEvent，ViewGroup的onTouchEvent返回false，所以事件又返回给上层的onTouchEvent方法

####View：
**dispatchTouchEvent 分发**
View接收到事件后，根据dispatchTouchEvent方法返回值，判断是否继续分发
- 1、默认返回super.dispatchTouchEvent(ev)，事件将分发下去，传递给View自己的onTouchEvent进行处理，
- 2、如果返回false，事件将不分发，直接返回给上层ViewGroup的onTouchEvent方法进行处理
- 3、如果返回true，事件也不再分发，直接由View的dispatchTouchEvent 进行消费，并且以后的事件同样直接被View的dispatchTouchEvent 方法消费了

**onTouchEvent 消费**
如果View的dispatchTouchEvent 将事件传递给onTouchEvent方法，将根据onTouchEvent方法的返回值决定是否消费事件
- 1、默认返回super.onTouchEvent(event)，默认不消费事件，事件将返回给上层ViewGroup的onTouchEvent方法处理
- 2、如果改为true，将事件消费，不再返回，以后其他事件同样被onTouchEvent方法消费
- 3 、如果改为false，同默认方法一样，不再消费，返回给上层处理

所以，触摸事件的所有流程已经很清晰了

对于事件分发：（dispatchTouchEvent）
**如果想事件不向下传递，自己消费掉**  ：将当前的dispatchTouchEvent返回true；
**如果想事件不向下传递，返回给上层**  ：将当前的dispatchTouchEvent返回false；
对于事件拦截：（onInterceptTouchEvent）
**如果想拦截事件，给自己的onTouchEvent方法消费** ：将onInterceptTouchEvent返回true
**如果不拦截事件，默认向下传递** ：将onInterceptTouchEvent返回false或者返回默认值
对于事件消费：（onTouchEvent）
**如果不想消费，返回给上层** ：将onTouchEvent返回默认或者返回false；
**如果想消费，不再返回** ：将onTouchEvent返回true；