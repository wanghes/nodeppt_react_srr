title: 同构直出演示
speaker: haisong.wang
url: https://github.com/wanghes
transition: slide3
theme: moon
usemathjax: yes

[slide]
# WULI 内容服务-大前端技术栈大纲
## 存在的开发的方式（暗指前端能接触到的开发方向）
----
* 开发基础的页面div+css {:&.rollIn}
* 借助js周边生态开发前端功能需求
* 基于nodejs用作中间层进行转发
* 同构直出
* ...
[slide]
## 前后分离&同构直出
----
直出是什么？到底是怎样的性能优化？我们将结合从在浏览器输入url，到展示最终页面的过程来对其进行一步步分析，并将在该技术的实际应用实践进行总结。

[slide]
## 模式 1 - 前后端分离
----
从用户输入url到展示最终页面的过程，这种模式可简单的分为以下 5 部分
1. 用户输入 url，开始拉取静态页面 {:&.rollIn}
2. 静态页面加载完成后，解析文档标签，并开始拉取 CSS （一般 CSS 放于头部）
3. 接着拉取 JS 文件（一般 JS 文件放于尾部）
4. 当 JS 加载完成，便开始执行 JS 内容，发出请求并拿到数据
5. 将数据与资源渲染到页面上，得到最终展示效果

[slide]
<div class="columns1">
    <img src="/img/1.png">
</div>

[slide]
## 模式 2 - 数据直出
数据请求在server端上提前获取，并和html一同返回，页面模板和数据的渲染在浏览器端上执行

[slide]
## 那么
1. 用户输入 url ，在 server 返回 HTML 前去请求获取页面需要的数据 {:&.rollIn}
2. 将数据拼接到 HTML 上 并 一起返回给前端
（可以插入 script 标签将数据添加到全局变量上，或放到某个标签的 data 属性中，如 ）
3. 在前端的JS代码中判断是否已在服务端拿到数据，直接拿该数据进行渲染页面，不再做数据请求

[slide]
<div class="columns1">
    <img src="/img/2.png">
</div>

[slide]
## 对比1和2模式
----


<pre><code class="markdown">
这种模式2与模式1 相比，减少了这两种模式请求数据的耗时差距。这块差距有多少呢？
## 发起一个 HTTP 的网络请求过程

DNS解析（100~200ms可以缓存）
         |
         |
        建立TCP链接 (三次握手100~200ms )
                |
                |
            HTTP Request( 半个RTT )
                   |
                   |
              HTTP Response( RTT 不确定优化空间 )

注: RTT 为 Round-trip time 缩写，表示一个数据包从发出到返回所用的时间。            
</code>
</pre>

[slide]
## HTTP 请求在前后端发出，差距有多少？
由上面对 HTTP 的网络请求过程可看到建立一次完整的请求返回在耗时上明显的，特别是外网用户在进行 HTTP 请求时，由于网络等因素的影响，在网络连接及传输上将花费很多时间。而在服务端进行数据拉取，即使同样是 HTTP 请求，由于后端之间是处于同一个内网上的，所以传输十分高效，这是差距来源的大头，是优化的刚需。


[slide]
## 模式 3 - 直出 (服务端渲染)
数据请求在server端上提前获取，页面模板结合数据的渲染处理也在server上完成，输出最终 HTML
模式 2 中将依赖于JS文件加载回来才能去发起的数据请求挪到 server 中，数据随着 HTML 一并返回。然后等待 JS 文件加载完成，JS 将服务端已给到的数据与HTML结合处理，生成最终的页面文档。

[slide]
<pre><code class="markdown">
数据请求能放到 server 上，对于数据与HTML结合处理也可以在server上做，
从而减少等待 JS 文件的加载时间。 这就是模式3 - 直出 (服务端渲染)，
主要处理如下
</code></pre>
----
1. server 上获取数据并将数据与页面模板结合，在服务端渲染成最终的 HTML {:&.rollIn}
2. 返回最终的 HTML 展示



[slide]
<pre><code class="markdown">
可以从下图看出，页面的首屏展示不再需要等待 JS 文件回来，
优化减少了这块时间
</code></pre>
----
<div class="columns1">
    <img src="/img/3.png" height="200">
</div>

[slide]
<pre><code class="markdown">
通过以上模式，将模式 1 - 常用模式中的第 3 和 4 点耗时进行了优化，
那么可以再继续优化吗？在页面文档不大情况下，可将CSS内联到HTML中，
这是优化请求量的做法。直出稍微不同的是需要考虑的是服务端最终渲染
出来的文档的大小，在范围内也可将 CSS 文件内联到 HTML 中。
这样的话，便优化了 CSS 的获取时间，
如下图
</code></pre>

<div class="columns1">
    <img src="/img/4.png" height="200">
</div>


[slide]
## 总结
-----
<pre><code class="markdown">
直出能够将常用模式优化到剩下了一次 HTML 请求，加快首屏渲染时间，
使用服务端渲染，还能够优化前端渲染难以克服的 SEO 问题。而不管是
简单的 数据直出 或是 服务端渲染直出 都能使页面的性能优化得到
较大提高
</code></pre>

