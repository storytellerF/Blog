# [Android] 小白条（导航栏）沉浸

## 前言

> 本人所有文章禁止任何形式的转载，谢谢

我们主要做的是小白条的沉浸，而不是安卓口中的沉浸。安卓口中的沉浸主要是对于阅读、游戏等场景下隐藏状态栏和导航栏。而我们的目标仅仅是使导航栏背景透明，并且能够显示导航栏下面的内容，IOS 那样的效果。

这在安卓中其实也是有的，我们可以看看安卓的Settings

<img src=content_2022-04-26-10-30-04.png width=50%></img>

效果很不错。其实也就是说凭借安卓本身就可以做到。差一点就要用自定义view 替换状态栏和导航栏了。其实使用自定义view 也是有好处的，就是不用关心怎么适配安卓，但是做出真正能用的，想必肯定要花不少时间，所以还是这种方法更经济一点。

## 实现

1. 这是什么处理都没有的。

    <img src=content_2022-04-26-10-30-39.png width=50%></img>

2. 首先第一步就是让内容可以扩展到下面。你很可能了解到的一个方法就是

    ```xml
    <item name="android:windowTranslucentNavigation">true</item>
    ```

    <img src=content_2022-04-26-10-31-00.png width=50%></img>

    确实扩展到底部了，但是蒙了一层灰色。且无法去掉，即使为其设置透明色。

3. 所以上面的方法应该弃用。考虑用新的方法设置“全屏”

    ```kotlin
    window.setDecorFitsSystemWindows(false)
    ```

    <img src=content_2022-04-26-10-31-19.png width=50%></img>

    但是这时候你的Android studio 就开始出现红色提示，告诉你这段代码有兼容问题。安卓提供了一系列的兼容库，一般就是原来的库后面加上Compat。所以

    ```kotlin
    WindowCompat.setDecorFitsSystemWindows(window, false)
    ```

    完美解决问题。

4. 现在考虑如何修改颜色。如果要修改颜色，需要添加

    ```kotlin
    window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS)
    ```

    然后是真正的修改颜色

    ```kotlin
    window.navigationBarColor = Color.TRANSPARENT
    ```

    关于这个navigationBarColor 的颜色，如果仅仅是小白条，直接使用透明色即可，但是如果是“三大金刚”按钮，需要使用一个透明度高的颜色，而不能直接使用透明色。

    <img src=content_2022-04-26-10-31-42.png width=50%></img>

    但是底部的小白条变成“不太显眼”了。如果你有把它变成白色的需求，应该是无法完成的。不管是使用

    ```kotlin
    WindowInsetsControllerCompat(window, window.decorView).isAppearanceLightNavigationBars = true
    ```

    还是

    ```kotlin
    window.isNavigationBarContrastEnforced = false
    ```

    都不行。虽然他们java doc看起来他们应该是可以的。不过，如果把整个页面变成黑色，这个小白条就变成白色了，也就是说小白条的颜色完全由系统处理，没有给开发人员多余的操作空间。

    虽然对于小白条来说没有任何用处，但是对经典3 键导航还是有效。

5. 现在的问题是顶部的内容被截了一部分。我们要的只是底部沉浸。通过layout inspector 发现

    内容上面顶了一个actionbar 的高度，actionbar 上面顶了一个通知栏的高度。挺诡异。只好把actionbar 去掉，使用toolbar。

    <img src=content_2022-04-26-10-32-09.png width=50%></img>

    我们可以使用

    ```kotlin
    findViewById<ConstraintLayout>(R.id.contentView).setOnApplyWindowInsetsListener { v, insets -> 
        val top = WindowInsetsCompat.toWindowInsetsCompat(insets, v).getInsets(WindowInsetsCompat.Type.statusBars()).top
        v.updatePadding(top = top)
        insets
    }
    ```

    使用什么view 来设置这个`setOnApplyWindowInsetsListener` 都可以，但是decorView 不行。 主要问题是这个设置的方法和我们设置`OnClickListener` 类似，一旦我们自定义之后，View 本身的`OnApplyWindowInsets` 就不在执行。

    <img src=content_2022-04-26-10-32-31.png width=50%></img>

## 参考链接

[通过自定义view的方式设置沉浸 Android 系统 Bar 沉浸式完美兼容方案](https://juejin.cn/post/7075578574362640421)