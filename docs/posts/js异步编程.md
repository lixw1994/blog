---
draft: false
comments: true
slug: javascript-async-programming
date: 2021-06-18
authors: [lixw1994]
title: Javascript 异步编程
description: >
  Javascript 异步编程
categories:
  - General
  - Tech
tags:
  - Javascript
  - 异步编程
---

# Javascript 异步编程
一个令人震惊的事实：JavaScript本身从来没有任何内建的异步概念。
<!-- more -->

## 事件轮询（Event Loop）

- JavaScript本身是单线程的。
- JS引擎本身除了在某个在被要求的时刻执行你程序的一个单独的代码块外，没有做过任何其他的事情。
	- JS引擎宿主环境：浏览器、Node、cocos-js等
- 一段假想的事件轮询的代码
	```
	// `eventLoop`是一个像队列一样的数组（先进先出）
	var eventLoop = [ ];
	var event;
	
	// “永远”执行
	while (true) {
		// 执行一个"tick"
		if (eventLoop.length > 0) {
			// 在队列中取得下一个事件
			event = eventLoop.shift();
	
			// 现在执行下一个事件
			try {
				event();
			}
			catch (err) {
				reportError(err);
			}
		}
	}
	```
- 我们的程序通常被打断成许多小的代码块儿，它们一个接一个地在事件轮询队列中执行。
- 可以这么理解：异步是关于 现在 与 稍后 之间的间隙

## 异步与callback
- 最常见的 代码块儿 单位是`function`，所以最简单的异步方式就是回调函数。
- 例如一个异步的Ajax请求
```
// ajax(..) 是某个包中任意的Ajax函数
ajax( "http://some.url.1", function callback(data){

	console.log( data ); // 稍后，我得到了一些数据

} );
console.log('请求开始');  // 现在，请求开始
```

- 这个输出的顺序是什么
```
function asyncAdd(a, b, cb) {
    setTimeout(cb, 0, a+b);
}

asyncAdd(3, 3, function(c) {
    console.log(c);
});

asyncAdd(1, 1, function(c) {
    console.log(c);
});

```

- 这个输出的顺序是什么
```
function add(a, b, cb) {
    cb(a + b);
}

function asyncAdd(a, b, cb) {
    setTimeout(cb, 0, a+b);
}



asyncAdd(3, 3, function(c) {
    console.log(c);
});

add(2, 2, function(c) {
    console.log(c);
});

asyncAdd(1, 1, function(c) {
    console.log(c);
});
// callback一定是异步吗
```

## 异步与并行
以下代码，如果可以并行执行，a可能的值是什么
实际js运行时，a可能的值是什么
```
// 随机延时
function ajax(url, cb) {
    setTimeout(cb, Math.random() * 100, url);
}

// 实际运行代码
let a = 20;

function foo() {
	a = a + 1;
}

function bar() {
	a = a * 2;
}
// 42 41
// ajax(..) 是一个给定的库中的随意Ajax函数
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
// 实际运行代码

// 输出a
setTimeout(() => {
    console.log(a);
}, 1000);
```
- js并不具备并行的能力
- 一旦某个代码块开始执行，必然"运行至完成"

## 回掉地狱(callback hell)
考虑下面的代码，分析执行顺序：
```
listen( "click", function handler(evt){
	setTimeout( function request(){
		ajax( "http://some.url.1", function response(text){
			if (text == "hello") {
				handler();
			}
			else if (text == "world") {
				request();
			}
		} );
	}, 500) ;
} );
```
简化版
```
doA( function(){
	doB();

try
	doC( function(){
		doD();
	} )

	doE();
} );

doF();
```
doA()
doF()
doB()
doC()
doE()
doD()

- 注意：上述分析一定对吗？一切皆异步，这是一个承诺，还是一个事实？
- 回掉的信任问题
	- 回掉意味着转移控制权 - 不能try catch
	- Release Zalgo？？(有时会同步完地成，而有时会异步地完成)
	- 调用回调过早（在它开始追踪之前）
	- 调用回调过晚 (或不调)
	- 调用回调太少或太多次（就像你遇到的问题！）
	- 没能向你的回调传递必要的环境/参数
	- 吞掉了可能发生的错误/异常
- 两种常见的尝试(没有解决本质问题)
	- `ajax( "http://some.url.1", success, failure );``
	- 错误优先风格
	```
	function response(err,data) {
		// 有错？
		if (err) {
			console.error( err );
		}
		// 否则，认为成功
		else {
			console.log( data );
		}
	}
	ajax( "http://some.url.1", response );
	```

## Promise风暴
- Promise就是一个稍后的值
	- 一个
	- 稍后
	- 值(成功 or 拒绝)
- 一个 Promise 必然处于以下几种状态之一：
待定（pending）: 初始状态，既没有被兑现，也没有被拒绝。
已兑现（fulfilled）: 意味着操作成功完成。
已拒绝（rejected）: 意味着操作失败。
![bb754dbb921cebfeadb2f0367b23cbba.png](:/66d39343586245dca802b248dc970981)

- Promise + Generator => async + await；是最终的解决方案吗？


## Promise不是银弹
- 本质上还是回调函数
- 跟普通的js函数一样，不能打断和撤销
	- 设计幂等的代码块儿处理类似的问题
- 未捕获的错误如何处理 (尾部接上catch)
	- catch内部也有错误呢
- 缺乏处理事件流的能力
```
考虑这样一个场景：你想要在一个特定事件每次被触发时触发一系列步骤。一个单独的 Promise 或序列不能表示这个事件全部的发生状况。所以，你不得不为每一个事件的发生创建一个全新的 Promise 链（或序列），比如：

listener.on( "foobar", function(data){

	// 创建一个新的事件处理 Promise 链
	new Promise( function(resolve,reject){
		// ..
	} )
	.then( .. )
	.then( .. );

} );
```

## 额外甜点
### 同步和异步 ｜ 并发和并行
- 同步和异步是对任务执行过程的描述
	- 异步并不意味着程序性能更高
- 并发是对多个任务的计划(planning)
	- 并发并不意味着程序性能更高
		- Concurrency is powerful.
		- Concurrency is not parallelism.
		- Concurrency enables parallelism.
		- Concurrency makes parallelism (and scaling and everything else) easy.
- 并行是多个任务同时执行(doing)
	- [Concurrency is not parallelism](https://talks.golang.org/2012/waza.slide#8)
### generator
- [打破"运行至完成"定律](https://github.com/getify/You-Dont-Know-JS/blob/1ed-zh-CN/async%20%26%20performance/ch4.md#%E6%89%93%E7%A0%B4%E8%BF%90%E8%A1%8C%E8%87%B3%E5%AE%8C%E6%88%90)
- [CSP模式](https://github.com/getify/You-Dont-Know-JS/blob/1ed-zh-CN/async%20%26%20performance/apB.md#%E9%80%9A%E4%BF%A1%E5%BA%8F%E5%88%97%E5%8C%96%E5%A4%84%E7%90%86csp)
- [响应式编程](https://github.com/ReactiveX/rxjs)

## 引用
- `You Don't Know JS` 系列书籍
