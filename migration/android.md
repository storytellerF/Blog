# android migration

## :material3:1.1.0" 是出现了Duplicate class kotlin.collections.jdk8.CollectionsJDK8Kt found in modules

jetified-kotlin-stdlib-1.8.10 (org.jetbrains.kotlin:kotlin-stdlib:1.8.10) and 
jetified-kotlin-stdlib-jdk8-1.7.20 (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.7.20)

1.8.10 很有可能是material3 使用的。所以整个项目都需要升级成1.8.10

jitpack 编译之后引入测试一下很重要
