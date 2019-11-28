## util
我们首先来看三个工具库文件 DOMUtils.js/LocationUtils.js/PathUtil.js 这三个文件
### DOMUtil.js
#### canUseDOM
这个值就是用来判断运行时是否能够使用 window.document.createElement
1. 首先判断 window 对象是否存在
2. 如果存在判断 window.document 是否存在
3. 如果存在则判断 window.document.createElement 是否存在
4. 最后加 !! 将判断结果转为 Boolean 值 因为 && 如果整体是 true 则返回 && 右边的值否则返回左边的值，所以需要 !! 转成 Boolean 值

window.document.createElement

参数: 

> tagName: 指定要创建元素类型的字符串，创建元素时的 nodeName(node.nodeName 只读属性储存了当前节点的节点名称字符串) 使用 tagName 值初始化，并且会将 tagName 转为小写，在Firefox, Opera 和 Chrome内核中. createElement(null) 等同于 createElement("null")

> options(可选): 一个包含属性名 is 的对象，如果这个对象被定义并赋予了一个 is 特性，则创建的element的 is 属性会被初始化为这个特性的值. 如果这个对象没有 is 特性，则值为空.

返回值: 新的元素
#### getConfirmation
这个函数就是让你弹个 confirm 框然后将弹框的message返回给你给的回调函数。非常简单

window.confirm

语法
```
result = window.confirm(message);
```
- message 是要在对话框中显示的可选字符串
- result 是一个 Boolean 值，表示是选择确定还是取消
#### supportsHistory
1. 获取当前用户代理字符串
2. 进行字符串匹配，如果是满足条件则是不支持 history 返回false
3. 如果不通过上面的 if 则判断是否存在 window.history.pushState 方法有则返回 true

window.history.pushState

浏览器不会向服务端请求数据，直接改变url地址，可以类似的理解为变相版的hash；但不像hash一样，浏览器会记录pushState的历史记录，可以使用浏览器的前进、后退功能作用

参数

> state: 可以通过 history.state 读取

> title: 可选参数，暂时没用，建议传个短标题

> url: 改变过后的 url 地址

window.history.replaceState

history.replaceState() 的使用与 history.pushState() 非常相似，区别在于  replaceState()  是修改了当前的历史记录项而不是新建一个。 注意这并不会阻止其在全局浏览器历史记录中创建一个新的历史记录项。

例子

如果此时你的地址是 http://ip/p/page
```
history.pushState({page: 1}, "title 1", "page1");
// 此时地址是 http://ip/p/page1 但是不会请求 并且浏览器记录了该历史记录(新建)
history.replaceState({page: 2}, "title 3", "/page2");
// 此时地址是 http://ip/page2 但是不会请求 并且浏览器记录了该历史记录(替换了http://ip/p/page1)
```
#### supportsPopStateOnHashChange
#### supportsGoWithoutReloadUsingHash
#### isExtraneousPopstateEvent
上面三个都是用来判断是否支持某个API的，到具体用到的时候我们会再介绍
### PathUtils.js
#### addLeadingSlash

1. 判断传入的 path 开头是否有 '/' 没有则加上返回

String.prototype.charAt()

从一个字符串中返回指定的字符。

参数：

> index：一个介于 0 和字符串长度减 1 之间的整数。如果没有提供索引则将使用 0

返回值：

返回指定的字符
#### stripLeadingSlash
1. 判断传入的path 第一个字符是否是 '/' 不是则直接返回，是则返回第一个字符后的字符

String.prototype.substr

参数：

> start: 开始提取字符的位置，如果是负值，则被看做 str.length + strat

> length(可选): 提取的字符数，如果是负值或0则返回空字符串，如果忽略，则提取字符直到字符串末尾。

#### hasBasename
忽略大小写匹配 传入的 path 是否是 prefix 后接 / 或者 # 或者 ? 或者直接结束。匹配到则返回true，否则返回false

注意：

如果是`new Regexp(pattern, flags)`创建的regexp pattern是字符串，如果想要匹配 '/' 要写成 '\\/' 因为 '\' 在字符串中需要先转义，如果是 /\// 这种写法就可以省略一个'\'

RegExp

语法：

字面量，构造函数和工厂符号都是可以的
```
new RegExp(pattern, (flags));
RegExp(pattern, (flags));
```

参数：

> pattern: 正则表达式文本

> flags: 如果指定标志可以具有以下值的任意组合

>> g: 全局匹配：找到所有匹配，而不是在第一个匹配后停止

>> i: 忽略大小写

>> m: 多行；将开始和结束字符（^和$）视为在多行上工作（也就是，分别匹配每一行的开始和结束 由 \n 或 \r 分割），而不只是匹配整个输入字符串的最开始和最末尾处

>> u: Unicode；将模式视为 Unicode 序列点的序列

>> y: 粘性匹配；仅匹配目标字符串中此正则表达式的 lastIndex 属性指示的索引

>> s: dotAll 模式,匹配任何字符

#### stripBasename
判断一下传入的path最后一个字符是否是 '/' 如果是则切割字符串返回去除最后一个字符的字符串，如果不是则直接返回path
#### parsePath
这个函数用来分割 路由 hash路由 查询参数的
1. 创建 pathname 并赋值传入的 path 如果 path 是空则赋值 '/',创建 search、hash 赋值空字符串
2. 匹配 '#' 所在位置如果 hashIndex 不是 -1 则说明存在 hash 路由，则将 # 之前的字符都赋值给 pathname 将#(包括)之后的赋值给 hash (hash 放在 url 参数之后)
3. 匹配 '?' 所在位置如果 searchIndex 不是 -1 则说明存在 search 参数，则将 ? 之前的字符都赋值给 pathname 将?(包括)之后的赋值给 search
4. 返回 {pathname, search, hash} 如果 search 只有 ? 只返回空字符串 hash 只有 # 同
#### createPath
这个函数就是用来拼接路由、search、hash成为一个url后返回
1. 将传入的对象解构赋值为 pathname/search/hash
2. 如果 Boolean(pathname) 为 false 则给 path 赋值为 '/' 否则为 pathname
3. search 存在且只不是 '?' 判断 search 第一个字符是否是 '?' 是则拼接 path 不是则拼接 '?' 再拼接 search
4. hash 同 search 只不过是 '#'
5. 返回path
### LocationUtils.js
