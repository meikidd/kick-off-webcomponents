# Kick Off Web Components

跟 Web Components 打个啵

## What are Web Components

- [Web Components](http://www.w3.org/TR/components-intro/) 是 W3C 定义的新标准，目前还在草案阶段。

## Why are they important
- 前端组件化 
  - bootstrap

  ```
  	// 初始化
	$('#myModal').modal({
	  keyboard: false
	});

	// 显示
	$('#myModal').modal('show');

	// 关闭事件
	$('#myModal').on('hidden.bs.modal', function (e) {
	  // do something...
	});
  ```
  
  - atom

  ```
  // 初始化组件
  var dialog = new Dialog(
  	 trigger: '#trigger-btn',
	 title: '我是自定义的标题',
	 content: 'hello world',
	 buttons: ['submit', 'cancel']
  });
  
  // 显示
  dialog.show();
  
  // 关闭事件
  dialog.after('hide', function() {
  	 // do something...
  });
        
  ```
- 统一标准、减少轮子
- 简化代码，提高可维护性
	![gmail](http://www.html5rocks.com/zh/tutorials/webcomponents/customelements/gmail.png)

	```
	<hangout-module>
	  <hangout-chat from="Paul, Addy">
	    <hangout-discussion>
	      <hangout-message from="Paul" profile="profile.png"
	           datetime="2013-07-17T12:02">
	        <p>Feelin' this Web Components thing.</p>
	        <p>Heard of it?</p>
	      </hangout-message>
	    </hangout-discussion>
	  </hangout-chat>
	  <hangout-chat>...</hangout-chat>
	</hangout-module>
	```
	
## 关键技术
- HTML Imports
- HTML Templates
- Custom Elements
- Shadow DOM

### HTML Imports
通过`<link>`标签来引入 HTML 文件，使得我们可以用不同的物理文件来组织代码。

```
<link rel="import" href="http://example.com/component.html" >
```
> 注意：受浏览器同源策略限制，跨域资源的 import 需要服务器端开启 CORS。
> `Access-Control-Allow-Origin: example.com`

通过`import`引入的 HTML 文件是一个包含了 html, css, javascript 的独立 component。

```
<template>
    <style>
        .coloured {
            color: red;
        }
    </style>
    <p>My favorite colour is: <strong class="coloured">Red</strong></p>
</template>
<script>
    (function() {
        var element = Object.create(HTMLElement.prototype);
        var template = document.currentScript.ownerDocument.querySelector('template').content;
        element.createdCallback = function() {
            var shadowRoot = this.createShadowRoot();
            var clone = document.importNode(template, true);
            shadowRoot.appendChild(clone);
        };
        document.registerElement('favorite-colour', {
            prototype: element
        });
    }());
</script>
```

### HTML Templates
关于 HTML 模板的作用不用多讲，用过 mustache、handlbars 模板引擎就对 HTML 模板再熟悉不过了。但原来的模板要么是放在 `script` 元素内，要么是放在 `textarea` 元素内，HTML 模板元素终于给了模板一个名正言顺的名分: `<template>`  

原来的模板形式：

- script 元素

```
<script type="text/template">
	<div>
		this is your template content.
	</div>
</script>
```
- textarea 元素

```
<textarea style="display:none;">
	<div>
		this is your template content.
	</div>
</textarea>
```

现在的模板形式：

- template 元素

```
<template>
	<div>
		this is your template content.
	</div>
</template>
```

### Custom Elements
自定义元素允许开发者定义新的 HTML 元素类型。带来以下特性：

1. 定义新元素
2. 元素继承
3. 扩展原生 DOM 元素的 API

#### 定义新元素
使用 `document.registerElement()` 创建一个自定义元素：

```
var Helloworld = document.registerElement('hello-world', {
  prototype: Object.create(HTMLElement.prototype)
});

document.body.appendChild(new Helloworld());
```
- 标签名必须包含连字符 ' - '
	- 合法的标签名：`<hello-world>`, `<my-hello-world>`
	- 不合法的标签名：`<hello_world>`, `<HelloWorld>`

#### 元素继承
如果 `<button>` 元素不能满足你的需求，可以继承它创建一个新元素，来扩展 `<button>` 元素：

```
var MyButton = document.registerElement('my-button', {
  prototype: Object.create(HTMLButtonElement.prototype)
});
```

#### 扩展原生 API

```
var MyButtonProto = Object.create(HTMLButtonElement.prototype);

MyButtonProto.sayhello = function() {
  alert('hello');
};

var MyButton = document.registerElement('my-button', {
  prototype: MyButtonProto
});


var myButton = new MyButton();
document.body.appendChild(myButton);

myButton.sayhello(); // alert: "hello"
```

#### 实例化
使用 `new` 操作符：

```
var myButton = new MyButton();
myButton.innerHTML = 'click me!';
document.body.appendChild(myButton);
```

或，直接在页面插入元素：

```
<my-button>click me!</my-button>
```

#### 生命周期
元素可以定义特殊的方法，来注入其生存周期内的关键时间点。生命周期的回调函数名称和时间点对应关系如下：

- createdCallback: 创建元素实例时
- attachedCallback: 向文档插入实例时
- detachedCallback: 从文档移除实例时
- attributeChangedCallback(attrName, oldVal, newVal): 添加，移除，或修改一个属性时

```
var MyButtonProto = Object.create(HTMLButtonElement.prototype);

MyButtonProto.createdCallback = function() {
  this.innerHTML = 'Click Me!';
};

MyButtonProto.attachedCallback = function() {
  this.addEventListener('click', function(e) {
    alert('hello world');
  });
};

var MyButton = document.registerElement('my-button', {
  prototype: MyButtonProto
});

var myButton = new MyButton();
document.body.appendChild(myButton);
```

### Shadow DOM
web 开发经典问题：封装。如何保护组件的样式不被外部 css 样式侵入，如何保护组件的 dom 结构不被页面的其他 javascript 脚本修改。

- dom 结构的封装
- css 的封装
- javascript 的封装

Shadow Dom 可以很好的解决组件封装问题。

类似 
<input type="range">

![](http://g02.a.alicdn.com/kf/HTB1laKYIpXXXXc8XVXX760XFXXXd.png)