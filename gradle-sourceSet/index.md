# \[Gradle] 编译任意地方的文件

我正在参加「掘金·启航计划」

>本人所有文件禁止任何形式的转载

gradle 对于参与编译的文件的位置没有限制，不必发布到maven repository 或者编译成aar、jar。

## SourceSet（添加代码）

在没有特殊处理的场景下，会通过我们添加的插件识别默认的sourceSet。

我们创建一个全新的kotlin 项目，查看他的sourceSet。

```kts
//build.gradle.kts
kotlin.sourceSets.forEach {
    println(it.name)
}
```

输出内容

> \> Configure project :
>
> main
>
> /Users/storytellerF/IdeaProjects/test/src/main/kotlin
>
> /Users/storytellerF/IdeaProjects/test/src/main/java
>
> test
>
> /Users/storytellerF/IdeaProjects/test/src/test/kotlin
>
> /Users/storytellerF/IdeaProjects/test/src/test/java

此方法用于导入**annotation processor**，**kotlin symbols processor** 生成的代码。曾经也用来复用**gradle plugin** 的代码，代码要放到buildSrc 用于调试，但是buildSrc 不能将gradle plugin 发布出去，所以还需要另一个module 用于发布，就将buildSrc 中的代码导入到了module 中，这种方法还导致一个警告⚠️，不过现在有了更好的办法， 后面会讲。

## include （添加模块）

一般来说我们添加一个module，会在settings.gradle 中添加这个module

```grovvy
include 'module'
```

添加的module 默认目录就是当前项目的根目录，不过我们可以覆盖掉指定一个新的路径，这个路径几乎可以是任意位置。

```kts
// include two projects, 'foo' and 'foo:bar'
// directories are inferred by replacing ':' with '/'
include 'foo:bar'

// include one project whose project dir does not match the logical project path
include 'baz'
project(':baz').projectDir = file('foo/baz')

// include many projects whose project dirs do not match the logical project paths
file('subprojects').eachDir { dir ->
    include dir.name
    project(":${dir.name}").projectDir = dir
}
```

我一般用这个方法导入本地项目或者submodule 的代码。本地项目没有发布，还有很多问题，发布出去在导入调试起来非常不便。或者是submodule，本身就不希望发布成一个库，用这种方式非常适合。并且**settings.gradle.kts**或者**settings.gradle** 应该是图灵完备的，支持条件判断，循环，读取外部数据。比如这样：

```groovy
def baoFolder = ext.baoFolder
def home = System.getProperty("user.home")
def debugBaoFolder
if (baoFolder == "local")
    debugBaoFolder = file("$home/AndroidStudioProjects/Bao/")
else
    debugBaoFolder = null
if (debugBaoFolder != null && debugBaoFolder.exists()) {
    def l = new String[]{
            "startup", "bao-library"
    }
    for (sub in l) {
        include("bao:$sub")
        project(":bao:$sub").projectDir = file("${debugBaoFolder.absolutePath}/$sub")
    }
}
```

**Bao** 是一个用于兜底thread exception的。通过上面的代码，就能够通过**gradle.properties** 决定是导入maven 还是导入本地代码。

## includeBuild （添加项目）

构建顺序早已其他模块，本质是另一个项目，可以文件系统中任意的位置。

可以通过设置gradle 的方式引入gradle。

```kts
pluginManagement {
    includeBuild("../../common-ui-list/version-manager")
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
```

上面的`includeBuild` 传入的参数是相对路径，同样的路径也被导入到了另一个项目，然后在当前项目（不是上述的version-manager）中当作一个gradle plugin引入。

```kts
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("kotlin-parcelize")
    id("androidx.navigation.safeargs.kotlin")
    id("com.storyteller_f.version_manager")
}
```

和你猜测的一样，通过includeBuild 引入的项目只能在**build.gradle** 中使用。如果想要通过**implementation** 引入，需要知道这个插件的classpath，上面我们指定的**com.storyteller\_f.version\_manager** 其实是这个插件的**pluginId**。我现在还不知道应该如何获取这个classpath。
