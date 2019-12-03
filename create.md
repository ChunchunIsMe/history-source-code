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

history

1. history.length 返回当前session中的个数，包含当前页面在内
2. history.scrollRestoration 允许 Web 应用程序在历史记录导航上显式设置默认滚动恢复行为可以将其设置为 'auto' 或者 'manual'
3. history.state 返回 history 的状态值 如果不使用 history.pushState/history.repalceState 这个值则为 null
4. history.back() 会话历史记录中向后移动一页。如果没有上一页不做任何操作
5. history.forward() 会话历史记录向前移动一页。如果没有则不做任何操作
5. history.go(delata) 相对当前页面去往历史页面的位置。负值表示向后移动正表示向前移动
6. history.pushState/histroy.replaceState 设置 state 的第一个参数改变url，第二个参数无用，第三个修改state 他们不会进行跳转页面但是会修改url 
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
1. 调用 Object.assign 将参数 nextState 中的属性复制/覆盖到 createBrowserHistory 函数最后定义的 history 对象的属性，这里要注意 const 的暂时性死区是函数调用前定义了即可
2. 但是不能覆盖 history 的 length 属性，history 的 length 属性必须是 window.history.length
3. 调用定义 setState 之前使用 createTransitionManager 创建的 transitionManager.notifyListeners 方法并且传入 history.location/ history.action 这个函数就是调用所有注册的监听函数的。
##### handlePop
因为 setState 后定义的两个函数都用到了 handlePop 所以先来讲这个函数
1. 首先在 handlePop 函数外部定义了一个 forceNextPop 变量初始值为 false
2. 判断 forceNextPop 如果是 true 则将其置为 false 然后调用 setState 这里没有传参，应该只会重置一下 history.length 和调用所有监听函数
3. 如果 forceNextPop 是 false 则调用 transitionManager.confirmTransitionTo 参数为 传入handelPop的location/'POP'/prop.getUserConfirmation||getConfirmation/回调函数
4. 这个回调函数如果被传入 true 则会调用 setState 并且改变 action 和 location 如果传入 false 则会传入 location 调用 revertPop
5. confirmTransitionTo 这个时候就要看有没有 setPrompt 了 如果 prompt 是 null 那这里直接调用 true 的回调函数 如果有 prompt 是函数则会直接调用 并传入 location 和 action 并将结果赋值给 result 如果不是函数则赋值给 result 如果 result 不是字符串也会直接调用回调并传参为 true 然后调用传入的第三个参数 并传入 result 和 callback
##### revertPop
因为 handlePop 的回调中调用了这个，所以我们先来看 revertPop
1. 将导出的 history 下的 location 属性赋值给 toLocation
2. 在外部定义的 allKeys 数组中找到 toLocation.key 并将 indexOf 的结果赋值给 toIndex
3. 找到传入 formLocation.key 在 allKeys 中找到并将 indexOf 的结果赋值给 formIndex
4. 如果 toIndex、formIndex 是 -1 则将其赋值为 0，然后 delta = toIndxe - formIndex
5. 如果 delta 不为 0 就将 forceNextPop 设置为 true 并调用 go(delta)
6. 这个 go 就是 window.histroy.go 
7. 这个函数很明显就是从传入的 Location 跳转到现在 history 下的 location
8. 外部定义了 allKeys 里面存放了所有路由的 key 这里的 key 会在 push 方法中看到 push 新元素，它的初始值是 getDOMLocation(getHistoryState()) 返回的 key getDOMLocation(getHistoryState()) 其实就是生成了一新的 location 这个 location 的各个属性都是当前 window 下的属性

##### handlePopState
1. 如果 event.state === undefined 并且是 CriOS 品台则直接return
2. isExtraneousPopstateEvent 这个函数好像这个老哥忘记返回了
3. 调用 handlePop 并使用传入的 event.state 创建的 location 
##### createHref
这个函数其实就是获取现在的 url + 路由 + 参数等
##### push
1. 如果 path 不是对象 或者 path.state 是 undefined 或者 state 是 undefined 都会抛出警告
2. 创建一个新的 location 
3. 调用 transitionManager.confirmTransitionTo 然后将会调用回调函数
4. 回调函数中将确创建一个 href 并且会从新创建的 location 中获取 key 和 state 属性
5. 检测是否能使用 pushState 如果不能直接抛出错误，如果可以则将调用 pushState 设置 history 中的 state 和 href
6. 如果传入的 forceRefresh 为 true 将会刷新页面
7. 如果传入的 forceRefresh 为 false 则会找到现在 history.location.key 在 allkeys 的位置赋值给 prevIndex，然后从 0-prevIndex 截取一个新key数组，并且 push 新创建的location的key到key数组 ，再将新数组赋值给allKeys实现allKeys的更新，然后调用setState更新导出的history对象

主要的方法都已经讲完了大部分都是基于这些的