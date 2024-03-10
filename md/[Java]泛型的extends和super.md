# [Java]泛型的extends和super

java 中的泛型不支持型变，比如

```java
List<Object> objs = new List<String>();
```

>incompatible types: ArrayList&lt;String> cannot be converted to List&lt;Object>

如果需要进行型变，需要使用**extends** 和**super**。

```java
public class T1 {
    static class View {}
    static class ViewGroup extends View {}
    static class LinearLayout extends ViewGroup {}
    static class MyLayout extends LinearLayout {}
    static class FrameLayout extends ViewGroup {}
}
```

## extends

```java
import java.util.*;

public class T1 {
    public static void main(String[] args) {
        List<? extends ViewGroup> view1 = new ArrayList<FrameLayout>();// 此处的泛型可以是 ViewGroup LinearLayout MyLayout FrameLayout
        view1.add(null);
        view1.add(new Object());//compile error
        view1.add(new View());//compile error
        view1.add(new ViewGroup());//compile error
        view1.add(new LinearLayout());//compile error
        view1.add(new MyLayout());//compile error
        view1.add(new FrameLayout());//compile error
        ViewGroup v = view1.get(0);
    }
}

```

view1 的泛型为`<? extends ViewGroup>`，那么它可以接受的值可以是

```java
new XXXList<ViewGroup>();
new XXXList<LinearLayout>();//LinearLayout 继承自ViewGroup
new XXXList<MyLayout>();//MyLayout 继承自LinearLayout
new XXXList<FrameLayout>();//FrameLayout 继承自VewGroup
```

但是view1 调用`add`就会全部出错。宛如一个生产者，我们只能从中获取数据，增加数据由它生命的地方完成。

比如 `view1.add(new ViewGroup());`会出错，虽然泛型为 `<? extends ViewGroup>`， 但是**view1**的值是不能够确定的，因为它还可以是`new XXXList<LinearLayout>();`，`List<LinearyLayout>` 不能接受一个 **ViewGroup** 实例。

同样的，`view1.add(new LinearLayout());` 也不行，因为**view1** 的值也可能是 `new XXXList<MyLayout>();`，我们不能把LinearLayout 转成MyLayout，因为MyLayout 继承自LinearLayout，只有MyLayout 可以转成LinearLayout。

其他的同理。至于 `Object` 和 `View` 本来就不太行。

## super

```java
public class T1 {
    public static void main(String[] args) {
        List<? super ViewGroup> view2 = new ArrayList<ViewGroup>();//可以传递的泛型是 Object View ViewGroup
        view2.add(null);
        view2.add(new Object());//compile error
        view2.add(new View());//compile error
        view2.add(new ViewGroup());
        view2.add(new LinearLayout());
        view2.add(new MyLayout());
        view2.add(new FrameLayout());
        Object object = view2.get(0);
    }
}
```

view2的泛型为`<? super ViewGroup>`，那么它可以接受的值可以是

```java
new XXXList<Object>();
new XXXList<View>();
new XXXList<ViewGroup>();
```

现在我们调用 `view2.add(new Object());`，虽然view2可能是 `new XXXList<Object>();`，但它也可能是 `new XXXList<View>();`，所以不成功。同理，调用 `view2.add(new View());`，也不行。

但是`view2.add(new ViewGroup());` 和后面的都可以。因为，不管**view2**的值是上面3种中的哪一种值，都可以进行安全的转换即子类转换成父类。

比如我们现在调用 `add(new LinearLayout())`，

当 `view2 = new XXXList<Object>();` 没有问题，因为 `LinearLayout` 可以转换成 `Object`。

当 `view2 = new XXXList<View>();` 也没有问题，因为 `LinearLayout` 也可以转换成 `View`。

当 `view2=new XXXList<ViewGroup>();` 也没有问题，因为 `LinearLayout` 也可以转换成 `ViewGroup`。

但是 `add(new View())` 就不行了，`view2` 也可能是 `new XXXList<ViewGroup>();`。虽然在 `view2 = new XXXList<View>();` 时可行，但是view2也可能时 `new XXXList<ViewGroup>();`。所以这个可能会出现问题，不可行。

宛如一个消费者，企图让我们来生产数据，并且生产的数据的类型不能够“大于”指定的类型，那这个类型就相当于上届。。

## 使用

```java
public static void test1(List<? extends ViewGroup> v) {
    ViewGroup vg = v.get(0);
}

public static void test2(List<ViewGroup> v) {
    ViewGroup vg = v.get(0);
}
```

查看上面的代码，`view1` 几乎是“不可写”，不管你传递哪个都不行，它到底有什么用？它的用处主要在参数传递，比如上面的`test1(List<? extends ViewGroup> fruits)` ,我可以传递的值为

```java
new XXXList<ViewGroup>();
new XXXList<LinearLayout>();
new XXXList<MyLayout>();
new XXXList<FrameLayout>();
```

函数内部把所有的数据都当作ViewGroup 来处理。

但是，`test2(List<ViewGroup> v)`就只能够传递 `new XXXList<ViewGroup>()`。
