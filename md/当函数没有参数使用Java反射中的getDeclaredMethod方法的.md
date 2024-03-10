# 当函数没有参数使用Java反射中的getDeclaredMethod方法的参数parameterTypes如何传递

## 问题

在反射中，当使用`getDeclaredMethod` 方法，本来是想要传递一个`null`的，但是eclipse显示一个警告，但是这个没有参数，我无法传递一个准确的类型，但我非常不喜欢看到警告，所以要想办法解决。

## 步骤

1. 查看文档

    ```java
    /**
        * Returns a {@code Method} object that reflects the specified
        * declared method of the class or interface represented by this
        * {@code Class} object. The {@code name} parameter is a
        * {@code String} that specifies the simple name of the desired
        * method, and the {@code parameterTypes} parameter is an array of
        * {@code Class} objects that identify the method's formal parameter
        * types, in declared order.  If more than one method with the same
        * parameter types is declared in a class, and one of these methods has a
        * return type that is more specific than any of the others, that method is
        * returned; otherwise one of the methods is chosen arbitrarily.  If the
        * name is "&lt;init&gt;"or "&lt;clinit&gt;" a {@code NoSuchMethodException}
        * is raised.
        *
        * <p> If this {@code Class} object represents an array type, then this
        * method does not find the {@code clone()} method.
        *
        * @param name the name of the method
        * @param parameterTypes the parameter array
        * @return  the {@code Method} object for the method of this class
        *          matching the specified name and parameters
        * @throws  NoSuchMethodException if a matching method is not found.
        * @throws  NullPointerException if {@code name} is {@code null}
        * @throws  SecurityException
        *          If a security manager, <i>s</i>, is present and any of the
        *          following conditions is met:
        *
        *          <ul>
        *
        *          <li> the caller's class loader is not the same as the
        *          class loader of this class and invocation of
        *          {@link SecurityManager#checkPermission
        *          s.checkPermission} method with
        *          {@code RuntimePermission("accessDeclaredMembers")}
        *          denies access to the declared method
        *
        *          <li> the caller's class loader is not the same as or an
        *          ancestor of the class loader for the current class and
        *          invocation of {@link SecurityManager#checkPackageAccess
        *          s.checkPackageAccess()} denies access to the package
        *          of this class
        *
        *          </ul>
        *
        * @jls 8.2 Class Members
        * @jls 8.4 Method Declarations
        * @since 1.1
        */
    ```

    说是需要一个标示方法的正规的（formal 有点不太好翻译）参数类型的Class 对象的数组。不过我们没有参数，那就传递一个长度为0的数组吧，比如这样`new Class[0]`，错误消失了，不过为了以后还有警告可以写成`new Class<?>[0]`（比如Android Studio）。

2. 然后是警告本身的内容

    >Type null of the last argument to method getDeclaredMethod(String, Class<?>...) doesn't exactly match the vararg parameter type. Cast to Class<?>[] to confirm the non-varargs invocation, or pass individual arguments of type Class<?> for a varargs invocation.

    最后说是可以传递一个单独的`Class<?>` 类型的参数，因为没有参数还是不好处理，尝试输入一个void，获得了`Void.class` 或者 `Void.TYPE`，警告消除了，但是运行时就出现了错误，说是找不到参数类型为void的此方法。所以这个方法应该不行。

3. 然后就是调用这个通过反射得到的方法了，尝试传递一个null，显示的错误信息为

    >Type null of the last argument to method invoke(Object, Object...) doesn't exactly match the vararg parameter type. Cast to Object[] to confirm the non-varargs invocation, or pass individual arguments of type Object for a varargs invocation.

    可以传递参数`new Object[0]` 。
