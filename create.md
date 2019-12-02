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