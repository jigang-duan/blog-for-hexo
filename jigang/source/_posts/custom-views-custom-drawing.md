---
title: 自定义视图(2) —— 自定义绘制
date: 2017-08-13 12:40:03
tags:
- Android
- UI
- 'Android 用户界面的最佳实践'
categories:
- Android

---

![android](http://oxwfu3w0v.bkt.clouddn.com/qiniu.jpg)


自定义视图最重要的部分是它的外观。
根据您的应用程序的需要，自定义绘制可以很简单或复杂。
这一课涵盖了一些最常见的操作。

<!-- more -->

## 覆写onDraw()

绘制自定义视图最重要的步骤是覆盖[onDraw()](https://developer.android.google.cn/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法。
onDraw()的参数是一个画布对象，视图可以使用它来绘制自己。
[Canvas](https://developer.android.google.cn/reference/android/graphics/Canvas.html)类定义了用于绘制文本、行、位图和许多其他图形原语的方法。
您可以在onDraw()中使用这些方法来创建定制的用户界面(UI)。

不过，在您调用任何绘图方法之前，需要创建一个[Paint](https://developer.android.google.cn/reference/android/graphics/Paint.html)对象。
下一节将更详细地讨论`Paint`

## 创建绘图对象

[android.graphics](https://developer.android.google.cn/reference/android/graphics/package-summary.html)框架将绘图分为两个领域:

* 画什么，用`Canvas`来处理
* 如何画，用`Paint`来处理

例如，Canvas提供了一种绘制线条的方法，而`Paint`则提供了定义线条颜色的方法。
Canvas有一个绘制矩形的方法，而`Paint`则定义了是用一个颜色填充这个矩形，还是把它留空。
简单地说，Canvas定义了你可以在屏幕上绘制的图形，而`Paint`则定义了你所绘制的每一个图形的颜色、样式、字体等等。

因此，在绘制任何东西之前，您需要创建一个或多个`Paint`对象。PieChart的例子在一个名为init的方法中这样做，这个方法在构造函数中调用的:

```java
private void init() {
   mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mTextPaint.setColor(mTextColor);
   if (mTextHeight == 0) {
       mTextHeight = mTextPaint.getTextSize();
   } else {
       mTextPaint.setTextSize(mTextHeight);
   }

   mPiePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mPiePaint.setStyle(Paint.Style.FILL);
   mPiePaint.setTextSize(mTextHeight);

   mShadowPaint = new Paint(0);
   mShadowPaint.setColor(0xff101010);
   mShadowPaint.setMaskFilter(new BlurMaskFilter(8, BlurMaskFilter.Blur.NORMAL));

   ...
```

提前创建对象是一个重要的优化。
视图经常被重新绘制，许多绘图对象需要昂贵的初始化。
在onDraw()方法中创建绘图对象可以显著降低性能，并使UI显得迟缓。

## 处理布局事件

为了正确地绘制您的自定义视图，您需要知道它的大小。
复杂的自定义视图通常需要根据屏幕上的区域的大小和形状来执行多个布局计算。
你不应该在屏幕上对视图的大小做出假设。
即使只有一个应用程序使用你的视图，这个应用程序也需要处理不同的屏幕大小，屏幕密度，以及横向和横向模式的不同方面的比率。

尽管视图有许多用于处理度量的方法，但它们中的大多数不需要被覆盖。
如果您的视图不需要对它的大小进行特殊的控制，那么您只需覆盖一个方法:[onSizeChanged()](https://developer.android.google.cn/reference/android/view/View.html#onSizeChanged(int,%20int,%20int,%20int))。

[onSizeChanged()](https://developer.android.google.cn/reference/android/view/View.html#onSizeChanged(int,%20int,%20int,%20int))是在您的视图第一次分配一个大小时调用的，如果您的视图的大小因任何原因而发生变化，则调用onSizeChanged()。
在onSizeChanged()中计算位置、维度和其他与视图大小相关的值，而不是每次绘制时都重新计算它们。
在`PieChart`的例子中，onSizeChanged()是`PieChart`视图计算饼图的边界矩形和文本标签和其他可视元素的相对位置的地方。

当您的视图被分配一个大小时，布局管理器假设这个大小包含了所有视图的padding。
在计算视图的大小时，必须处理padding值。
下面是PieChart.onSizeChanged()的一个片段，展示了如何做到这一点:

```java
       // Account for padding
       float xpad = (float)(getPaddingLeft() + getPaddingRight());
       float ypad = (float)(getPaddingTop() + getPaddingBottom());

       // Account for the label
       if (mShowText) xpad += mTextWidth;

       float ww = (float)w - xpad;
       float hh = (float)h - ypad;

       // Figure out how big we can make the pie.
       float diameter = Math.min(ww, hh);
```

如果您需要更好地控制视图的布局参数，那么可以实现onMeasure()。
这个方法的参数是View.MeasureSpec值，它告诉你视图的父视图有多大，它的大小是一个硬的最大值还是一个建议。
作为一个优化，这些值被存储为打包的整数，您使用View.MeasureSpec的静态方法来解存储在每个整数中的信息。

下面是onMeasure()的一个示例实现。
在这个实现中，PieChart试图让它的区域足够大，使这个馅饼和它的label一样大:

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
   // Try for a width based on our minimum
   int minw = getPaddingLeft() + getPaddingRight() + getSuggestedMinimumWidth();
   int w = resolveSizeAndState(minw, widthMeasureSpec, 1);

   // Whatever the width ends up being, ask for a height that would let the pie
   // get as big as it can
   int minh = MeasureSpec.getSize(w) - (int)mTextWidth + getPaddingBottom() + getPaddingTop();
   int h = resolveSizeAndState(MeasureSpec.getSize(w) - (int)mTextWidth, heightMeasureSpec, 0);

   setMeasuredDimension(w, h);
}
```

在这段代码中有三件重要的事情需要注意:

* 计算考虑到视图的填充。正如前面提到的，这是视图的责任。

* 助手方法[resolveSizeAndState()](https://developer.android.google.cn/reference/android/view/View.html#resolveSizeAndState(int,%20int,%20int))用于创建最终的宽度和高度值。通过将视图所需的大小与传递给[onMeasure()](https://developer.android.google.cn/reference/android/view/View.html#onMeasure(int,%20int))的规范相比较，这个助手返回一个适当的View.MeasureSpec值。

* onMeasure()没有返回值。
相反，该方法通过调用[setMeasuredDimension()](https://developer.android.google.cn/reference/android/view/View.html#setMeasuredDimension(int,%20int))来传递其结果。
调用此方法是强制性的。
如果省略了这个调用，视图类会抛出一个运行时异常。

## 绘制

一旦您创建了对象创建和度量代码，您就可以实现onDraw()。
每个视图都以不同的方式实现onDraw()，但是有一些常见的操作是大多数视图共享的:

* 绘制文本使用[drawText()](https://developer.android.google.cn/reference/android/graphics/Canvas.html#drawText(char[],%20int,%20int,%20float,%20float,%20android.graphics.Paint))。
通过调用setTypeface()来指定字体，并通过调用setColor()来指定文本颜色。

* 使用[drawRect()](https://developer.android.google.cn/reference/android/graphics/Canvas.html#drawRect(android.graphics.Rect,%20android.graphics.Paint))、[drawOval()](https://developer.android.google.cn/reference/android/graphics/Canvas.html#drawOval(android.graphics.RectF,%20android.graphics.Paint))和[drawArc()](****)绘制原始图形。
通过调用setStyle()来改变这些形状是否填充、概括或两者都有。

* 使用[Path](https://developer.android.google.cn/reference/android/graphics/Path.html)类绘制更复杂的图形。
通过向路径对象添加行和曲线来定义一个形状，然后使用[drawPath()](https://developer.android.google.cn/reference/android/graphics/Canvas.html#drawPath(android.graphics.Path,%20android.graphics.Paint))来绘制图形。
就像原始的形状一样，可以根据setStyle()对路径进行概述、填充，或者两者都可以。

* 通过创建[LinearGradient](https://developer.android.google.cn/reference/android/graphics/LinearGradient.html)对象来定义渐变填充。
调用setShader()在填充的图形上使用你的线性渐变。

* 绘制位图使用[drawBitmap()](https://developer.android.google.cn/reference/android/graphics/Canvas.html#drawBitmap(android.graphics.Bitmap,%20android.graphics.Matrix,%20android.graphics.Paint))。

例如，这里是绘制PieChart的代码。它使用了文本、线条和形状的混合。

```java
protected void onDraw(Canvas canvas) {
   super.onDraw(canvas);

   // Draw the shadow
   canvas.drawOval(
           mShadowBounds,
           mShadowPaint
   );

   // Draw the label text
   canvas.drawText(mData.get(mCurrentItem).mLabel, mTextX, mTextY, mTextPaint);

   // Draw the pie slices
   for (int i = 0; i < mData.size(); ++i) {
       Item it = mData.get(i);
       mPiePaint.setShader(it.mShader);
       canvas.drawArc(mBounds,
               360 - it.mEndAngle,
               it.mEndAngle - it.mStartAngle,
               true, mPiePaint);
   }

   // Draw the pointer
   canvas.drawLine(mTextX, mPointerY, mPointerX, mPointerY, mTextPaint);
   canvas.drawCircle(mPointerX, mPointerY, mPointerSize, mTextPaint);
}
```
