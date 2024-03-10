# Android 开发过程中发生的问题

## 关于资源

menu 资源文件中，创建的 `SearchView`可能会无法使用，我认为原因在`showAsAction` 和`actiovViewClass` ，他们的前缀是`app` ，但是你的项目如果是`androidx` 的，可能就会发生问题，需要改回`android` ，Android Studio 还会发生错误警告，不让我用这个前缀（😔）。通过软件内的反馈，后面具体的就他们解决了。

## 关于代码

1. 当时使用`SearchView` 时，其他博客里说的设置监听事件在`onPrepareOptionsMenu` ，可是这个函数会在每一次点击那三个点都会执行一次，感觉效率不好，这种监听事件一般不会发生改变，还是和创建menu的代码放到一起吧。不过关于设置menu 的每一个项的时候，比如是否启用，是否选中，就只能在`onPrepareOptionsMenu` 。

2. 读取sqlite 数据库，抛出这个异常`unknown error (code 14): Could not open database`。找了很多答案，都不行。在模拟器没有问题，放到小米（小米8 MIUI 9.11.14）上就不行了，突然想到我更改了目标版本，回退过去，没问题了。但是我需要知道这是小米的问题还是版本的问题，然后运行我的`API` 为29的虚拟机，没有问题（当前目标版本是28，因为回退了一次），然后再一次调整目标版本为29，依然没有问题，那应该就是小米的问题了，把`小米 29 sqlite` 的关键词放到搜索引擎上找不到什么有用的东西，所以这个问题我至今无法解决（尝试反馈给小米）。

3. TextView 设置文字大小（`setTextSize()`）无效。原因是在下面还调用了一个函数`setTextAppearance`，通过查看源代码，应该是在`applyTextAppearance` 函数中的

    ```java
        if (attributes.mTextSize != 0) {
            setRawTextSize(attributes.mTextSize, true /*    shouldRequestLayout */);
        }
    ```

    导致的，需要把设置大小放到这个函数的下面而不是上面。

4. 当我在广播接收器中打开应用时—通过`startActivity` ，除了第一次成功，其他全部启动失败，但是我的Pixel 虚拟机就没有问题，我的机器还是小米8😔。后续发现不止是在广播接收器中，只要不是在Activity 中启动都不行，日志显示

    >I/Timeline: Timeline: Activity_launch_request time:xxxxxxx

    感觉和小米的阻止链式调用有关，却不知道该如果处理😔。

    这个也找到办法了，给应用`后台弹出界面`权限即可。

5. 开发时做一个group notification，每一个通知有三个按钮，然后在接受Intent 传入的值时，发现`int` 返回的永远是`8`（或者只有键没有值），`String` 返回的时`null`，虚拟机也会出现问题。

    破案了，在请求`PendingIntent`获取广播时传入的`request code` 是0，通过传入不同的`request code` 就没有问题了。

6. 软件老是重复打印一条错误
    >W/System: A resource failed to call release.

    我觉得我所有的资源都手动关闭了，最后一步一步的测试，发现错误出现在`Cursor` 上，特别是在线程中执行，原本的代码是

    ```java
        Cursor cursor = sqLiteDatabase.rawQuery(xx4, new String[]{xx3});
        if (cursor != null && cursor.moveToFirst()) {
        do {
            String xx1 = cursor.getString(0);
            xx2.add(xx1);
        } while (cursor.moveToNext());
            cursor.close();
            return xx2;
        }
        return null;
    ```

    然后就用了`try finally` 处理关闭操作，这个提示就没有了。
    为了搞清为什么，添加`catch` 语句捕获异常，什么也没有。
7. 做一个无障碍应用，发现别的应用都没有问题，但是我却获得不到百词斩的事件，费解。
原来百词斩的包名是`com.jiongji.andriod.card` 而不是`com.jiongji.android.card`。更加费解了。
