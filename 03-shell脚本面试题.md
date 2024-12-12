**#shell脚本**

什么是shell？

命令解释器

什么是默认登录shell，如何改变指定用户的登录shell

答:在Linux操作系统，“/bin/bash”是默认登录shell，是在创建用户时分配的。使用chsh命令可以改变默认的shell。

如何将标准输出和错误输出同时重定向到同一位置?

答：这里有两个方法来实现：

方法一：2>&1 (如# ls /usr/share/doc > out.txt 2>&1 )

方法二：&> (如# ls /usr/share/doc &> out.txt )

Shell 脚本传参方法有哪几种？

Shell 脚本传参主要依赖位置参数、特殊变量（包括 $@、$\*、$# 等）、环境变量

shell脚本中“$?”标记的用途是什么？

答：在写一个shell脚本时，如果你想要检查前一命令是否执行成功，在if条件中使用“$?”可以来检查前一命令的结束状态

可以在shell脚本中使用哪些类型的变量?

答：在shell脚本，我们可以使用两种类型的变量：

系统定义变量、用户定义变量

系统变量是由系统系统自己创建的。这些变量通常由大写字母组成，可以通过“set”命令查看。

用户变量由系统用户来生成和定义，变量的值可以通过命令“echo $<变量名>”查看。

` `shell脚本中break命令的作用 ?

答：break命令一个简单的用途是退出执行中的循环。我们可以在while和until循环中使用break命令跳出循环。

shell脚本中continue命令的作用 ?

答：continue命令不同于break命令，它只跳出当前循环的迭代，而不是整个循环。continue命令很多时候是很有用的，例如错误发生，但我们依然希望继续执行大循环的时候

Shell脚本的运行方式有哪些？区别？

Sh 1.sh      +x    /path/1.sh            source /path/1.sh  (刷新)   . /path/1.sh   

脚本第一行声明“#!/bin/bash”的作用 ?

答：#!/bin/bash是shell脚本的第一行，称为释伴（shebang）行。这里#符号叫做hash，而! 叫做 bang。它的意思是命令通过 /bin/bash 来执行。

` `如何调试shell脚本 ?

答：使用'-x'参数（sh -x myscript.sh）可以调试shell脚本。另一个种方法是使用‘-nv’参数( sh -nv myscript.sh)。

在shell脚本中，如何测试文件 ?

答：test命令可以用来测试文件。基础用法如下：

Test         用法

-d 文件名    如果文件存在并且是目录，返回true

-e 文件名    如果文件存在，返回true

-f 文件名    如果文件存在并且是普通文件，返回true

-r 文件名    如果文件存在并可读，返回true

-s 文件名    如果文件存在并且不为空，返回true

-w 文件名    如果文件存在并可写，返回true

-x 文件名    如果文件存在并可执行，返回true

在shell脚本中，如何写入注释 ?

答：注释可以用来描述一个脚本可以做什么和它是如何工作的。每一行注释以#开头。

如何让 shell 就脚本得到来自终端的输入?

答：read命令可以读取来自终端（使用键盘）的数据。read命令得到用户的输入并置于你给出的变量中。

如何取消变量或取消变量赋值 ?

答：“unset”命令用于取消变量或取消变量赋值。语法如下所示：unset <变量名>

如何执行算术运算 ?

答：有两种方法来执行算术运算：

1.使用expr命令

\# expr 5 + 2

2.用一个美元符号和方括号（$[ 表达式 ]）例如：

test=$[16 + 4] ; test=$[16 + 4]

什么是shell函数？作用是什么？

shell函数的作用 1.避免代码重复,减少代码的冗余 2.模块化处理,随调随用,增强代码的可读性和维护性

何写一个函数 ?

function example {

echo "Hello world!"

}


如何获取一个文件每一行的第三个字段 ?

awk'{print $3}'

假如文件中某行第一个元素是 first，如何获取满足条件行的第二个字段

awk'{ if ($1 == "FIND") print $2}'

写出 shell 脚本中所有循环语法 ?

for 循环 :

For i in$(ls);do

echo item:$i

Done

while 循环 :

#!/bin/bash

COUNTER=0

while [ $COUNTER -lt 10 ]; do

echo The counter is $COUNTER

let COUNTER=COUNTER+1

Done

until 循环 :

#!/bin/bash

COUNTER=20

until [ $COUNTER -lt 10 ]; do

echo COUNTER $COUNTER

let COUNTER-=1

done

如何获取文本文件的第 10 行 ?

head -10 file|tail -1

命令 “export” 有什么用 ?

使变量在子 shell 中可用。 定义环境变量

如何在后台运行脚本 ?

nohup command&

& 和 && 有什么区别

& - 希望脚本在后台运行的时候使用它

&& - 当前一个脚本成功完成才执行后面的命令/脚本的时候使用它

' 和 " 引号有什么区别 ?

' - 当我们不希望把变量转换为值的时候使用它。

" - 会计算所有变量的值并用值代替。

如何使用 awk 列出 UID 小于 100 的用户名 ?

awk -F: '$3<100{print $1}' /etc/passwd

写出输出数字 0 到 100 中 3 的倍数(0 3 6 9 …)的命令 ?

for i in {0..100..3}; do echo $i; done

或

for (( i=0; i<=100; i=i+3 )); do echo "Welcome $i times"; done


[ $a == $b ] 和 [ $a -eq $b ] 有什么区别？

[ $a == $b ] - 用于字符串比较

[ $a -eq $b ] - 用于数字比较

