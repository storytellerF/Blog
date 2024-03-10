# [Java]泛型的extends和super

## 代码

```java
import java.util.*;

public class T1 {
    static class View {}
    static class ViewGroup extends View {}
    static class LinearLayout extends ViewGroup {}
    static class MyLayout extends LinearLayout {}
    static class FrameLayout extends ViewGroup {}
    public static void main(String[] args) {
        List<? extends ViewGroup> view1 = new ArrayList<FrameLayout>();// 可以传递的泛型是 ViewGroup LinearLayout MyLayout FrameLayout
        view1.add(null);
        view1.add(new Object());//compile error
        view1.add(new View());//compile error
        view1.add(new ViewGroup());//compile error
        view1.add(new LinearLayout());//compile error
        view1.add(new MyLayout());//compile error
        view1.add(new FrameLayout());//compile error
        ViewGroup v = view1.get(0);

        //下界
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

    public static void test1(List<? extends ViewGroup> v) {
        ViewGroup vg = v.get(0);
    }

    public static void test2(List<ViewGroup> v) {
        ViewGroup vg = v.get(0);
    }

    public static void test3(ViewGroup vg) {

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

但是view1 调用`add`就会全部出错。

比如`view1.add(new ViewGroup());`，虽然泛型为`<? extends ViewGroup>`,但是view1的值是不能够确定，因为它还可以是`new XXXList<LinearLayout>();`,父类不能够转换成子类，只能子类转换成父类，同样的，传递LinearLayout 也不行，因为view1 的值也可能是`new XXXList<MyLayout>();`，其他的同理。至于`Object`和`View` 本来就不太行。

view2的泛型为`<? super ViewGroup>`，那么它可以接受的值可以是

```java
new XXXList<Object>();
new XXXList<View>();
new XXXList<ViewGroup>();
```

现在我们调用`view2.add(new Object());`，虽然view2可能是`new XXXList<Object>();`，但它也可能是`new XXXList<View>();`，所以不成功。同理，调用`view2.add(new View());`，也不行。

但是`view2.add(new ViewGroup());`和后面的都可以。因为，不管view2的值是上面3种中的哪一种值，都可以进行安全的转换即子类转换成父类。

比如传递`LinearLayout`，`view2=new XXXList<Object>();`没有问题，因为`LinearLayout`可以转换成`Object`,`view2=new XXXList<View>();`也没有问题，因为`LinearLayout`也可以转换成`View`。假如`view2=new XXXList<ViewGroup>();`也没有问题，因为`LinearLayout`也可以转换成`ViewGroup`。

但是传递`View`就不行了，`view2`也可能是`new XXXList<ViewGroup>();`。虽然在`view2=new XXXList<View>();`时可行，但是view2也可能时`new XXXList<ViewGroup>();`。所以这个可能会出现问题，不可行。

## 使用

查看上面的代码，`view1`几乎是“不可写”，不管你传递哪个都不行，它到底有什么用？它的用处主要在参数传递，比如上面的`test1(List<? extends ViewGroup> fruits)` ,我可以传递的值为

```java
new XXXList<ViewGroup>();
new XXXList<LinearLayout>();
new XXXList<MyLayout>();
new XXXList<FrameLayout>();
```

但是，`test2(List<ViewGroup> v)`就只能够传递`new XXXList<ViewGroup>()`。
