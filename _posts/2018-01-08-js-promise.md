---
layout: post
title: 揭开Promise的神秘面纱 - JavaScript
date: 2018-01-08 18:32:24.000000000 +09:00
tag: React-Native
---

<h4>目录</h4>

    一、前言
    二、Promise是什么
    三、Promise有什么用
    四、Promise的特点
    五、Promise基本用法
    六、Promise的两个快捷创建方式
        1 Promise.resolve()
        2 Promise.reject()
    七、Promise操作多个异步事件
        1 Promise.all()
        2 Promise.race()
    八、Promise处理同步操作
    九、Promise的链式调用
    10 十、Promise的缺点

<h4>一、前言</h4>
文章描述本人在学习JavaScript时，对<code>Promise</code>语法的学习、摘抄和理解，其中大部分内容都是借鉴下面两篇文章的。也总结了目前项目中<code>Promise</code>的用法，以最适合理解的方式记录下来，算是一个入门文章，以防忘记，也供他人参考。如有雷同，纯属我抄他们的！:blush:
<blockquote>
  参考链接：
1.《ECMAScript 6 入门》- 阮一峰：http://es6.ruanyifeng.com/#docs/promise
2.《JavaScript Promise迷你书（中文版）》：http://liubin.org/promises-book/</blockquote>
<h4>二、Promise是什么</h4>
<code>Promise</code>最初被提出是在<code>E语言</code>中， 它是基于并列/并行处理设计的一种编程语言，是异步编程的一种解决方案。

现在JavaScript也拥有了这种特性，最早由社区提出和实现，比传统的解决方案——回调函数和事件——更合理和更强大，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。
<h4>三、Promise有什么用</h4>
<code>Promise</code>主要用用来代替回调函数，进行异步处理。

使用回调函数的异步处理：
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">getAsync("fileA.txt", function(error, result){
        if(error){
            return error; // 取得失败时的处理
        } else {
            return result; // 取得成功时的处理
        }
})
</code></pre>
<blockquote>
  Node.js等则规定在JavaScript的回调函数的第一个参数为 Error 对象，这也是它的一个惯例。</blockquote>
像上面这样基于回调函数的异步处理如果统一参数使用规则的话，写法也会很明了。 但是，这也仅是编码规约而已，即使采用不同的写法也不会出错。

而<code>Promise</code>则是把类似的异步处理对象和处理规则进行规范化， 并按照采用统一的接口来编写，而采取规定方法之外的写法都会出错。

下面是使用了Promise进行异步处理的一个例子：
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">var promise = getAsyncPromise("fileA.txt"); 
promise.then(function(result){
    // 获取文件内容成功时的处理
}).catch(function(error){
    // 获取文件内容失败时的处理
});
</code></pre>
我们可以向这个预设了抽象化异步处理的<code>Promise</code>对象， 注册这个<code>Promise</code>对象执行成功时和失败时相应的回调函数。

这和回调函数方式相比有哪些不同之处呢？ 在使用<code>Promise</code>进行一步处理的时候，我们必须按照接口规定的方法编写处理代码。

也就是说，除<code>Promise</code>对象规定的方法(这里的 <code>then</code> 或 <code>catch</code> )以外的方法都是不可以使用的， 而不会像回调函数方式那样可以自己自由的定义回调函数的参数，而必须严格遵守固定、统一的编程方式来编写代码。

这样，基于<code>Promise</code>的统一接口的做法， 就可以形成基于接口的各种各样的异步处理模式。

所以，<code>Promise</code>的功能是可以将复杂的异步处理轻松地进行模式化， 这也可以说得上是使用<code>Promise</code>的理由之一。
<h4>四、Promise的特点</h4>
简单来说，<code>Promise</code>就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作，但也可以是同步）的结果。

特点：
<code>Promise</code>对象有以下两个特点：

