# javac编译选项
## 前言
javac有很多选项,在jdk1.8中,通过javac -help 可以看到如下信息的输出：  
```
javac
用法: javac <options> <source files>
其中, 可能的选项包括:
  -g                         生成所有调试信息
  -g:none                    不生成任何调试信息
  -g:{lines,vars,source}     只生成某些调试信息
  -nowarn                    不生成任何警告
  -verbose                   输出有关编译器正在执行的操作的消息
  -deprecation               输出使用已过时的 API 的源位置
  -classpath <路径>            指定查找用户类文件和注释处理程序的位置
  -cp <路径>                   指定查找用户类文件和注释处理程序的位置
  -sourcepath <路径>           指定查找输入源文件的位置
  -bootclasspath <路径>        覆盖引导类文件的位置
  -extdirs <目录>              覆盖所安装扩展的位置
  -endorseddirs <目录>         覆盖签名的标准路径的位置
  -proc:{none,only}          控制是否执行注释处理和/或编译。
  -processor <class1>[,<class2>,<class3>...] 要运行的注释处理程序的名称; 绕过默认的搜索进程
  -processorpath <路径>        指定查找注释处理程序的位置
  -parameters                生成元数据以用于方法参数的反射
  -d <目录>                    指定放置生成的类文件的位置
  -s <目录>                    指定放置生成的源文件的位置
  -h <目录>                    指定放置生成的本机标头文件的位置
  -implicit:{none,class}     指定是否为隐式引用文件生成类文件
  -encoding <编码>             指定源文件使用的字符编码
  -source <发行版>              提供与指定发行版的源兼容性
  -target <发行版>              生成特定 VM 版本的类文件
  -profile <配置文件>            请确保使用的 API 在指定的配置文件中可用
  -version                   版本信息
  -help                      输出标准选项的提要
  -A关键字[=值]                  传递给注释处理程序的选项
  -X                         输出非标准选项的提要
  -J<标记>                     直接将 <标记> 传递给运行时系统
  -Werror                    出现警告时终止编译
  @<文件名>                     从文件读取选项和文件名
```

关于这个option所对应的类就是Option.接下来我们就来看一下这个类。

## 解析
Option类是一个枚举,代表javac的选项.处理命令行选项的特定选项是通过按顺序搜索此枚举的成员来标识的,找到第一个匹配的。   
其中,Option又分为OptionKind,OptionGroup,ChoiceKind不同的类型.这三种也是枚举,定义如下：  

### OptionKind
该类代表命令行参数的类型,通过-help 和 -X来使用的.  
```java
public enum OptionKind {
    // 标准的选项,被-help所文档化
    STANDARD,
    // 扩展的选项,被-X所文档化
    EXTENDED,
    // 隐藏的选项,没有被文档化
    HIDDEN,
}
```
查看扩展的选项：   

```
javac -X
  -Xlint                     启用建议的警告
  -Xlint:{all,auxiliaryclass,cast,classfile,deprecation,dep-ann,divzero,empty,fallthrough,finally,options,overloads,overrides,path,processing,rawtypes,serial,static,try,unchecked,varargs,-auxiliaryclass,-cast,-classfile,-deprecation,-dep-ann,-divzero,-empty,-fallthrough,-finally,-options,-overloads,-overrides,-path,-processing,-rawtypes,-serial,-static,-try,-unchecked,-varargs,none} 启用或禁用特定的警告
  -Xdoclint                  为 javadoc 注释中的问题启用建议的检查
  -Xdoclint:(all|none|[-]<group>)[/<access>] 
        为 javadoc 注释中的问题启用或禁用特定检查,
        其中 <group> 为 accessibility, html, missing, reference 或 syntax 之一。
        <access> 为 public, protected, package 或 private 之一。
  -Xbootclasspath/p:<路径>     置于引导类路径之前
  -Xbootclasspath/a:<路径>     置于引导类路径之后
  -Xbootclasspath:<路径>       覆盖引导类文件的位置
  -Djava.ext.dirs=<目录>       覆盖所安装扩展的位置
  -Djava.endorsed.dirs=<目录>  覆盖签名的标准路径的位置
  -Xmaxerrs <编号>             设置要输出的错误的最大数目
  -Xmaxwarns <编号>            设置要输出的警告的最大数目
  -Xstdout <文件名>             重定向标准输出
  -Xprint                    输出指定类型的文本表示
  -XprintRounds              输出有关注释处理循环的信息
  -XprintProcessorInfo       输出有关请求处理程序处理哪些注释的信息
  -Xprefer:{source,newer}    指定读取文件, 当同时找到隐式编译类的源文件和类文件时
  -Xpkginfo:{always,legacy,nonempty} 指定 package-info 文件的处理
  -Xplugin:"名称参数"            要运行的插件的名称和可选参数
  -Xdiags:{compact,verbose}  选择诊断模式

这些选项都是非标准选项, 如有更改, 恕不另行通知。
```


