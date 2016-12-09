# proguard-rules
Android Studio自身集成Java语言的ProGuard作为压缩，优化和混淆工具，配合Gradle构建工具使用很简单，只需要在工程应用目录的gradle文件中设置minifyEnabled为true即可。然后我们就可以到proguard-rules.pro文件中加入我们的混淆规则了。

```
android {
    ...
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
````
以上示例代码表示对release版本就行混淆处理。下面我们先来简介下ProGuard的三大作用，并简要说明下它们常用的命令。

> ProGuard作用

压缩（Shrinking）：默认开启，用以减小应用体积，移除未被使用的类和成员，并且会在优化动作执行之后再次执行（因为优化后可能会再次暴露一些未被使用的类和成员）。
```
-dontshrink 关闭压缩
```
优化（Optimization）：默认开启，在字节码级别执行优化，让应用运行的更快

```
-dontoptimize  关闭优化
-optimizationpasses n 表示proguard对代码进行迭代优化的次数，Android一般为5
```

混淆（Obfuscation）：默认开启，增大反编译难度，类和类成员会被随机命名，除非用keep保护。
```
-dontobfuscate 关闭混淆
```

混淆后默认会在工程目录app/build/outputs/mapping/release下生成一个mapping.txt文件，这就是混淆规则，我们可以根据这个文件把混淆后的代码反推回源本的代码，所以这个文件很重要，注意保护好。原则上，代码混淆后越乱越无规律越好，但有些地方我们是要避免混淆的，否则程序运行就会出错，所以就有了下面我们要教大家的，如何让自己的部分代码避免混淆从而防止出错。

>基本规则

先看如下两个比较常用的命令，很多童鞋可能会比较迷惑以下两者的区别。
```
-keep class cn.hadcn.test.**
-keep class cn.hadcn.test.*
```
一颗星表示只是保持该包下的类名，而子包下的类名还是会被混淆；两颗星表示把本包和所含子包下的类名都保持；用以上方法保持类后，你会发现类名虽然未混淆，但里面的具体方法和变量命名还是变了，这时如果既想保持类名，又想保持里面的内容不被混淆，我们就需要以下方法了。
```
-keep class cn.hadcn.test.* {*;}
```
在此基础上，我们也可以使用Java的基本规则来保护特定类不被混淆，比如我们可以用extend，implement等这些Java规则。如下例子就避免所有继承Activity的类被混淆.

```
-keep public class * extends android.app.Activity
```

如果我们要保留一个类中的内部类不被混淆则需要用$符号，如下例子表示保持ScriptFragment内部类JavaScriptInterface中的所有public内容不被混淆。

```
-keepclassmembers class cc.ninty.chat.ui.fragment.ScriptFragment$JavaScriptInterface {
   public *;
}
```
再者，如果一个类中你希望保持全部内容被混淆，而只是希望保护类下的特定内容，就可以使用
```
<init>;     //匹配所有构造器
<fields>;   //匹配所有域
<methods>;  //匹配所有方法方法
```
你还可以在<fields>或<methods>前面加上private 、public、native等来进一步指定不被混淆的内容，如
```
-keep class cn.hadcn.test.One {
    public <methods>;
}
```
表示One类下的所有public方法都不会被混淆，当然你还可以加入参数，比如以下表示用JSONObject作为入参的构造函数不会被混淆

```
-keep class cn.hadcn.test.One {
   public <init>(org.json.JSONObject);
}
```
