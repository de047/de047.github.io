
## 二进制与编码

> ### Uint8Array 、 Buffer  、ArrayBuffer、 Blob、base64、hs256

新建Uint8Array： `i = new Uint8Array([65,65,65])`

从文件新建： `new Uint8Array(fs.readFileSync(''))`

Node.js `fs.readFileSync()`返回`Buffer`，写入也需要`Buffer.from()`

从网页文件选取器新建：

```js
const reader = new FileReader()
reader.onload=e=> i = new Uint8Array(e.target.result)
reader.readAsArrayBuffer(file)
```

转到Buffer 并写入文件：

```js
const fileStream = fs.createWriteStream('./文件名')
fileStream.write(Buffer.from(i))
```

文本编码和解码：

```js
new TextEncoder().encode('AAA')
new TextDecoder().decode(i)
```

链接参数部分编码和解码：

```js
encodeURIComponent('中文?/#{}[]')
decodeURIComponent('')
```

转为Blob和url下载文件：

```js
const blob = new Blob([包含i的数组])
const url = URL.createObjectURL(blob)
```

fetch 经典代码：

```js
const res = await fetch(url,options)
const reader = res.body.getReader()
while(1){
	const {value,done} = await reader.read()
	if(done){break}else{
		此时的value就是Uint8Array		
	}
}
```



合并uint8array数组：

```js
 let myArrays = [new Uint8Array(16384), new Uint8Array(16384), new Uint8Array(16384), new Uint8Array(16384), new Uint8Array(16384), new Uint8Array(8868)];

// Get the total length of all arrays.
let length = 0;
myArrays.forEach(item => {
  length += item.length;
});

// Create a new array with total length and merge all source arrays.
let mergedArray = new Uint8Array(length);
let offset = 0;
myArrays.forEach(item => {
  mergedArray.set(item, offset);
  offset += item.length;
});

// Should print an array with length 90788 (5x 16384 + 8868 your source arrays)
console.log(mergedArray);
```

## 加密

SHA256：得到Uint8Array → 生成摘要 → 转为16进制

```js
const a=await crypto.subtle.digest('SHA-256', i)
const b = [...new Uint8Array(a)].map(x=>x.toString(16).padStart(2,'0')).join('')
```

DER二进制、PEM可打印二进制（base64），核心内容、元数据、容器格式、文件编码格式的区别

云服务的签名： 请求要素原文拼接 → HS256 → base64 → encodeURI

base64、base64URL、sha256、Hmacsha256、encodeURIComponent、UTF8、ASCII

关键伪代码： `原文、hmacsha256(原文，密钥)`


## 文本 

> 文本是字符组成的数组，方法诸多相似

字符串是不可变的，删除替换都会返回新的

长度：`'js'.length()` 

包含检测：`js.includes('j')` 

比较相等：`'js'==='js'`

找位置：`'js'.indexOf('s')` 没找到就报错

拼接拆分完全一样：

```js
[1,2].join('&')  → '1&2'
'1&2'.split('&')  → ['1','2']
```

拼接用加号，减0转数值：`'1'-0`，加空转文本：`3+''` 

比较：`1=="1"==true`、`1!=="1"`

正负索引：`'js'.at(-1)`

子集： `'JavaScript'.slice(0,-1)`注意-1是倒数第二个，不写才到最后

`.slice(-4,) `  最后4个，`slice() `  独立拷贝

变量插值，JS使用反引号美元符花括号，Py使用f前缀和括号`f"静态文本{变量或表达式}"`：

```js
`静态文本${变量或表达式}`
```

JS正则：不加引号
```js
找到的数组='hello,world'.match(/h.*?o/gi)
替换后的文本='hello,world'.replace(/h.*?o/gi,'你好')
```

## 对象

> 注意，Python不要用str、list、dict、len、sum、max做变量名，会覆盖内置的

`{a:1,b:2}`、 `{'a':1,'b:2'}` 注意Py的键一定要等号

JS静态属性：`obj.key` 、`obj['key']`，Py中只能用`mydict['key']`

动态属性：`obj[key]`，无引号，py中变量属性`mydict[key]key是变量`

简写属性和方法 `{a:a}`→`{a}`、`{ f(){} }`

JS内存地址相同检测：`obj1===obj2` ，py用`obj1 is obj2`

一个示例对象遍历：
```js
for (const [k,v] of Object.entries({a:1,b:2})){console.log(`${k}:${v}`)}
```

合并对象` {...obj1,...obj2}`

只用于原始值元素，规避指针拷贝问题

 

## 数组

转为正式数组，JS用`[...类数组]`或 `Array.from(类数组)`

长度、包含检查、查找同文本

JS合并数组，返回新的`[...arr1 , ...arr2 ]`，Py用加号：`list1+list2`。注意嵌套陷阱，一般只用于原始值元素

push() 追加，pop(n)删除第n项，不写即最后

`arr.方法(item,index)=>{逻辑，返回值})`，map每次返回一个新元素，forEach直接操作源数组无返回，filter每次返回布尔筛选出true的

数组转set会去重：

```js
const set = new Set(数组)
`has(item)`是否包含某个元素
`delete(item)`删除元素，返回是否成功
```

## 模块

注意注意，务必注意！相对路径一定要加`./`，否则从`node_modules`一直往上找，然后找全局的`node_modules`，即模块查找顺序


cjs导出导入：

```js
module.exports={a,b} //或
exports.a=...

//千万不能写成：
exports=... // 错误

导入
const {a,b}=require('./module')
```


ESM导出导入，注意，node环境需要.mjs文件，模块名不要写后缀会自动推导

具名：

```js
具名：
export {a,b}
import {a as alias1,b as alias2} from './module'

默认：
export default A
import 本地别名 from './module'
```


## 节流防抖

setTimeout(函数名,毫秒时间)

可嵌套以模拟循环

setInterval(函数名,毫秒时间)

可赋值给变量以传给clearInterval()、clearTimeout()以打断定时

```js
function debounce(func, wait = 50) {
  let timer
  return function (...args) {   
    clearTimeout(timer)
    timer=setTimeout(() => {
        func.apply(this,args)
      },wait)
  }
```


```js
function throttle(func,wait){
   let timer
   return function(...args){
      if(!timer){
       timer=setTimeout(()=>{
         func.apply(this,args)
         timer=null
       },wait)
   }
   }  
}
```

一个是先取消定时，一个是!timer判断定时器内执行后才转空