### OptionGroup
OptionGroup --> option的组别,这决定了该option在什么情况下是可被接受的.  
```java
enum OptionGroup {
    // 基本的option,可以通过命令行或者编译器api来使用
    BASIC,
	 // 一个javac 标准的JavaFileManager的选项,其他的file manager可能不支持这个选项
    FILEMANAGER,
   	// 一个请求信息的命令行选项,比如 -help
    INFO,
    // 一个代表文件或者类名的命令行选项
    OPERAND
}
```

### ChoiceKind
ChoiceKind --> 关于 选择的类型.  
举例说明如下:  

-Xpkginfo: 这个编译选项是处理package-info 文件的,该选项的值是{always,legacy,nonempty} 中的任意一个,此时ChoiceKind为ONEOF.  

-g: 这个编译选项是生成某些调试信息(lines,vars,source). 这个选项只能是(lines,vars,source),不能是其他的值.如果传入的参数值为 -g:vars,-g:xx,则最终不会进行编译   

关于这两个选项的区别,可以通过查看源码的方式来观察:  

```java
	// com.sun.tools.javac.main.Option.matches(String)
    public boolean matches(String option) {
        if (!hasSuffix)
            return option.equals(text);

        if (!option.startsWith(text))
            return false;

        if (choices != null) {
            String arg = option.substring(text.length());
            if (choiceKind == ChoiceKind.ONEOF)
                return choices.keySet().contains(arg);
            else {
                for (String a: arg.split(",+")) {
                    if (!choices.keySet().contains(a))
                        return false;
                }
            }
        }

        return true;
    }
```

可以看到,如果choiceKind为ONEOF,则只要在选项集中有对应的值即可,而ANYOF,则要求选项值需要在选项集中都存在.  

## 总结
现通过表格的方式,将Option的所有选项列出:  

