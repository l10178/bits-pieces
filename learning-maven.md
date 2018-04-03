 # Maven

 在多模块的工程中，如果模块之间存在依赖关系，那模块的编译必须要有顺序的要求。例如：P(parent)中包含A模块和B模块，且A模块依赖于B模块，那么在P中的pom,xml中需申明为：
```
   <modules>
        <module>B</module>
        <module>A</module>
    </modules>
```
B需要声明在A的前面，这样先编译后的内容才能被A依赖。

auto run findbugs,how to.
