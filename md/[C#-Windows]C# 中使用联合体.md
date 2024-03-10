# [C#-Windows]解决 C# 中使用联合体的问题（因为它在0偏移位置处包含一个对象字段...）

## 问题

开发的程序会使用到Windows api，因为在C 中开发图形界面程序有点麻烦，所以改为了C# ，但是这个api 里有一个数据是以联合体存储的，C# 的联合体又有一些限制。

```C#
public struct Anonymous_d34c77ee_53b2_47a5_a97c_d5c2b29c8ee3 {

    /// RAWMOUSE->tagRAWMOUSE
    [System.Runtime.InteropServices.FieldOffset(0)]
    public TagRAWMOUSE mouse;

    /// RAWKEYBOARD->tagRAWKEYBOARD
    [System.Runtime.InteropServices.FieldOffset(0)]
    public TagRAWKEYBOARD keyboard;

    /// RAWHID->tagRAWHID
    [System.Runtime.InteropServices.FieldOffset(0)]
    public TagRAWHID hid;
}

```

产生的错误的信息是

>...因为他在0偏移位置处包含一个对象字段，该字段由一个非对象字段不正确地对齐或重叠。

经过测试，联合体如果都是一些简单类型，比如整形之类的，就没有问题。如果是这种复杂类型，就不行。

## 解决

耽搁了很长时间，一直想不到解决办法，有一天，突然开窍了，我把整个结构体拆成三个正常的、没有联合体地结构体不就行了，反正是操作那一片地址，如何解释还不是我说了算吗。

```c#
//原本长这样
public struct TagRAWINPUT {

    /// RAWINPUTHEADER->tagRAWINPUTHEADER
    public TagRAWINPUTHEADER header;

    /// Anonymous_d34c77ee_53b2_47a5_a97c_d5c2b29c8ee3
    public Anonymous_d34c77ee_53b2_47a5_a97c_d5c2b29c8ee3 data;
}
//改成这样
public struct TagRAWINPUTONE {
    public TagRAWINPUTHEADER header;
}
```

仅通过后面的`header`就能够了解后面的联合体具体存储地是哪一个数据，是mouse，还是keyboard，还是hid。

```C#
//原本
TagRAWINPUT rawInput = Marshal.PtrToStructure<TagRAWINPUT>(dataBuffer);
//新的
TagRAWINPUTONE rawInputOne = Marshal.PtrToStructure<TagRAWINPUTONE>(dataBuffer);
```

因为`TagRAWINPUTONE` 没有联合体，所以这段代码执行没有任何问题。根据`header` 判断联合体的类型，然后在通过`Marshal` 获取真正的结构体。

比如这个联合体存储的是`TagRAWMOUSE` ，那就把结果转换成`TagRAWINPUT1`。

```c#
//另外两个以此类推
public struct TagRAWINPUT1 {

    /// RAWINPUTHEADER->tagRAWINPUTHEADER
    public TagRAWINPUTHEADER header;

    /// Anonymous_d34c77ee_53b2_47a5_a97c_d5c2b29c8ee3
    public TagRAWMOUSE mouse;
}
```

```c#
TagRAWINPUT1 raw = Marshal.PtrToStructure<TagRAWINPUT1>(dataBuffer);
```
