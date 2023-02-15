---
layout: post
title:  "Shell"
date:   2023-02-02
categories: shell

---

# Shell

Shell，命令解释器，**将用户指令翻译给系统内核执行，返回结果。**

种类有很多：

```shell
> cat /etc/shells
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/dash
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```



Shell脚本，即一组命令按语法进行组合。

Linux的启动过程中，就会执行很多的Shell脚本。

通过编写Shell脚本，就不用直接和内核的C等代码打交道，仅用简单的Shell脚本语法就可以和内核交互了。



# 变量

`bash` 中的变量是弱类型的，默认会当作字符串处理。

## 变量的使用

定义：

```sh
# 变量名=值
name="youker"
```

注意，`=` 左右两边不可以有空格！（平常有良好编程习惯的尤其要注意这点！好习惯在这里成了错误语法...）



引用：

```sh
echo ${name}
# 或者不产生语法歧义时，简化掉{}
echo $name
```



撤销：

```sh
# 直接跟变量名
unset name
```

不用的变量，应该及时撤销释放掉，这是个好的习惯。



## 不同bash变量

「本地变量」

当运行脚本时，会产生bash下的一个脚本子进程。即便同一个脚本，两个子进程中的变量仍是互相独立的本地变量。

注意，如果使用`source` 执行脚本，这不会产生子进程，而是在当前bash进程中执行，就能访问到当前bash的本地变量！

方式：

```sh
source /XXX/XXX/xx.sh
# 或者可以简化为点 .
. /XXX/XXX/xx.sh
```



「环境变量」

环境变量的生效范围为**当前shell进程及其子进程。**

使用`export`关键字指明对应的变量为环境变量：`export varname=value`。



「特殊变量」

- `$$`：当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。
- `$?` ：上次执行命令的返回值
- `$@` 或`$*` ：都表示传给脚本的参数列表
- `$#`：传给脚本的参数个数
- `$0`：当前脚本名
- `$1`：传递给脚本的第一个参数

注意，如果在函数体内，则表示传入给这个函数的参数列表。

# 结构控制语句

## if

长这样用：

```sh
if 判断条件
then 命令1
else 命令2
fi
```

或者这样更常见：

```sh
if 判断条件1; then
	命令1
elif 判断条件2; then
	命令2
else
	命令3
fi
```

「判断条件」包括：**命令语句、测试语句**，这2种方式。



「命令语句」

一般以命令的执行成功与否来判断。

如果命令**成功执行**并**正常结束**，则返回值 **0**，判断条件**为真**；反之为假。（这里与Python的True相当于1，False相当于0，是反着来的！）



**「测试语句」**

Shell特有，计算一个表达式的结果真假来作为条件。包括**「字符串测试」、「文件测试」、「数值测试」。**

写法两种：

```sh
test -f  "$1"  # 使用关键字 test
[ -f "$1" ]    # 直接用方括号表示测试语句，注意里边表达式左右两侧留空格！
```

且条件之间可以用「逻辑操作符」进行组合：

```sh
if [ "$s1" -gt 0 -o "$s3" -lt 10 ]; then
	echo "s1大于0且s3小于10"
else
	echo "不在指定范围内"
fi
```

- `-a` 与，也可以用`&&`，但要注意`[]`的使用，避免语法错误。**`&&` 是短路与，只要前一个命令执行失败，就不会执行后一个命令！**
- `-o` 或，也可以用`||`，**同理`||`有短路功能，只要前一个命令为真，后一个命令就不执行了！**
- `!` 非





「字符串测试」常用示例：

```sh
str1="str1"
str2="str2"

if [ -z "" ]; then
    echo "-z 为空串(长度为0)时返回真"
else
    echo "XXX"
fi

if [ -n $str2 ]; then
    echo "-n 为非空串时返回真"
else
    echo "XXX"
fi


if [ $str1 = $str2 ]; then
    echo "相等时返回真"
elif [ $str1 != $str2 ]; then
    echo "不相等时返回真！"
else
    echo "XXX"
fi

---

$ sh tmp.sh
-z 为空串(长度为0)时返回真
-n 为非空串时返回真
不相等时返回真！
```



「文件测试」

格式如：`[ -flag pathname ]`

常用标志如下：

```sh
-d pathname : 当pathname 存在并且是一个目录时返回真
-f filename : 存在且是文件时 返回真
-e pathname : 文件或目录存在 返回真
-w pathname : 存在且可写 返回真
-x pathname : 存在且可执行 返回真
file1 -nt file2 : file1比file2新 返回真
```



「数值测试」

格式如：`[ val1 -flag val2 ]`

```sh
int1 -eq int2 : 如果int1 等于int2，则返回真
int1 -ne int2 : 不等于，则返回真 not equal
int1 -lt int2 : 小于 less than
int1 -le int2 : 小于等于 less equal
int1 -gt int2 : 大于 great than
int1 -ge int2 : 大于等于 great equal
```



## [] 与 [[]]

### 判断变量是否为空

当使用”-n”或者”-z”这种方式判断变量是否为空时，”[ ]”与”[[  ]]”是有区别的。

使用”[ ]”时需要在变量的外侧加上双引号，与test命令的用法完全相同，使用”[[  ]]”时则不用。



### 组合判断条件时

在使用`[[  ]]`时，不能使用`-a`或者`-o` 对多个条件进行连接。

在使用`[  ]`时，如果使用`-a`或者`-o`对多个条件进行连接，`-a`或者`-o`必须被包含在`[ ]`之内。

在使用`[  ]`时，如果使用`&&`或者`||`对多个条件进行连接，`&&`或者`||`必须在`[ ]`之外。





## case

直接用例子说明语法结构：

```sh
#* 按照用户的命令执行服务
command=$1
case $command in
  start)
        echo "start"
        ;;
  stop | Stop)
        echo "stop"
        ;;
  restart)
        echo "stop"
        sleep 2
        echo "start"
        ;;
  *)
        echo $"Usage: $0 {start|stop|restart}"
        exit 1
esac
```

将变量与底下各个「正则字符串」选项按顺序进行比较。

符合则执行子句，不再与其他项比较。`;;` 相当于`break` 语句，防止执行后续其他比较。



## for迭代

`for` 结构：

```sh
for variable [in argument-list]; do
    command-list
done
```

示例：

```sh
# 显示当前路径下所有文件，* 默认当前路径下的文件/目录名
for name in *; do
    if [ -f "${name}" ]; then
        echo "文件存在：${name}"
    fi
done
```



## while循环

格式：

```sh
while 判断条件; do
    command-list
done
```



## exit 

`exit` 退出脚本执行。

- `exit 0`：正常退出
- `exit 1`：异常退出。数字大于0，有不同含义。1到255中，1、2、127 为系统预留的错误状态码，其他状态码可自定义。





# 命令

很经常用到`echo`命令，这里说明下`echo`的输出格式控制：

- `-n` 不换行输出
- `-e` 特殊字符生效。比如`\n`  认为是换行



(待补充)



# 函数定义与调用

Shell 函数的本质是一段可以重复使用的脚本代码。

定义函数的语法：

```sh
function name() {
    statements
    [return value]
}

# 或者省略 function 关键字
name() {
    statements
    [return value]
}

# 或者省略圆括号
function name {
    statements
    [return value]
}
```



调用很简单，直接使用函数名即调用了。

调用时可以传递参数，也可以不传递。如果不传递参数，直接给出函数名字即可：

```sh
name
```

如果传递参数，那么多个参数之间以空格分隔：

```sh
name param1 param2 param3
```

**且，调用的语句，在定义语句的前后都可以！**