|      选项值	       |          argsNameKey           |           	descrKey            |   	类型   |    	组别     | 	选择类型 |       	选项值集       | 	是否有后缀 |                           	作用                           |
| :------------------: | :----------------------------: | ------------------------------ | --------- | ----------- | --------- | ----------------------- | ------------- | ------------------------------------------------------------ |
|          -g          |            	null            | 	javac.opt.g                  | 	STANDARD | 	BASIC      | 	null   | 	null                  | 	false     | 	生成所有调试信息                                           |
|       -g:none        |           	null	            | javac.opt.g.none               | 	STANDARD | 	BASIC      | 	null   | 	null	              | false         | 	不生成任何调试信息                                         |
|         -g:          |           	null	            | javac.opt.g.lines.vars.source	 | STANDARD  | 	BASIC	   | ANYOF	   | {lines,vars,source}	  | true	      | 只生成某些调试信息                                             |
|       -Xlint:        |              null              | javac.opt.Xlint.suboptlist     | EXTENDED  | BASIC       | ANYOF     | LintCategory对应的枚举值 | true          | 启用或禁用特定的警告                                           |
|      -Xdoclint:      |              null              | javac.opt.Xdoclint             | EXTENDED  | BASIC       | null      | null                    | false         | 为javadoc注释中的问题启用建议的检查                             |
|      -Xdoclint:      |   javac.opt.Xdoclint.subopts   | javac.opt.Xdoclint.custom      | EXTENDED  | BASIC       | null      | null                    | false         | 为 javadoc 注释中的问题启用或禁用特定检查                        |
|       -nowarn        |              null              | javac.opt.nowarn               | STANDARD  | BASIC       | null      | null                    | false         | 不生成任何警告(保留命令行向后兼容性)                            |
|       -verbose       |              null              | javac.opt.verbose              | STANDARD  | BASIC       | null      | null                    | false         | 输出有关编译器正在执行的操作的消息                              |
|     -deprecation     |              null              | javac.opt.deprecation          | STANDARD  | BASIC       | null      | null                    | false         | 输出使用已过时的 API 的源位置(保留命令行向后兼容性)              |
|      -classpath      |       javac.opt.arg.path       | javac.opt.classpath            | STANDARD  | FILEMANAGER | null      | null                    | false         | 指定查找用户类文件和注释处理程序的位置                           |
|         -cp          |       javac.opt.arg.path       | javac.opt.classpath            | STANDARD  | FILEMANAGER | null      | null                    | false         | 指定查找用户类文件和注释处理程序的位置(这个选项是-classpath的简写) |
|     -sourcepath      |       javac.opt.arg.path       | javac.opt.sourcepath           | STANDARD  | FILEMANAGER | null      | null                    | false         | 指定查找输入源文件的位置                                       |
|    -bootclasspath    |       javac.opt.arg.path       | javac.opt.bootclasspath        | STANDARD  | FILEMANAGER | null      | null                    | false         | 覆盖引导类文件的位置                                           |
|  -Xbootclasspath/p:  |       javac.opt.arg.path       | javac.opt.Xbootclasspath.p     | EXTENDED  | FILEMANAGER | null      | null                    | false         | 置于引导类路径之前                                             |
|  -Xbootclasspath/a:  |       javac.opt.arg.path       | javac.opt.Xbootclasspath.a     | EXTENDED  | FILEMANAGER | null      | null                    | false         | 置于引导类路径之后                                             |
|   -Xbootclasspath:   |       javac.opt.arg.path       | javac.opt.bootclasspath        | EXTENDED  | FILEMANAGER | null      | null                    | false         | 覆盖引导类文件的位置(这个选项是和-bootclasspath一样的)           |
|       -extdirs       |       javac.opt.arg.dirs       | javac.opt.extdirs              | STANDARD  | FILEMANAGER | null      | null                    | false         | 覆盖所安装扩展的位置                                           |
|   -Djava.ext.dirs=   |       javac.opt.arg.dirs       | javac.opt.extdirs              | EXTENDED  | FILEMANAGER | null      | null                    | false         | 覆盖所安装扩展的位置(和-extdirs一样)                            |
|        -proc:        |              null              | javac.opt.proc.none.only       | STANDARD  | BASIC       | ONEOF     | none,only               | false         | 控制是否执行注释处理和/或编译                                   |
|      -processor      |    javac.opt.arg.class.list    | javac.opt.processor            | STANDARD  | BASIC       | null      | null                    | false         | 要运行的注释处理程序的名称; 绕过默认的搜索进程                    |
|    -processorpath    |       javac.opt.arg.path       | javac.opt.processorpath        | STANDARD  | FILEMANAGER | null      | null                    | false         | 指定查找注释处理程序的位置                                      |
|     -parameters      |              null              | javac.opt.parameters           | STANDARD  | BASIC       | null      | null                    | false         | 生成元数据以用于方法参数的反射                                  |
|          -d          |    javac.opt.arg.directory     | javac.opt.d                    | STANDARD  | FILEMANAGER | null      | null                    | false         | 指定放置生成的类文件的位置                                      |
|          -s          |    javac.opt.arg.directory     | javac.opt.sourceDest           | STANDARD  | FILEMANAGER | null      | null                    | false         | 指定放置生成的源文件的位置                                      |
|          -h          |    javac.opt.arg.directory     | javac.opt.headerDest           | STANDARD  | FILEMANAGER | null      | null                    | false         | 指定放置生成的本机标头文件的位置                                 |
|      -implicit:      |              null              | javac.opt.implicit             | STANDARD  | BASIC       | ONEOF     | none,class              | false         | 指定是否为隐式引用文件生成类文件                                 |
|      -encoding       |     javac.opt.arg.encoding     | javac.opt.encoding             | STANDARD  | FILEMANAGER | null      | null                    | false         | 指定源文件使用的字符编码                                        |
|       -source        |     javac.opt.arg.release      | javac.opt.source               | STANDARD  | BASIC       | null      | null                    | false         | 提供与指定发行版的源兼容性                                      |
|       -target        |     javac.opt.arg.release      | javac.opt.target               | STANDARD  | BASIC       | null      | null                    | false         | 生成特定 VM 版本的类文件                                       |
|       -profile       |     javac.opt.arg.profile      | javac.opt.profile              | STANDARD  | BASIC       | null      | null                    | false         | 确保使用的 API 在指定的配置文件中可用                           |
|       -version       |              null              | javac.opt.version              | STANDARD  | INFO        | null      | null                    | false         | 输出版本信息                                                  |
|     -fullversion     |              null              | null                           | HIDDEN    | INFO        | null      | null                    | false         | 输出版本信息                                                  |
|      -XDdiags=       |              null              | null                           | HIDDEN    | INFO        | null      | null                    | false         | 这是编译器选项的后门                                           |
|        -help         |              null              | javac.opt.help                 | STANDARD  | INFO        | null      | null                    | false         | 输出标准选项的提要                                             |
|          -A          | javac.opt.arg.key.equals.value | javac.opt.A                    | STANDARD  | BASIC       | null      | null                    | true          | 传递给注释处理器的选项                                         |
|          -X          |              null              | javac.opt.X                    | STANDARD  | BASIC       | null      | null                    | false         | 输出非标准选项的提要                                           |
|          -J          |       javac.opt.arg.flag       | javac.opt.J                    | STANDARD  | BASIC       | null      | null                    | true          | 直接将 <标记> 传递给运行时系统,如果在命令行中使用,则抛出异常        |
|      -moreinfo       |              null              | null                           | HIDDEN    | BASIC       | null      | null                    | false         | 输出更多的信息                                                 |
|       -Werror        |              null              | javac.opt.Werror               | STANDARD  | BASIC       | null      | null                    | false         | 出现警告时终止编译                                             |
|       -prompt        |              null              | null                           | HIDDEN    | BASIC       | null      | null                    | false         |                                                              |
|       -prompt        |              null              | null                           | HIDDEN    | BASIC       | null      | null                    | false         | 每次错误后提示                                                 |
|         -doe         |              null              | null                           | HIDDEN    | BASIC       | null      | null                    | false         | 在发生错误时dump堆栈信息                                       |
|     -printsource     |              null              | null                           | HIDDEN    | BASIC       | null      | null                    | false         | 在类型擦除后输出source                                         |
|    -warnunchecked    |              null              | null                           | HIDDEN    | BASIC       | null      | null                    | false         | 在对泛型进行未检查的操作时发出警告信息                           |
|      -Xmaxerrs       |      javac.opt.arg.number      | javac.opt.maxerrs              | EXTENDED  | BASIC       | null      | null                    | false         | 设置要输出的错误的最大数目                                      |
|      -Xmaxwarns      |      javac.opt.arg.number      | javac.opt.maxwarns             | EXTENDED  | BASIC       | null      | null                    | false         | 设置要输出的警告的最大数目                                      |
|       -Xstdout       |       javac.opt.arg.file       | javac.opt.Xstdout              | EXTENDED  | INFO        | null      | null                    | false         | 重定向标准输出                                                 |
|       -Xprint        |              null              | javac.opt.print                | EXTENDED  | BASIC       | null      | null                    | false         | 输出指定类型的文本表示                                         |
|    -XprintRounds     |              null              | javac.opt.printRounds          | EXTENDED  | BASIC       | null      | null                    | false         | 输出有关注释处理循环的信息                                      |
| -XprintProcessorInfo |              null              | javac.opt.printProcessorInfo   | EXTENDED  | BASIC       | null      | null                    | false         | 输出有关请求处理程序处理哪些注释的信息                           |
|       -Xprefer       |              null              | javac.opt.prefer               | EXTENDED  | BASIC       | ONEOF     | source,newer            | false         | 指定读取文件, 当同时找到隐式编译类的源文件和类文件时              |
|      -Xpkginfo:      |              null              | javac.opt.pkginfo              | EXTENDED  | BASIC       | ONEOF     | always,legacy,nonempty  | false         | 指定读取文件, 当同时找到隐式编译类的源文件和类文件时              |
|          -O          |              null              | null                           | HIDDEN    | BASIC       | null      | null                    | false         | 不进行任何操作,后向兼容性                                       |
|        -Xjcov        |              null              | null                           | HIDDEN    | BASIC       | null      | null                    | false         | 生成jcov(代码覆盖)所需要的table                                |
|      -Xplugin:       |      javac.opt.arg.plugin      | javac.opt.plugin               | EXTENDED  | BASIC       | null      | null                    | false         | 要运行的插件的名称和可选参数                                    |
|       -Xdiags:       |              null              | javac.opt.diags                | EXTENDED  | BASIC       | ONEOF     | compact,verbose         | false         | 选择诊断模式                                                  |
|         -XD          |              null              | null                           | HIDDEN    | BASIC       | null      | null                    | false         | 这是编译器选项的后门,-XDx=y 将选项x的值设置为y                   |
|          @           |       javac.opt.arg.file       | javac.opt.arg.file             | STANDARD  | INFO        | null      | null                    | true          | 从文件读取选项和文件名,由CommandLine实现                        |
|      sourcefile      |              null              | null                           | HIDDEN    | INFO        | null      | null                    | false         |                                                              |

