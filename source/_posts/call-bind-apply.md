---
title: call、bind和aplly
date: 2022-07-26 13:00
---
## call
每个`javascript`函数都是`Function`对象，`Function`对象是一个构造函数，构造函数是有原型对象的，也就是`prototype`。`call`就是`Function.prototype`中的其中一个属性。
```js
 Function.prototype.newCall = function(self) {
    const _this = self || window
    _this.callBack = this
    const [ _, ...args ] = Array.from(arguments)
    const result = _this.callBack(...args)
    delete _this.callBack
    return result
 }
```

## apply
`aplly`和`call`的差别是第二个参数是一个数组，并不像`call`有无限多个参数
```js
Function.prototype.newApply = function(self, list) {
    const _this = self || window
    _this.callBack = this
    const result = _this.callBack(...list)
    delete _this.callBack
    return result
}
```

## bind
`bind`函数与`call`、`apply`不同的是并不是立即执行，而是返回一个新的函数，可以基于返回的新函数使用`new`关键字创建对象，也可以执行新函数并传入对应的参数
```js
Function.prototype.newBind = function(self) {
    if(typeof this !== 'function') {
        throw new TypeError('error')
    }
    const [ _, ...args] = Array.from(arguments)
    const _this = this
    const fullFunc = function() {}
    const newFunc = function () {
        const sum = args.concat(Array.from(arguments))
        if (_this instanceof fullFunc) {
           return _this.apply(self, sum)
        } else {
            return _this.apply(self, sum)
        }
    }
    fullFunc.prototype = _this.prototype
    newFunc.prototype = new fullFunc
    return newFunc
}
```