<strong>1.对象的状态不受外界影响</strong>。
<code>Promise</code>对象代表一个异步操作，有三种状态：<code>pending</code>（进行中）、<code>fulfilled</code>（已成功）和<code>rejected</code>（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

<strong>2.一旦状态改变，就不会再变，任何时候都可以得到这个结果</strong>。
<code>Promise</code>对象的状态改变，只有两种可能：从<code>pending</code>变为<code>fulfilled</code>和从<code>pending</code>变为<code>rejected</code>。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为<code>resolved</code>（已定型）。
也就是说，Promise与Event等不同，当Promise的对象状态发生变化时，在<code>.then</code>后执行的函数可以肯定地说只会被调用一次。
<h4>五、Promise基本用法</h4>
ES6 规定，<code>Promise</code>对象是一个构造函数，用来生成<code>Promise</code>实例。

下面代码创造了一个<code>Promise</code>实例。
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">const promise = new Promise(function(resolve, reject) {
  // ... some code
  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
</code></pre>
<code>Promise</code>构造函数接受一个函数作为参数，该函数的两个参数分别是<code>resolve</code>和<code>reject</code>。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。

<code>resolve</code>函数的作用是，将<code>Promise</code>对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；

<code>reject</code>函数的作用是，将<code>Promise</code>对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

<code>Promise</code>实例生成以后，可以用<code>then</code>方法分别指定<code>resolved</code>状态和<code>rejected</code>状态的回调函数。
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
</code></pre>
<blockquote>
  注意：
const promise = new Promise(function(param1, param2) { }
function(resolve, reject){ }函数只接收两个参数，两个参数有严格的顺序，第一个参数表示成功，第二个参数表示失败。
如果设置第三个参数，例如param3，调用param3时会无响应。
调用顺序跟参数名字没有关系，形参名字可以随便取。
也就是说：如果异步执行成功了，那就用第一个参数param1返回；
如果异步执行失败了，就用第二个参数param2返回。</blockquote>
<code>.then</code>函数有两个参数，第一个参数是<code>resolve</code>函数，第二个是<code>reject</code>函数。
代码如下：
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">promise.then((result)=&gt;{
            // 成功
        }, (error)=&gt;{
            // 异常
})
</code></pre>
<code>.catch</code>方法是<code>.then(null, rejection)</code>的别名，用于指定发生错误时的回调函数。在发生异常时，可以使用<code>.then</code>方法的<code>reject</code>函数捕获，也可以使用<code>.catch</code>捕获，这种属于链式调用。

<code>Promise</code>对象异步执行完毕，在由<code>pending</code>状态，改变为<code>resolve</code>状态时，会调用<code>.then</code>函数，也就是说<code>.then</code>函数中，可以处理成功状态，也可以处理失败状态。但是，一般情况下推荐只在<code>.then</code>中处理成功回调，而在<code>.catch</code>中处理失败回调，在<code>.then</code>后使用<code>.catch</code>处理异常属于链式调用。

<strong>这样做的好处是：</strong>
把<code>.catch</code>放在<code>.then</code>后，在<code>.then</code>里发生的错误，在<code>.catch</code>中也可以捕获。

<strong><code>Promise</code>对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个<code>catch</code>语句捕获。</strong>
<h4>六、Promise的两个快捷创建方式</h4>
一般情况下我们都会使用 <code>new Promise()</code> 来创建promise对象，但是除此之外我们也可以使用其他方法。

分别是 <code>Promise.resolve</code> 和 <code>Promise.reject</code> 这两个方法：
<h5>Promise.resolve()</h5>
静态方法Promise.resolve(value) 可以认为是 <code>new Promise()</code> 方法的快捷方式。

<code>Promise.resolve(value)</code>的参数分四种情况：
①参数是一个 Promise 实例；
如果参数是 Promise 实例，那么<code>Promise.resolve</code>将不做任何修改、原封不动地返回这个实例。

