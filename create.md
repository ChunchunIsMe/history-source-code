## createXXX
createXXX 这些就是 history 的核心代码了
### createTransitionManager.js
这个文件先导入了 tiny-warning 这个导出了一个方法，如果是 development 环境并且第一个参数为 false 则会打印出错误并且抛出错误这个错误使用try/catch包裹了所以不会直接暂停代码执行。
#### createTransitionManager
这个文件导出了一个函数，这个函数的返回值是一个对象中有四个函数
##### setPrompt
1. 判断函数外部变量prompt是否是null 如果不是则抛出错误
2. 将参数赋值给外部变量 prompt
3. 返回一个函数，如果外部变量 prompt 和 参数相等调用则会重置外部变量 prompt 为 null
##### confirmTransitionTo
1. 接收四个参数 location、action、getUserConfirmation、callback
2. 如果外部变量 prompt 是 null 则调用 callback 并传入 true
3. 如果外部变量 prompt 不是 null 则创建一个变量 result 
4. 如果 prompt 此时是 函数则调用 prompt 并传入 location 和 action 将返回值作为 result 的值
5. 如果 prompt 不是函数则直接将 prompt 赋值给result
6. 如果 result 不是 String 类型 则直接调用 callback 并传入 result !== false
7. 如果是 String 类型并且传入的 getUserConfirmation 是 function 类型则调用 getUserConfirmation 并传入 result 和 callback
8. 如果不是函数则直接调用 warning 抛出错误，然后调用 callback 并传入 true
##### appendListener
1. 定义 isActive 为 true
2. 定义 listener 函数该函数中如果 isActive 为 true 则调用 fn 并且参数与 listener 一致
3. 外部 listeners 数组 push 进刚刚定义的 listener 函数
4. 返回一个函数，如果调用这个函数则会将 isActive 置为 false 并将刚刚这次调用 appendListener 推入 listeners 的 listener 去除
##### notifyListeners
这个函数就是循环调用 listeners 中的函数并且参数都是传入 notifyListeners 函数的参数。
### createBrowserHistory.js
我们先来看他导入的东西和全局定义的东西
1. warning 第一个参数是false则抛出错误但并不会阻断代码运行
2. invariant 第一个参数是false抛出错误，如果不是线上环境抛出错误的信息更全，会阻断代码运行
3. createLocation 创建location的方法，参数是路由
4. addLeadingSlash 判断传入的 path 开头是否有 '/' 没有则加上返回
5. stripTrailingSlash 判断一下传入的path最后一个字符是否是 '/' 如果是则切割字符串返回去除最后一个字符的字符串，如果不是则直接返回path
6. createPath 拼接路由、search、hash成为一个url后返回
7. createTransitionManager 上个解读的文件
8. canUseDOM 判断是否能用window.document.createElement
9. getConfirmation 弹个 confirm 框然后将弹框的message返回给你给的回调函数
10. supportsHistory 判断 window.history.pushState 是否能用
11. supportsPopStateOnHashChange 检测是否是 Trident 客户端
12. isExtraneousPopstateEvent 检测是否是 CriOS 客户端
13. 定义 PopStateEvent 和 HashChangeEvent 分别赋值 'popstate'/'hashchange'

这个文件只导出一个函数那就是 createBrowserHistory 并且这个函数返回了一大堆方法和属性，我们来一个个看
```
{
  length: globalHistory.length,
  action: 'POP',
  location: initialLocation,
  createHref,
  push,
  replace,
  go,
  goBack,
  goForward,
  block,
  listen
}
```
#### getHistoryState
返回 window.history.state 如果不存在或者报错则返回 {}
#### createBrowserHistory
1. 首先如果不支持 createElement 直接抛出错误并且暂停代码执行
2. 将 window.history 赋值给 globalHistory 判断是否能用 pushState 并赋值给 canUseHistory 检测是否是 Trident 客户端取反赋值给 needsHashChangeListener 
3. 从参数中取得 forceRefresh/getUserConfirmation/keyLength 三个属性，默认值为 false/getConfirmation(导入的弹框函数)/6
4. 定义 basename 如果传入的参数有 basename 属性则str开头加'/'(如果没有)最后去掉'/'(如果有) 没有这个属性则 设为 ''
##### getDOMLocation
createBrowserHistory 中定义了 getDOMLocation 函数，这个函数就是来设置 state/key

1. 定义 key、state 从传入参数的属性中获取
2. 定义 pathname、search、hash 从 window.location 的属性中获取
3. 定义 path = pathname + search + hash
4. 如果 basename 存在或者 path 不是正确的路由则抛出错误
5. 如果 basename 存在则将 path 赋值为 stripBasename 函数传入 path/basename 的返回值(?后的值)
6. 返回 createLocation 调用后的返回值参数为 path/state/key 这个函数会设置 state 和 key 然后返回一个 location 对象其中有 pathname/state/search/hash 属性
##### createKey
就是生成一个a-z或0-9的随机字符串
##### setState
1. 调用 Object.assign 将参数 nextState 中的属性复制/覆盖到 window.history 对象的属性
2. 但是不能覆盖 window.history 的 length 属性因为之前将 window.history 赋值给了