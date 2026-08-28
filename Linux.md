

### curl

CNB的token：`8nKNqW88E1139XKvcE00EGpFnKA`

```sh
curl.exe --user "cnb:8nKNqW88E1139XKvcE00EGpFnKA" --remote-name --location "https://api.cnb.cool/wangxuyang01/clouddrive/-/git/raw/main/test/file.md"
```

- --request 指定请求方法（GET、POST）
- --header 指定请求头，可多个
- --data 发送POST数据，JSON格式
- --remote-name使用远程文件名
- --output 新文件名，重命名文件
- --location 自动重定向
- --user 用户名:密码  
- --include 包含响应头

重要，关键，发送JSON数据固定语法：

```sh
curl --request POST --header "Content-Type: application/json"
--data '{"k1":"v1","k2":v2}' 链接
```

认证token：

```sh
curl --header "Authorization: Bearer 你的Token" 链接
```

### 环境配置

两边家目录都是~

```sh
export name="value" ; echo "$name"
```

常用操作，追加PATH：

```sh
export PATH=$PATH:你的路径
```

sh启动文件：`. ~/.bashrc`

注意，ps脚本格式必须是UTF8 with BOM ！！！

ps启动文件`$PROFILE`，需新建

ps设置Path：

```sh
$env:path += ";你的路径" 
```

### APT

每次安装必须更新升级

```sh
apt update && apt upgrade -y
```

针对Nodejs：

```sh
apt install -y nodejs npm
```

针对Python：

```sh
apt install -y python3 python3-pip ; alias python='python3' 
```


## 单行多命令

### 子命令

非常有用的技巧！命令中的命令，Shell函数的本质就是子命令

两端语法均为 `$(子命令) `

```sh
text=$(cat 1.md)
for file in $(ls)
```


### 函数

注意注意，Linux不能写在一行，一定要回车，PS可以

结构基本一样，支持中文，Linux形参用数字，PS形参用名称

```sh
function 函数名(){
    echo "$1 $2" 
}
```

调用函数：`res=$(函数名 arg1 arg2) ` 

Linux脚本本质上也可看做函数，文件名就是函数名，内容是函数体

可无参数，用于常见命令行封装

```sh
function 函数名($name){echo "你好,$name"} 
$res=$(函数名 shell)
```

PS脚本可看做函数，形参用 `$args[0]`、`$args[1]`表示

### 管道与重定向

一根竖线，左边输出作为右边输入：`ls -lh | head -n 1 `

覆盖：`echo "text" > 1.md`

追加（避免误删更推荐）：`echo "text" >> 1.md`

### 分号

用分号将多行命令写在一行，非常适合终端场景

各命令依次执行，执行成功与否相互独立

分号两边需要加空格

`a="hello" ; b="world" ; echo "${a} ${b}"`

### 条件判断

不要用if else，Linux中所有条件判断都应该使用`[[]]`

```sh
[[ 条件表达式 ]] && 为真时执行 || 为假时执行
```

```sh
[[ 2 -gt 1 ]] && echo "真" || echo "假"
```

PS5中使用` if(){} else{}` 

PS7版本才支持 `&&` 和 `||`

两边整数比较都是用`-gt、-lt、-eq`

路径存在性，`-e`  `test-path`

```sh
[[ -e test.txt ]] && echo "存在" || echo "不存在"
```

```powershell
if(test-path "路径"){echo "存在"}
```



## 变量

### 声明和使用

保险起见，定义和使用时字符串都必须加双引号！！！

声明变量`str="hello"`，等号两边一定不能有空格

路径含变量也要加双引号

使用变量如`echo "$变量名" 或 "其它文本${变量名}"`

用read让用户输入变量值：

```sh
echo "提示词:" ; read name ; echo "$name"
```

PowerShell中：`${变量名} = "值" ` ， `echo "其它文本${变量名}"`

无歧义时可不加花括号

整数均会自动推算：

```sh
n=3 ; n=$((n+1))
```

四则运算：`n=$((n+1))`，`$(())`必须写，内部若有优先级再加括号




### 通配包含检测

通配：`*` 任意数量任意字符

`==`文本通配检测

```sh
[[ "hello" == *e* ]] && echo true
```

ps中通配检测：

```sh
"hello" -like "*e*"
```

限定首尾就不要星号


### 替换

通配替换：用变量，无引号，后不写即删除

```sh
str="hello,shell" ; echo ${str/h*o/你好} 
```

PS上是正则替换，无echo，后不写即删除：

```sh
"hello,shell" -replace "h.*o" , "你好"
```

### 数组和遍历

定义：

```sh
arr=(1 2 3 4 5)
arr=$(ls)
```

注意注意，务必注意！使用数组：`${arr[@]}`，`${arr}`是错的！

```sh
for item in ${arr[@]} ; do 语句1 ; 语句2 ; done
```

也可多行写，for do done开头

文件列表或整数序列：

```sh
for file in $(ls)
for n in $(seq 5)
```

追加`arr=(${arr[@]} 6 7)`

负索引：`${arr[-1]}`

变量索引：`n=3 ; ${arr[n]}`

子集：`${arr[@]:0:3}`

ps中：

```sh
$arr = @(1, 2, 3, 4, 5)
foreach ($file in $(ls)) { echo $file.name }
```

对象定义与遍历：

```sh
declare -A obj=(["a"]=1 ["b"]=2)
obj["c"]=3
for key in "${!obj[@]}"; do echo "$key: ${obj[$key]}"; done
```



### 文件管理

ls自动显示兆字节：`ls -lh`

查看cat，前/后5行`head/tail -n 5 file`

ps中是`get-content 文件 -head 5`

查看内核：`uname -a`

回上一级 `cd ..`，回家： `cd ~`

复制，-r递归选项一定要加：

```ls
cp -r f1 f2 dir1 目标目录
```

移动就改成mv

重命名的本质为移动：`mv 旧 新`

新建文件 `touch 1.md` `new-item`

递归新建目录 `mkdir -p dir1/dir2`，先判断是否有

强制递归删除非空目录： `rm -rf dir` `remove-item 路径 -recurse -force`

删除空目录：`rmdir dir`

强制删除文件：`rm -f f1 f2`

### 7z

查看：`7z l 压缩名.zip`

整个压缩与解压： `7z a 压缩名.zip 目录` 、 `7z x 压缩名.zip` 

指定解压目录`-o目录`，自动新建且不能有空格

强制覆盖 `-ao`

提取单个文件，不会嵌套目录 ：`7z e 压缩名.zip 里面的哪个文件`

### Git

Git快速提交：

```sh
git add . && git commit -m "$(date)" && git push
```