[slide]
## 用nodejs实现同构直出

[slide]
Node 驾着祥云腾空而来，谷歌 V8 引擎给力支持，众前端拿着看家本领(JavaScript)开始涉足服务端，于是服务端渲染上又一步进阶
> 浏览器渲染首屏直接向node服务器发起请求，然后通过内网获取到首屏数据后，组装成HTML直接返回给浏览器。
同构就是解决直出的一种思想，node出现后使得javascript脚本也可以在服务器端执行，通过维护一套项目代码，实现在前后端都可以执行的目的。



[slide]
## 同构直出三要素
<div class="columns1">
    <img src="/img/7.png" height="200">
</div>
* 保证DOM的一致性 {:&.rollIn}
* 保证前后端数据的一致性
* 保证路由的一致性


[slide]
## 前后同构
----
<pre><code class="markdown">
有了Node 后，前端便有了更多的想象空间。前端框架开始考虑兼容服务端渲染，
提供更方便的 API，前后端共用一套代码的方案，让服务端渲染越来越便捷。
当然，不只是 React 做了这件事，但 React 将这种思想推向高潮，同构
的概念也开始广为人传。
可以查看阮老师的react-demos https://github.com/ruanyf/react-demos
另一篇文章 React 数据流管理架构之Redux https://github.com/joeyguo/blog/issues/3
</code></pre>
<div class="columns1">
    <img src="/img/6.png" height="200">
</div>



[slide]
<pre><code class="markdown">
我们以react为实例，其实可以选择任何框架
同构直出是一种优化的思想，不受任何框架限制，理解其中的原理才是最重要的。
那么问题就来了，如何使用react来保证dom一致性，又如何使用redux保证数
据一致性？先来看一下dom一致性的实现。
</code></pre>

<div class="columns-2">
    <img src="/img/8.png" height="200">
    <img src="/img/9.png" height="200">
</div>

[slide]
<pre><code class="markdown">
在使用react做同构直出时，很关键的一个因素就是它提供了虚拟DOM的支持，
是一种在内存中的对象数，使其可以支持在浏览器和node环境下执行，这也是
代码可以同构的关键所在。在浏览器端通过render方法生成虚拟dom并挂载到
真实DOM上。在服务端通过renderToString方法将虚拟dom拼装成HTML字符
串。使用这两个方法就可以解决dom一致性的问题了，来看一下具体的实现。
</code></pre>

<div class="columns-2">
    <img src="/img/10.png" height="100">
</div>
<pre><code class="markdown">
首先服务端通过调用rendertostring方法将react组件渲染为html字符串，
但是通过react组件渲染出来的并不是标准的html格式，需要将其嵌入HTML
模板中才能够被浏览器解析。当浏览器向直出服务器发起请求后，服务端将渲
染好的html字符串返回，浏览器收到响应后进行渲染。浏览器通过解析html
拉取到js脚本后，会执行render方法，在render方法处理过程中会校验节点
中的checksum属性，该属性是在服务端调用rendertostring方法时追加的，
用于前端校验dom一致性，当校验一致时，直接执行脚本中后续的绑定事件等行为，
如果不一致，将会进行虚拟DOM的diff操作，然后再进行增量更新DOM、绑定事件。
在红框处，可以看到同构代码的部分。
</code></pre>


[slide]
## 数据一致性是如何保证的
----
<pre><code class="markdown">
Redux使用单一的Store对象保存、管理页面中的所有状态，和虚拟dom一样，
是一种驻在内存中的对象，代码完全可以同构。
保证数据一致性的原理其实很简单。只要在最后组装HTML字符串时，将服务端的状
态通过script标签一起输出给前端，然后在前端初始化 Store 时使用该数据，
即可完成了数据的传递和共享，达到保证数据一致性的目的。
</code></pre>
<div class="columns-2">
    <img src="/img/11.png" height="200">
    <img src="/img/12.png" height="200">
</div>


[slide]
## 我们又该怎样保证代码的同构呢？
<pre><code class="markdown">
Node环境和浏览器环境毕竟还是不一样的，有这么多前端代码是不能直接在node端执行的，
应该怎样在同构代码上做好平台区分呢，发送异步请求前端使用的是ajax方法，node端使用
的是http模块的request方法，这个问题怎么解决？
</code></pre>

* webpack帮我们做了这件事
<div class="columns-2">
    <img src="/img/13.png" height="200">
</div>
<pre><code class="markdown">
借助这种思考方式，通过构建工具处理，就不需要对源码进行任何更改。源码中使用的
是ajax方法，同时在node服务器上在全局变量下实现了一个window.ajax的方法，这样
通过自定义babel插件，在对源码打包时，将ajax方法名替换成为window.ajax方法名，
问题就得到了解决。
</code></pre>


[slide]
## 谁在用同构直出
* 淘宝首页首屏直出渲染 {:&.fadeIn}
* 手Q的家校群首屏，
* QQ兴趣部落

[slide]
## 如何在我们的产品中实施同构
（h5的列表页面与详情页面）

[slide]
# thanks