②参数是一个<code>thenable</code>对象；
<code>Promise.resolve</code>方法会将这个对象转为 Promise 对象，然后就立即执行<code>thenable</code>对象的<code>then</code>方法。

③参数不是具有<code>then</code>方法的对象，或根本就不是对象；
如果参数是一个原始值，或者是一个不具有<code>then</code>方法的对象，则<code>Promise.resolve</code>方法返回一个新的 Promise 对象，状态为<code>resolved</code>。

④不带有任何参数。
<code>Promise.resolve</code>方法允许调用时不带参数，直接返回一个<code>resolved</code>状态的 Promise 对象。
所以，如果希望得到一个 Promise 对象，比较方便的方法就是直接调用<code>Promise.resolve</code>方法。

比如 Promise.resolve(42); 可以认为是以下代码的语法糖。
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">new Promise(function(resolve){
    resolve(42);
});
</code></pre>
在这段代码中的 resolve(42); 会让这个promise对象立即进入确定（即resolved）状态，并将 42 传递给后面then里所指定的 <code>onFulfilled</code> 函数。

注意：虽然是立即进入<code>resolved</code>状态，但是<code>.then</code>中指定的方法调用是异步进行的，看下面的示例：
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">var promise = new Promise(function (resolve){
    console.log("inner promise"); // 1
    resolve(42);
});
promise.then(function(value){
    console.log(value); // 3
});
console.log("outer promise"); // 2
</code></pre>
Promise被创建时，会立即<code>同步</code>执行其参数中的function函数，function可以是异步函数，也可以是同步函数，等待function函数异步执行完毕，返回结果后，成功则调用resolve方法（异步调用），失败则调用reject方法（异步调用）。

所以：上面例子的调用顺序是，1 -&gt; 2 -&gt; 3， 而不是 2 -&gt; 1 -&gt; 3，也不是1 -&gt; 3 -&gt; 2。

同步的先执行完毕， 然后执行微任务， 接着是宏任务。
<blockquote>
  JS任务队列：
JS中，仅支持单线程，有两类任务队列：<code>宏任务队列</code>（macro tasks）和<code>微任务队列</code>（micro tasks）。宏任务队列可以有多个，微任务队列只有一个。那么什么任务，会分到哪个队列呢？
宏任务：<code>script（全局任务）</code>, <code>setTimeout</code>, <code>setInterval</code>, <code>setImmediate</code>, <code>I/O</code>, <code>UI rendering</code>.
微任务：<code>process.nextTick</code>, <code>Promise</code>, <code>Object.observer</code>, <code>MutationObserver</code>.
我们上面讲到，当js任务栈空的时候，就会从任务队列中，取任务来执行。共分3步：
取一个宏任务来执行。执行完毕后，下一步。
取一个微任务来执行，执行完毕后，再取一个微任务来执行。直到微任务队列为空，执行下一步。
更新UI渲染。</blockquote>
<h5>Promise.reject()</h5>
<code>Promise.reject(error)</code>是和 <code>Promise.resolve(value)</code> 类似的静态方法，是 <code>new Promise()</code> 方法的快捷方式。

比如 <code>Promise.reject(new Error("出错了"))</code> 就是下面代码的语法糖形式。
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">new Promise(function(resolve,reject){
    reject(new Error("出错了"));
});
</code></pre>
这段代码的功能是调用该promise对象通过<code>then</code>指定的 <code>onRejected</code> 函数，并将错误（Error）对象传递给这个 <code>onRejected</code> 函数。

<code>Promise.reject</code> 在编写测试代码或者进行debug时，比较方便。
<h4>七、Promise操作多个异步事件</h4>
<h5>Promise.all()</h5>
<code>Promise.all</code>方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">const p = Promise.all([p1, p2, p3]);
</code></pre>
上面代码中，<code>Promise.all</code>方法接受一个数组作为参数，p1、p2、p3都是 Promise 实例，如果不是，就会先调用上面讲到的<code>Promise.resolve</code>方法，将参数转为 Promise 实例，再进一步处理。（<code>Promise.all</code>方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。）

