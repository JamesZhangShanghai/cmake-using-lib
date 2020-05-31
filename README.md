#CMAKE 使用外部头文件及动态库

##头文件
为了让我们的工程能够找到 hello.h 头文件，我们需要引入一个新的指令
INCLUDE_DIRECTORIES，其完整语法为：
```
INCLUDE_DIRECTORIES([AFTER|BEFORE] [SYSTEM] dir1 dir2 ...)
```
这条指令可以用来向工程添加多个特定的头文件搜索路径，路径之间用空格分割，如果路径
中包含了空格，可以使用双引号将它括起来，默认的行为是追加到当前的头文件搜索路径的
后面，你可以通过两种方式来进行控制搜索路径添加的方式：
１，CMAKE_INCLUDE_DIRECTORIES_BEFORE，通过 SET 这个 cmake 变量为 on，可以
将添加的头文件搜索路径放在已有路径的前面。
２，通过 AFTER 或者 BEFORE 参数，也可以控制是追加还是置前。
现在我们在 src/CMakeLists.txt 中添加一个头文件搜索路径，方式很简单，加入：
`INCLUDE_DIRECTORIES(/usr/include/hello)`


##共享库

为 target 添加共享库
我们现在需要完成的任务是将目标文件链接到 libhello，这里我们需要引入两个新的指令

LINK_DIRECTORIES 和 TARGET_LINK_LIBRARIES
LINK_DIRECTORIES 的全部语法是：
```
LINK_DIRECTORIES(directory1 directory2 ...)
```
这个指令非常简单，添加非标准的共享库搜索路径，比如，在工程内部同时存在共享库和可
执行二进制，在编译时就需要指定一下这些共享库的路径。这个例子中我们没有用到这个指
令。
TARGET_LINK_LIBRARIES 的全部语法是:
```
TARGET_LINK_LIBRARIES(target library1
<debug | optimized> library2
...)
```
这个指令可以用来为 target 添加需要链接的共享库，本例中是一个可执行文件，但是同样可以用于为自己编写的共享库添加共享库链接。
为了解决我们前面遇到的 HelloFunc 未定义错误，我们需要作的是向
src/CMakeLists.txt 中添加如下指令：
`TARGET_LINK_LIBRARIES(main hello)`
也可以写成
`TARGET_LINK_LIBRARIES(main libhello.so)`
这里的 hello 指的是我们上一节构建的共享库 libhello.

##特殊的环境变量 CMAKE_INCLUDE_PATH 和 CMAKE_LIBRARY_

务必注意，这两个是环境变量而不是 cmake 变量。
使用方法是要在 bash 中用 export 或者在 csh 中使用 set 命令设置或者
```
CMAKE_INCLUDE_PATH=/home/include cmake ..等方式。
```
这两个变量主要是用来解决以前 autotools 工程中
--extra-include-dir 等参数的支持的。
也就是，如果头文件没有存放在常规路径(`/usr/include`, `/usr/local/include` 等)，
则可以通过这些变量就行弥补。
我们以本例中的 hello.h 为例，它存放在`/usr/include/hello` 目录，所以直接查找肯
定是找不到的。
前面我们直接使用了绝对路径 `INCLUDE_DIRECTORIES(/usr/include/hello)`告诉工
程这个头文件目录。
为了将程序更智能一点，我们可以使用 CMAKE_INCLUDE_PATH 来进行，使用 bash 的方法
如下：
`export CMAKE_INCLUDE_PATH=/usr/include/hello`
然后在头文件中将 `INCLUDE_DIRECTORIES(/usr/include/hello)`替换为：
```
FIND_PATH(myHeader hello.h)
IF(myHeader)
INCLUDE_DIRECTORIES(${myHeader})
ENDIF(myHeader)
```
上述的一些指令我们在后面会介绍。
这里简单说明一下，FIND_PATH 用来在指定路径中搜索文件名，比如：
```
FIND_PATH(myHeader NAMES hello.h PATHS /usr/include
/usr/include/hello)
```
这里我们没有指定路径，但是，cmake 仍然可以帮我们找到 hello.h 存放的路径，就是因
为我们设置了环境变量 `CMAKE_INCLUDE_PATH`。
如果你不使用` FIND_PATH，CMAKE_INCLUDE_PATH `变量的设置是没有作用的，你不能指
望它会直接为编译器命令添加参数-I<CMAKE_INCLUDE_PATH>。
以此为例，`CMAKE_LIBRARY_PATH` 可以用在 FIND_LIBRARY 中。
同样，因为这些变量直接为 FIND_指令所使用，所以所有使用 FIND_指令的 cmake 模块都
会受益。