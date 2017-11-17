---
title: android 代码优化 - 注解检查
date: 2017-10-10 14:16:32
tags:
- Android
- Java
categories:
- Android优化

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1510910024071&di=ca4239c8df6b302e2b358cfbdbc0f0c6&imgtype=0&src=http%3A%2F%2Fimgmini.dfshurufa.com%2Fmobile%2F20160312074356_5724841a4822976c779062ecb8ca1b8c_1.jpeg)


* [使用 Lint 改进您的代码](https://developer.android.google.cn/studio/write/lint.html)

* [使用注解改进代码检查](https://developer.android.google.cn/studio/write/annotations.html)

[android.support.annotation](https://developer.android.google.cn/reference/android/support/annotation/package-summary.html)

**资源类注解**:

| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 | e.g. |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- | :----- |
| AnimatorRes  | 👌 | 👌  | 👌 | ❌ | ❌ | 动画资源引用 | android.R.animator.fade_in |
| AnimRes  | 👌 | 👌  | 👌 | ❌ | ❌ | anim资源引用 | android.R.animator.fade_in |
| AnyRes | 👌 | 👌  | 👌 | ❌ | ❌ | 任何类型的资源引用 |  -|
| ArrayRes | 👌 | 👌  | 👌 | ❌ | ❌ | 数组资源引用 | android.R.array.phoneTypes |
| AttrRes | 👌 | 👌  | 👌 | ❌ | ❌ | 属性引用 |  android.R.attr.action |
| BoolRes | 👌 | 👌  | 👌 | ❌ | ❌ | 布尔资源引用 |  -|
| ColorRes | 👌 | 👌  | 👌 | ❌ | ❌ | 颜色资源引用 | android.R.color.black |
| DimenRes | 👌 | 👌  | 👌 | ❌ | ❌ | 尺寸资源引用 | android.R.dimen.app_icon_size |
| DrawableRes | 👌 | 👌  | 👌 | ❌ | ❌ | Drawable资源引用 | android.R.attr.alertDialogIcon |
| FontRes | 👌 | 👌  | 👌 | ❌ | ❌ | 字体资源引用 | R.font.myfont | 
| StringRes | 👌 | 👌  | 👌 | ❌ | ❌ | String资源引用 | android.R.string.ok | 
| StyleableRes | 👌 | 👌  | 👌 | ❌ | ❌ | styleable资源引用 | android.R.styleable.TextView_text | 
| StyleRes | 👌 | 👌  | 👌 | ❌ | ❌ | style资源引用 | android.R.style.TextAppearance | 
| TransitionRes | 👌 | 👌  | 👌 | ❌ | ❌ | transition资源引用 | -| 
| LayoutRes | 👌 | 👌  | 👌 | ❌ | ❌ | 布局资源引用 | android.R.layout.list_content | 
| MenuRes | 👌 | 👌  | 👌 | ❌ | ❌ | 菜单资源引用 | - | 
| NavigationRes | 👌 | 👌  | 👌 | ❌ | ❌ | 导航资源引用 | R.navigation.flow | 
| PluralsRes | 👌 | 👌  | 👌 | ❌ | ❌ | 复数的资源引用 | - | 
| RawRes | 👌 | 👌  | 👌 | ❌ | ❌ | raw资源引用 | - | 
| XmlRes | 👌 | 👌  | 👌 | ❌ | ❌ | XML资源引用 | - |


**特定类型值相关注解**：

| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- |
| ColorInt | 👌 | 👌  | 👌 | ❌ | ❌ | 填充色的int，AARRGGBB |
| ColorLong | 👌 | 👌  | 👌 | ❌ | ❌ | 填充[Color](https://developer.android.google.cn/reference/android/graphics/Color.html)的long |
| Dimension | 👌 | 👌  | 👌 | ❌ | ❌ | 表示一个尺寸 |
| Px | 👌 | 👌  | 👌 | ❌ | ❌ | 表示一个像素维度 | 

**线程相关注解**：

| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- |
| AnyThread | ❌ | ❌  | ❌ |👌 | 👌 | 表示可以从任何线程调用带注释的方法(例如，它是“线程安全的”)。如果带注释的元素是一个类，那么类中的所有方法都可以从任何线程调用。 |
| BinderThread | ❌ | ❌  | ❌ |👌 | 👌 | 在[Binder](http://blog.csdn.net/universus/article/details/6211589)线程上调用 |
| MainThread | ❌ | ❌  | ❌ |👌 | 👌 | 应该在主线程上调用 |
| UiThread | ❌ | ❌  | ❌ |👌 | 👌 | 应该在UI线程上调用 |
| WorkerThread | ❌ | ❌  | ❌ |👌 | 👌 | 应该只在工作线程上调用 |

```Java
@AnyThread
public void deliverResult(D data) { ... }
```

四个*线程相关注解*的**差异** 参考[这里](http://stackoverflow.com/questions/31892146/difference-between-mainthread-uithread-workerthread-binderthread-in-android-a)


**取值范围注解**：

| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- |
| FloatRange |  |   |  |  | ❌ | 浮点数或双精度数给定范围内 |
| IntRange |  |   |  |  | ❌ | int或long给定范围内 |

```Java
@FloatRange(from=0.0,to=1.0)
public float getAlpha() {
    ...
}
```

**空指针检查**:

| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- |
| NonNull | 👌 | 👌  | 👌 | ❌ | ❌ | 永远不能为空 | 
| Nullable | 👌 | 👌  | 👌 | ❌ | ❌ | 可以为null | 


**权限相关注解**：

| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- |
| RequiresPermission | ❌ | 👌  | ❌ |👌 | 👌 | 需要(或可能需要)一个或多个权限 |
| RequiresPermission.Read | ❌ | 👌  | ❌ |👌 | 👌 | 指定的权限是读操作所必需的 |
| RequiresPermission.Write | ❌ | 👌  | ❌ |👌 | 👌 | 指定的权限是写操作所必需的 |


**可见性相关注解**：

| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- |
| RequiresApi | ❌ | ❌  | ❌ |👌 | 👌 | 在给定的API级别或更高级别上调用 |
| RestrictTo | ❌ | ❌  | ❌ |👌 | 👌 | 应该只从一个特定的范围内访问(由[RestrictTo.Scope](https://developer.android.google.cn/reference/android/support/annotation/RestrictTo.Scope.html)定义) |
| VisibleForTesting | ❌ | 👌  | ❌ | 👌 | 👌 | 测试可见 |

**其他注解**：

| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- |
| CallSuper | ❌ | ❌  | ❌ | 👌 | ❌ | 表示任何复写方法都应该调用此方法 |
| Keep | ❌ | ❌  | ❌ |👌 | 👌 | 表示当代码在构建时被压缩时，不应该删除带注释的元素。这通常用于方法和类，这些方法和类只能通过反射来访问，因此编译器可能认为代码是未使用的。 |  

| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- |
| StringDef | 👌 | 👌  | 👌 | ❌ | ❌ | 表示一个逻辑类型，它的值应该是一个显式命名的常量 | 

```java
@Retention(SOURCE)
  @StringDef({
     POWER_SERVICE,
     WINDOW_SERVICE,
     LAYOUT_INFLATER_SERVICE
  })
  public @interface ServiceName {}
  public static final String POWER_SERVICE = "power";
  public static final String WINDOW_SERVICE = "window";
  public static final String LAYOUT_INFLATER_SERVICE = "layout_inflater";
  ...
  public abstract Object getSystemService(@ServiceName String name);
```

| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- |
| Size | 👌 | ❌  | 👌 | ❌ | ❌ | 复数的资源引用,用于数组或集合，指定大小或长度 | 

```java
public void getLocationInWindow(@Size(2) int[] location) {
      ...
}
```


| 注释  | 参数  | 字段 | 方法返回值 | 方法 | 类 |描述 |
|:------ |:---------:| :--------:| :--------:|:--------:| :-------:| :------- |
| CheckResult | ❌ | ❌  | 👌 | ❌ | ❌ | 表示带注释的方法返回一个结果,通常用于没有副作用的方法 |
| CheckResult(suggest) | ❌ | ❌  | ❌ | 👌 | ❌ | 建议的方法的名称 |

```Java
public @CheckResult String trim(String s) { return s.trim(); }
  ...
  s.trim(); // this is probably an error
  s = s.trim(); // ok
 
 
 // CheckResult(suggest) 
 @CheckResult(suggest="#redirectErrorStream(boolean)")
 public boolean redirectErrorStream() { ... }
```