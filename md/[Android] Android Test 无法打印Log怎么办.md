# [Android] 使用Android Test 进行测试却无法打印Log怎么办

## 问题

在Android Test中进行测试还是很方便的，但有时也会不太方便。比如我有很多数据，检测一个函数是否工作正常，放到一个数组中遍历测试的，当出现问题时，我不知道是哪个数据处理出现了问题。

如果你想要使用Log的功能，发现毫无用处，找不到你的日志在哪里。
这个日志应该不是由你当前的软件打印的，而是包名为`your_original_package_name.test` 的软件打印的，可能跟这个有关吧。反正就是`Android Studio` 没有展示日志。

## 思考

虽然日志无法打印，但是出现错误时还是会有提示的，长得好像`Exception`（其实不是，后面会说）。

既然是个“异常”，那就捕获它吧。然后换成自己的异常，在这个异常中添加一些调试信息，甚至自己做一个异常类不就好了。

```java
    try {
        //...
    } catch (Exception e) {
        //...
    }

```

但是这样其实不能够捕获异常，仔细看那个错误，是`java.lang.AssertionError`，是`Error` 而不是`Exception`。

谷歌的文档写的是

>An Error is a subclass of Throwable that indicates serious problems that a reasonable application should not try to catch. Most such errors are abnormal conditions. The ThreadDeath error, though a "normal" condition, is also a subclass of Error because most applications should not try to catch it.

`Error` 是一个`Throwable` 的子类，标识一个严重错误，一个`resonable application` 不应该尝试去捕获它。
>A method is not required to declare in its throws clause any subclasses of Error that might be thrown during the execution of the method but not caught, since these errors are abnormal conditions that should never occur. That is, Error and its subclasses are regarded as unchecked exceptions for the purposes of compile-time checking of exceptions.

上面说一个应用不应该捕获`Error` 但是我不管，我就要这么干。反正运行这段代码的也不是一个`resonable application`。

```java
try {
    assertFalse(...);
} catch (AssertionError ignored) {
    throw new AssertionError("hello world");
}
```

这样就可以正常调试了。
