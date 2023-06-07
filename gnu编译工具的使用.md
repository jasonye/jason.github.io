# 概述
重点介绍：aclocal、autoconf、automake、libtoolize configure、make 等GNU工具的使用、



工具之间的关系：
*   ![IMG](https://pic002.cnblogs.com/img/itech/200905/2009052517385173.png)



1. source code ---(autoscan)--> configure.scan  扫描目录生成文件
2. configure.scan--> rename to: configure.in  重命名文件

```
configure.in 和 configure.ac 的区别？

答案：
参考这里：https://stackoverflow.com/questions/4820045/configure-in-or-configure-ac

Its just a matter of style. Historically autoconf files were named `configure.in`. Now `configure.ac` is the preferred naming scheme. Its also discussed in the [documentation](http://www.gnu.org/software/autoconf/manual/html_node/Writing-Autoconf-Input.html#Writing-Autoconf-Input).

```

3.   Configure.in ---(aclocal)---> aclocal.m4  申城aclocal.m4文件、
4.   configure.in +(aclocal.m4 ---(autoconf)-----> configure 
5.   Makefile.am ---- (automake) ----> Makefile.in 
6.   Makefile.in ---> configure ---> Makefile 

     
# 每个步骤详细介绍
```
1.autoscan (autoconf): 扫描源代码以搜寻普通的可移植性问题，比如检查编译器，库，头文件等，生成文件configure.scan,它是configure.ac的一个雏形。

2.aclocal (automake):根据已经安装的宏，用户定义宏和acinclude.m4文件中的宏将configure.ac文件所需要的宏集中定义到文件 aclocal.m4中。aclocal是一个perl 脚本程序，它的定义是：“aclocal - create aclocal.m4 by scanning configure.ac”

user input files   optional input     process          output files
================   ==============     =======          ============
                    acinclude.m4 - - - - -.
                                          V
                                      .-------,
configure.ac ------------------------>|aclocal|
                 {user macro files} ->|       |------> aclocal.m4
                                      `-------'
3.autoheader(autoconf): 根据configure.ac中的某些宏，比如cpp宏定义，运行m4，声称config.h.in
user input files    optional input     process          output files
================    ==============     =======          ============
                    aclocal.m4 - - - - - - - .
                                             |
                                             V
                                     .----------,
configure.ac ----------------------->|autoheader|----> autoconfig.h.in
                                     `----------'
4.automake: automake将Makefile.am中定义的结构建立Makefile.in，然后configure脚本将生成的Makefile.in文件转换为Makefile。如果在configure.ac中定义了一些特殊的宏，比如AC_PROG_LIBTOOL，它会调用libtoolize，否则它会自己产生config.guess和config.sub
user input files   optional input   processes          output files
================   ==============   =========          ============
                                     .--------,
                                     |        | - - -> COPYING
                                     |        | - - -> INSTALL
                                     |        |------> install-sh
                                     |        |------> missing
                                     |automake|------> mkinstalldirs
configure.ac ----------------------->|        |
Makefile.am  ----------------------->|        |------> Makefile.in
                                     |        |------> stamp-h.in
                                 .---+        | - - -> config.guess
                                 |   |        | - - -> config.sub
                                 |   `------+-'
                                 |          | - - - -> config.guess
                                 |libtoolize| - - - -> config.sub
                                 |          |--------> ltmain.sh
                                 |          |--------> ltconfig
                                 `----------'
5.autoconf:将configure.ac中的宏展开，生成configure脚本。这个过程可能要用到aclocal.m4中定义的宏。
user input files   optional input   processes          output files
================   ==============   =========          ============
                   aclocal.m4 - - - - - -.
                                         V
                                     .--------,
configure.ac ----------------------->|autoconf|------> configure ----->autoconfig.h,Makefile
```





# 各种工具介绍

## 1、 autoscan

autoscan是用来扫描源代码目录生成configure.scan文件的。autoscan可以用目录名做为参数，但如果你不使用参数的话，那么autoscan将认为使用的是当前目录。autoscan将扫描你所指定目录中的源文件，并创建configure.scan文件。



## 2、 configure.scan

　　configure.scan包含了系统配置的基本选项，里面都是一些宏定义。我们需要将它改名为configure.in



## 3、 aclocal

　　aclocal是一个perl 脚本程序。aclocal根据configure.in文件的内容，自动生成aclocal.m4文件。aclocal的定义是：“aclocal - create aclocal.m4 by scanning configure.ac”。

 

## 4、 autoconf

　　使用autoconf，根据configure.in和aclocal.m4来产生configure文件。configure是一个脚本，它能设置源程序来适应各种不同的操作系统平台，并且根据不同的系统来产生合适的Makefile，从而可以使你的源代码能在不同的操作系统平台上被编译出来。

　　configure.in文件的内容是一些宏，这些宏经过autoconf 处理后会变成检查系统特性、环境变量、软件必须的参数的shell脚本。configure.in文件中的宏的顺序并没有规定，但是你必须在所有宏的最前面和最后面分别加上AC_INIT宏和AC_OUTPUT宏。

　　在configure.ini中：

　　#号表示注释，这个宏后面的内容将被忽略。

　　AC_INIT(FILE)　这个宏用来检查源代码所在的路径。

　　AM_INIT_AUTOMAKE(PACKAGE, VERSION)　这个宏是必须的，它描述了我们将要生成的软件包的名字及其版本号：PACKAGE是软件包的名字，VERSION是版本号。当你使用make dist命令时，它会给你生成一个类似helloworld-1.0.tar.gz的软件发行包，其中就有对应的软件包的名字和版本号。

　　AC_PROG_CC　　这个宏将检查系统所用的C编译器。

　　AC_OUTPUT(FILE)　　这个宏是我们要输出的Makefile的名字。

　　我们在使用automake时，实际上还需要用到其他的一些宏，但我们可以用aclocal 来帮我们自动产生。执行aclocal后我们会得到aclocal.m4文件。

　　产生了configure.in和aclocal.m4 两个宏文件后，我们就可以使用autoconf来产生configure文件了。

 

## 5、 Makefile.am

　　Makefile.am是用来生成Makefile.in的，需要你手工书写。Makefile.am中定义了一些内容：

　　AUTOMAKE_OPTIONS　　这个是automake的选项。在执行automake时，它会检查目录下是否存在标准GNU软件包中应具备的各种文件，例如AUTHORS、ChangeLog、NEWS等文件。我们将其设置成foreign时，automake会改用一般软件包的标准来检查。

　　bin_PROGRAMS　　这个是指定我们所要产生的可执行文件的文件名。如果你要产生多个可执行文件，那么在各个名字间用空格隔开。

　　helloworld_SOURCES　　这个是指定产生“helloworld”时所需要的源代码。如果它用到了多个源文件，那么请使用空格符号将它们隔开。比如需要helloworld.h，helloworld.c那么请写成helloworld_SOURCES= helloworld.h helloworld.c。

　　如果你在bin_PROGRAMS定义了多个可执行文件，则对应每个可执行文件都要定义相对的filename_SOURCES。

 

## 6、 automake

　　我们使用automake，根据configure.in和Makefile.am来产生Makefile.in。

　　选项--add-missing的定义是“add missing standard files to package”，它会让automake加入一个标准的软件包所必须的一些文件。

　　我们用automake产生出来的Makefile.in文件是符合GNU Makefile惯例的，接下来我们只要执行configure这个shell 脚本就可以产生合适的 Makefile 文件了。

　　

## 7、 Makefile

　　在符合GNU Makefiel惯例的Makefile中，包含了一些基本的预先定义的操作：

　　make　　根据Makefile编译源代码，连接，生成目标文件，可执行文件。

　　make clean　　清除上次的make命令所产生的object文件（后缀为“.o”的文件）及可执行文件。

　　make install　　将编译成功的可执行文件安装到系统目录中，一般为/usr/local/bin目录。

　　make dist　　产生发布软件包文件（即distribution package）。这个命令将会将可执行文件及相关文件打包成一个tar.gz压缩的文件用来作为发布软件的软件包。它会在当前目录下生成一个名字类似“PACKAGE-VERSION.tar.gz”的文件。PACKAGE和VERSION，是我们在configure.in中定义的AM_INIT_AUTOMAKE(PACKAGE, VERSION)。

　　make distcheck　　生成发布软件包并对其进行测试检查，以确定发布包的正确性。这个操作将自动把压缩包文件解开，然后执行configure命令，并且执行make，来确认编译不出现错误，最后提示你软件包已经准备好，可以发布了。
　　make distclean　　类似make clean，但同时也将configure生成的文件全部删除掉，包括Makefile。


## 8.   libtoolize 
在mac上没有这个。 通常需要使用glibtool和glibtoolize，因为libtool作为创建Mach-O动态库的二进制工具已在OS X上存在。因此，这就是MacPorts使用程序名称转换安装它的方式，尽管端口本身仍被命名为'libtool'。

     参考这个问题：https://stackoverflow.com/questions/15448582/installed-libtool-but-libtoolize-not-found
     
     
# 参考文档：
* 工具链图：https://www.laruence.com/2008/11/11/606.html
* 演示Demo：http://blog.chinaunix.net/uid-26748223-id-4360080.html
* 链接图：https://www.cnblogs.com/itech/archive/2010/11/28/1890220.html