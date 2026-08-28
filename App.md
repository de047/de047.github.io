
> 关键技术点：
> HTTPS：头、查、体；  协议、路径
> HS256：哈希、密钥、hex
> 都要用到编码函数
> 都要用到json或JS对象：配置、数据、传递

其实这些东西都被HTTP协议用到

前Vue ，后Express ，中密码学，其它的不过是框架，包括AI应用也是一样

## 页面传值

```html
<navigator :url="'/pages/navigate/navigate?item='+ encodeURIComponent(JSON.stringify(item))"></navigator>
```

```js
// navigate.vue页面接受参数
onLoad: function (option) {
	const item = JSON.parse(decodeURIComponent(option.item));
}
```

## wxml

app.json   pages.json  project.*.json home.json

App对象和Page对象，均有生命周期

按钮绑定bind:tap="函数名"  页面.js中定义该函数传event

this.setData()和this.data

```js
Page({ 
	data:{ 
		obj = {a:1,b:2}
		arr = [{c:3,d:4},{c:5,d:6}]
	}, 
	changeC(){ 
		this.setData({ 
			arr[1].d: this.data.arr[1].d+1
		})
	}
})
```

页面初始化从云端获取一个对象数组
页面接收一个对象数组
输入框、数字按钮、选择按钮等表单元素绑定到对象数组的对象的某一个具体key，用户交互时自动更新
arr → index，item → key , value 

```js
wx:for="{{arr}}" 
	{{index}} {{item}}
	wx:for="{item}"
		{{key}}{{value}}
		input model:value="{{arr[index][key]}}"
```

## Vue

[演练场](https://play.vuejs.org/)

### 响应性

这是最最重要的核心，vue的其它都不重要，响应性最重要

ref()可传对象或数组或它们的嵌套，obj.a.b也是响应式的

数据流向：ref()→onMounted设置.value→@click设置.value→@input设置.value→可选watcheffect副作用→界面更新

`watchEffect(()=>{})`监听任何响应式变量的改动，自动执行，一个响应式变量依赖另一个时有用

响应式变量改变的时机：
- onload页面url传参、网络请求
- @click、@input等交互

### v-model

v-model非常非常重要，既可用来同步表单输入，亦可同步父子组件

v-model的本质是@input

注意小技巧，v-model亦可为响应式对象的键`v-model="obj[key]"`

- 绑定文本或value：input、radio、单select
- 绑定布尔：单checkbox
- 绑定数组：多checkbox、多select

```
 <select v-model="choice" >
    <option>A</option>
    <option>B</option>
  </select>
```


###  文本插值

注意，ref要先引入，这个容易忘

```
import {ref} from 'vue'
const obj = ref({a:0})
   
{{ obj }}

```

注意注意，务必注意！在模板中的变量一定不能写`.value`，在脚本中的变量一定要写`.value`


### v-for

渲染数组：

```
v-for="(item,index) in arr" :key="item.id"
```

渲染对象：

```
v-for="(value, key) in obj"
```

### v-bind

`v-bind:属性` → `:属性`

根据变量或表达式的值控制元素的属性，可嵌套


内联style插值，注意数字值和文本值的规范，font-size这种写成fontSize：

```html
<标签 :style="{color:变量1, fontSize:变量2+'px' }">内容</标签>
```

事件`@click='函数名'`

### v-if

```
  v-if="条件表达式"
  紧跟v-else
```

### 自定义属性


```html
const { obj } = defineProps(['obj'])
```

像原生html属性一样传值，动态属性加冒号，静态不加

注意注意，无论是组件名还是属性名，定义时若大小驼峰，则使用时小写连字符