p的状态由p1、p2、p3决定，分成两种情况。

（1）只有p1、p2、p3的状态都变成<code>fulfilled</code>，p的状态才会变成<code>fulfilled</code>，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

（2）只要p1、p2、p3之中有一个被<code>rejected</code>，p的状态就变成<code>rejected</code>，此时第一个被<code>reject</code>的实例的返回值，会传递给p的回调函数。

<code>Promise.all</code>的各元素之间相当于<code>与</code>的关系。
<h5>Promise.race()</h5>
<code>Promise.race</code>方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">const p = Promise.race([p1, p2, p3]);
</code></pre>
上面代码中，只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。

<code>Promise.race</code>方法的参数与<code>Promise.all</code>方法一样，如果不是 Promise 实例，就会先调用上面讲到的<code>Promise.resolve</code>方法，将参数转为 Promise 实例，再进一步处理。

<code>Promise.race</code>的各元素之间相当于<code>或</code>的关系。
<h4>八、Promise处理同步操作</h4>
实际开发中，经常遇到一种情况：不知道或者不想区分，函数<code>f</code>是同步函数还是异步操作，但是想用 Promise 来处理它。因为这样就可以不管<code>f</code>是否包含异步操作，都用<code>then</code>方法指定下一步流程，用<code>catch</code>方法处理<code>f</code>抛出的错误。

有两种写法：
第一种：用<code>async</code>函数来写：
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">const f = () =&gt; console.log('now');
(async () =&gt; f())();
console.log('next');
// now
// next
</code></pre>
上面代码中，第二行是一个立即执行的匿名函数，会立即执行里面的<code>async</code>函数，因此如果<code>f</code>是同步的，就会得到同步的结果；如果<code>f</code>是异步的，就可以用<code>then</code>指定下一步，就像下面的写法，用<code>catch</code>捕捉异常。
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">(async () =&gt; f())()
.then(...)
.catch(...)
</code></pre>
第二种：使用<code>new Promise()</code>：
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">const f = () =&gt; console.log('now');
(
  () =&gt; new Promise(
    resolve =&gt; resolve(f())
  )
)();
console.log('next');
// now
// next
</code></pre>
上面代码也是使用立即执行的匿名函数，执行<code>new Promise()</code>。这种情况下，同步函数也是同步执行的。
<h4>九、Promise的链式调用</h4>
<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">function doubleUp(value) {
    return value * 2;
}
function increment(value) {
    return value + 1;
}
function output(value) {
    console.log(value);// =&gt; (1 + 1) * 2
}

var promise = Promise.resolve(1);
promise
    .then(increment)
    .then(doubleUp)
    .then(output)
    .catch(function(error){
        // promise chain中出现异常的时候会被调用
        console.error(error);
    });
</code></pre>
上面示例的每个方法中 return 的值不仅只局限于字符串或者数值类型，也可以是对象或者promise对象等复杂类型。

return的值会由 <code>Promise.resolve</code>(return的返回值) 进行相应的包装处理，因此不管回调函数中会返回一个什么样的值，最终 <code>then</code> 的结果都是返回一个新创建的promise对象。

也就是说， <code>Promise.then</code> 不仅仅是注册一个回调函数那么简单，它还会将回调函数的返回值进行变换，创建并返回一个promise对象。

从代码上乍一看，<code>aPromise.then(...).catch(...)</code>像是针对最初的 <code>aPromise</code> 对象进行了一连串的方法链调用。

然而实际上不管是 <code>then</code> 还是 <code>catch</code> 方法调用，都返回了一个新的promise对象。
<h4>十、Promise的缺点</h4>
<code>Promise</code>也有一些缺点。首先，无法取消<code>Promise</code>，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，<code>Promise</code>内部抛出的错误，不会反应到外部。第三，当处于<code>pending</code>状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。