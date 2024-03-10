# 【android】让Android 的EditText 显示行号（并解决行号与内容不对齐的问题）

## 引言

如果想要在Android 上完成一个带有代码编写功能的EditText，那就要有高亮和行号，在这里讲解如何绘制出行号。

## 步骤

1. 新建一个项目（这些步骤太基本了，不再附图），新建一个类继承自EditText。然后实现它的方法。

    ```java
        public class MyEditText extends EditText {
            public MyEditText(Context context) {
                super(context);
            }

        public MyEditText(Context context, AttributeSet attrs) {
            super(context, attrs);
        }
    }
    ```

    这是最基本的代码，就在这个基础上完成。
2. 添加一个函数用来写初始化的内容, 在构造函数中调用。

    ```java
        private void init() {
            setPadding(100,getPaddingTop(),getPaddingRight(),getPaddingBottom());
        }
    ```

    需要空出一部分来绘制行号，添加边距来让出这个空间。
    只需要设置左边距，其他的用默认的。并且这个值应该根据某些情况改变。

3. 重写 `onDraw` 方法。

    ```java
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        Paint lineNumberPaint=getPaint();
        int lineHeight = getLineHeight();
        for (int i=0;i<getLineCount();i++){
            int i1 = i + 1;
            canvas.drawText(i1 +"",0,(i1* lineHeight)+getPaddingTop(),lineNumberPaint);
        }
    }
    ```

    也可以设置一个基础值，然后在循环中不断增加而不是用乘法。
4. 选择 `build` 下面的 `Make Project` 或者快捷键`Ctrl+F9`来构建项目，这样就能够在视图上添加我们的自定义`view` 了。搜索我们的`MyEditText` ，然后添加到视图上。

    ![选择我们写的MyEditText](https://upload-images.jianshu.io/upload_images/10647432-c1d4678ad46d18b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5. 现在需要对`init` 函数继续操作。
    设置重心。
    ```setGravity(Gravity.TOP|Gravity.START);```

    ```java
        private void init() {
            setPadding(100,getPaddingTop(),getPaddingRight(),getPaddingBottom());
            setGravity(Gravity.TOP|Gravity.START);
            // setText("te\na\na\na\na\na\na\nte\na\na\na\na\n"+
            // "a\na\nte\na\na\na\na\na\na\nte\na\na\na\na\na\na\n" +
            // "te\na\na\na\na\na\na\nte\na\na\na\na\na\na\nte\n"+
        //  "a\na\na\na\na\na\nte\na\na\na\na\na\na\n" +
        // "te\na\na\na\na\na\na\nte\na\na\na\na\na\na\nte\n"+
        // "a\na\na\na\na\na\nte\na\na\na\na\na\na\n" +
            //"te\na\na\na\na\na\na\nte\na\na\na\na\na\na\nte"+
        // "\na\na\na\na\na\na\nte\na\na\na\na\na\na\n");  这里的代码不再使用
        }
    ```

    增加的那些奇怪的部分是用来测试用的。
    `setPadding` 用来空出地方用来绘制行号。

6. 运行到虚拟机中效果挺好。

    ![Android 虚拟机的截图](https://upload-images.jianshu.io/upload_images/10647432-55ffe29566847c6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 问题

把这个程序放到真机上运行便不行了—— 行号和内容的行对不齐。
通过读开源项目和源码，发现一个`getLineTop` 和 `getLineBottom` 函数，控制行号位置不再通过乘法或者加法。

```java
  /**
    * Return the vertical position of the top of the specified line
    * (0&hellip;getLineCount()).
    * 返回指定行的顶部的垂直位置
    * If the specified line is equal to the line count, returns the
    * bottom of the last line.
    * 如果指定的行等于行号，返回最后一行的底部
    */
```

```java
   /**
     * Return the vertical position of the bottom of the specified line.
     * 返回指定行的底部的垂直位置
     */
    public final int getLineBottom(int line) {
        return getLineTop(line + 1);
    }
```

```java
    private int getCurrentLine() {//获取光标所在行
        int selectionStart = getSelectionStart();
        Layout layout = getLayout();
        if (selectionStart != -1 && layout != null) {
            return layout.getLineForOffset(selectionStart);
        }
        return -1;
    }
    Paint lineHeightPaint = new Paint();
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        lineHeightPaint.setStyle(Paint.Style.FILL_AND_STROKE);
        lineHeightPaint.setColor(Color.argb(100, 90, 164, 161));
        int currentCursorPositionLine = getCurrentLine();
        int getCurrentTop = getLayout().getLineTop(currentCursorPositionLine);//如果是0 获取第0 行的顶部
        int getCurrentBottom = getLayout().getLineBottom(currentCursorPositionLine);
        int paddingTop = getPaddingTop();
        int lineCount = getLineCount();
        canvas.drawRect(0, getCurrentTop+paddingTop, getWidth(), getCurrentBottom+paddingTop, lineHeightPaint);
        Paint lineNumberPaint=getPaint();
        for (int i=0;i<lineCount;i++){
            int lineBottom = getLayout().getLineBottom(i);
            //这里的y 是基线，需要得到这个基线
            canvas.drawText(Integer.toString(i+1), 0, lineBottom-lineNumberPaint.descent()+paddingTop, lineNumberPaint);
        }
    }
```

2019/9/28 更新了代码，虽然之前完成了功能，但是存在问题，修改问题，并力求直观。

现在点击`EditText` 看高亮的行和行号是不是对齐的了。

初此之外，android 还提供了`getBaseline` 函数，返回的值是`top boundary` 到基线的位置。

```java
  /**
   * <p>Return the offset of the widget's text baseline from the widget's top
   * boundary. If this widget does not support baseline alignment, this
   * method returns -1. </p>
   *
   * @return the offset of the baseline within the widget's bounds or -1
   *         if baseline alignment is not supported
   */
```
