#### 1 _返回值0表示成功，其他值表示失败_
shell的返回值只能是一个介于0~255之间的整数，其中只有0表示成功，其他值表示失败。

#### 2 _shell编程不限制函数定义和调用的顺序_
先写调用后定义或者先写定义后调用都是可以的
 
#### 3 _shell编程函数的返回值仅仅表示函数退出的状态_
不像高级语言编程中，函数能够返回执行的结果。要想拿到推出的结果。
- _方法一_ 使用全局变量
```shell
#!/bin/bash
sum=0  #全局变量
function getsum(){
    for((i=$1; i<=$2; i++)); do
        ((sum+=i))  #改变全局变量
    done
    return $?  #返回上一条命令的退出状态
}
read m
read n
if getsum $m $n; then
    echo "The sum is $sum"  #输出全局变量
else
    echo "Error!"
fi
```
-  _方法二_ 用echo输出函数执行结果，使用$()获取结果
```shell
#!/bin/bash
function getsum(){
    local sum=0  #局部变量
    for((i=$1; i<=$2; i++)); do
        ((sum+=i))
    done
   
    echo $sum
    return $?
}
read m
read n
total=$(getsum $m $n)
echo "The sum is $total"
#也可以省略 total 变量，直接写成下面的形式
#echo "The sum is "$(getsum $m $n)
```

#### 4 _shell中所有变量默认是字符串_
shell是脚本语言，没有高级语言严格区分整数和字符串的要求。
可以使用以下方式声明，进行整数运算
```shell
#!/bin/bash
declare -i m n ret
m=10
n=30
```

#### 5 _判断语句和逻辑运算的区别_
```
if 判断条件时，用 (()) 来处理整型数字，用 [[ ]] 来处理字符串或者文件
```

#### 6 _until和select循环_
```shell
   until ((i > 100))
   do
    ((sum += i))
    ((i++))
   done
```
until循环只有不符合条件时才能继续循环

```shell
   select name in "Linux" "Windows" "Mac OS" "UNIX" "Android"
   do
    echo $name
   done
```
select循环就是一个死循环，只有遇到break或者Ctrl+D退出循环

#### 7 _continue和break_
shell编程的continue和break能够跳出或者跳过多层循环

