---
title: 自定义视图(3) —— 视图交互
date: 2017-08-13 13:28:45
tags:
- Android
- UI
- 'Android 用户界面的最佳实践'
categories:
- Android

---

![android](http://oxwfu3w0v.bkt.clouddn.com/qiniu.jpg)

绘制UI只是创建自定义视图的一部分。
您还需要让您的视图以一种与您模拟的实际操作非常相似的方式对用户输入作出响应。
对象应该以与实际对象相同的方式操作。
例如，图像不应该立即跳出存在并重新出现在其他地方，因为现实世界中的物体不这样做。
相反，图像应该从一个地方移动到另一个地方。

用户还可以在界面中感知微妙的行为或感觉，并对模仿真实世界的细微差别做出最好的反应。
例如，当用户fling一个UI对象时，他们应该在开始时感觉到摩擦力，从而延迟了这个动作，然后在结束时感觉的动量，使这个动作超越了fling。

这个课程演示了如何使用Android框架的特性将这些真实的行为添加到您的自定义视图中。

<!-- more -->

## 处理输入的手势

与许多其他的UI框架一样，Android支持输入事件模型。
用户操作被转换为触发回调的事件，您可以覆盖回调，以定制应用程序对用户的响应。
Android系统中最常见的输入事件是触摸，它触发[onTouchEvent(android.view.motionevent)](https://developer.android.google.cn/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))。
覆盖该方法来处理事件:

```java
 @Override
 public boolean onTouchEvent(MotionEvent event) {
    return super.onTouchEvent(event);
 }
```

触摸事件本身并不是特别有用。
现代的触摸UIs定义了交互作用，比如点击、拉动、推送、投掷和缩放等。
为了将原始的触摸事件转化为手势，Android提供了手势探测器([GestureDetector](https://developer.android.google.cn/reference/android/view/GestureDetector.html))。

通过传递一个实现[GestureDetector.OnGestureListener](https://developer.android.google.cn/reference/android/view/GestureDetector.OnGestureListener.html)的类的实例来构造一个手势检测器。
如果您只想处理一些手势，您可以扩展[GestureDetector.SimpleOnGestureListener](https://developer.android.google.cn/reference/android/view/GestureDetector.SimpleOnGestureListener.html)，而不是实现GestureDetector.OnGestureListener接口。
例如，这段代码创建了一个扩展GestureDetector.SimpleOnGestureListener和覆盖[onDown(MotionEvent)](https://developer.android.google.cn/reference/android/view/GestureDetector.SimpleOnGestureListener.html#onDown(android.view.MotionEvent))的类。

```java
class mListener extends GestureDetector.SimpleOnGestureListener {
   @Override
   public boolean onDown(MotionEvent e) {
       return true;
   }
}
mDetector = new GestureDetector(PieChart.this.getContext(), new mListener());
```


无论您是否使用[GestureDetector.SimpleOnGestureListener](https://developer.android.google.cn/reference/android/view/GestureDetector.SimpleOnGestureListener.html)，您必须始终实现一个返回true的[onDown()](https://developer.android.google.cn/reference/android/view/GestureDetector.OnGestureListener.html#onDown(android.view.MotionEvent))方法。
这一步是必要的，因为所有的手势都是从`onDown()`消息开始的。
如果`onDown()`返回false，如`GestureDetector.OnGestureListener`所做的那样，系统假定您想要忽略该手势的其余部分，而`GestureDetector.OnGestureListener`的其他方法则永远不会被调用。
如果您真的想忽略一个完整的手势，那么惟一时间是`onDown()`返回false的。
一旦您实现了[GestureDetector.OnGestureListener](https://developer.android.google.cn/reference/android/view/GestureDetector.OnGestureListener.html)并创建了一个手势检测器的实例，您就可以使用您的[手势检测器](https://developer.android.google.cn/reference/android/view/GestureDetector.html)来解释您在[onTouchEvent()](https://developer.android.google.cn/reference/android/view/GestureDetector.html#onTouchEvent(android.view.MotionEvent))中接收到的触摸事件。

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
   boolean result = mDetector.onTouchEvent(event);
   if (!result) {
       if (event.getAction() == MotionEvent.ACTION_UP) {
           stopScrolling();
           result = true;
       }
   }
   return result;
}
```

当您通过onTouchEvent()一个触摸事件时，它不承认它是一个手势的一部分，它返回`false`。
然后你可以运行你自己的自定义手势检测代码。

## 创建物理上合理的运动

手势是控制触屏设备的一种强有力的方式，但它们可能是违反直觉的，而且很难记住，除非它们产生了看似合理的结果。
一个很好的例子就是“fling”手势，用户可以快速地将手指移动到屏幕上，然后抬起它。
如果UI的响应是快速移动到fling的方向，然后减速，就像用户推着一个飞轮并让它旋转一样，这个手势是有意义的。

然而，模拟飞轮的感觉并不简单。
需要大量的物理和数学才能使飞轮模型正常工作。
幸运的是，Android提供了帮助类来模拟这个和其他行为。
[Scroller](https://developer.android.google.cn/reference/android/widget/Scroller.html)类是处理飞轮式风格的投掷动作的基础。

要开始一个投掷，调用[fling()](https://developer.android.google.cn/reference/android/widget/Scroller.html#fling(int,%20int,%20int,%20int,%20int,%20int,%20int,%20int))以初始速度和最小值和最大值x和y值。
对于速度值，可以使用手势检测器计算的值。

```java
@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
   mScroller.fling(currentX, currentY, velocityX / SCALE, velocityY / SCALE, minX, minY, maxX, maxY);
   postInvalidate();
}
```

> ⚠️:虽然手势检测的速度是物理精确的，但是很多开发者认为使用这个值会让动画的速度太快。
把x和y的速度除以4到8是很常见的。

fling()的调用建立了投掷动作的物理模型。
然后，您需要通过定期调用[Scroller.computeScrollOffset()](https://developer.android.google.cn/reference/android/widget/Scroller.html#computeScrollOffset())来更新[Scroller](https://developer.android.google.cn/reference/android/widget/Scroller.html)。
通过读取当前时间和使用物理模型来计算x和y的位置，computeScrollOffset()更新Scroller对象的内部状态。
调用getCurrX()和getCurrY()来检索这些值。

大多数视图将Scroller对象的x和y位置直接传递给scrollTo()。
PieChart的例子有一点不同:它使用当前的scroll  y轴来设置图表的旋转角度。

```java
if (!mScroller.isFinished()) {
    mScroller.computeScrollOffset();
    setPieRotation(mScroller.getCurrY());
}
```

Scroller类为您计算滚动位置，但它不会自动将这些位置应用到您的视图中。
你的责任是确保你得到并应用新的坐标，使滚动的动画看起来很平滑。
有两种方法可以做到这一点:

* 在调用fling()之后调用postInvalidate()，以强制重新绘制。
该技术要求您在onDraw()中计算滚动偏移量，并在每次滚动偏移量变化时调用postInvalidate()。
* 在fling的持续时间设置一个ValueAnimator，并添加一个侦听器，通过调用addUpdateListener()来处理动画更新。

PieChart的例子使用了第二种方法。
这种技术的设置稍微复杂一些，但是它与动画系统更紧密地合作，并且不需要潜在的不必要的视图失效。
缺点是，[ValueAnimator](https://developer.android.google.cn/reference/android/animation/ValueAnimator.html)在API级别11之前是不可用的，所以这种技术不能用于运行Android版本低于3.0的设备上。

> 注意:您可以在针对较低API级别的应用程序中使用ValueAnimator。
您只需要确保在运行时检查当前的API级别，如果当前级别低于11，就忽略对视图动画系统的调用。

```java
mScroller = new Scroller(getContext(), null, true);
       mScrollAnimator = ValueAnimator.ofFloat(0,1);
       mScrollAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
           @Override
           public void onAnimationUpdate(ValueAnimator valueAnimator) {
               if (!mScroller.isFinished()) {
                   mScroller.computeScrollOffset();
                   setPieRotation(mScroller.getCurrY());
               } else {
                   mScrollAnimator.cancel();
                   onScrollFinished();
               }
           }
       });

```

## 让你的过渡平滑

用户期望一个现代的UI在状态之间平稳地过渡。
UI元素会淡入淡出，而不是出现和消失。
运动的开始和结束都很顺利，而不是突然开始和停止。
Android 3.0版本的Android[属性动画框架](https://developer.android.google.cn/guide/topics/graphics/prop-animation.html)使得流畅的过渡变得容易。

要使用动画系统，只要属性更改会影响视图的外观，就不要直接更改属性。
相反，使用[ValueAnimator](https://developer.android.google.cn/reference/android/animation/ValueAnimator.html)来进行更改。
在下面的例子中，修改当前选中的饼图在PieChart中会使整个图表旋转，以便选择指针位于所选的切片中。
ValueAnimator会在几百毫秒内改变旋转，而不是立即设置新的旋转值。

```java
mAutoCenterAnimator = ObjectAnimator.ofInt(PieChart.this, "PieRotation", 0);
mAutoCenterAnimator.setIntValues(targetAngle);
mAutoCenterAnimator.setDuration(AUTOCENTER_ANIM_DURATION);
mAutoCenterAnimator.start();
```

如果您想要更改的值是基本视图属性之一，那么做动画就更容易了，因为视图有一个内置的[ViewPropertyAnimator](https://developer.android.google.cn/reference/android/view/ViewPropertyAnimator.html)，它针对多个属性的同步动画进行了优化。
例如:

```java
animate().rotation(targetAngle).setDuration(ANIM_DURATION).start();
```
