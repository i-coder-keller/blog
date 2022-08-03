---
title: 手写深拷贝
date: 2022-08-03 23:44
---
在javascript中变量存储的的引用数据类型并不是源数据，而是数据指针地址，当进行变量接收数据时接收的实际是指针地址，所以当我们修改新变量时，旧变量的属性也会随之改变

![](https://image.liuyongzhi.cn/images/deepClone-init.png)

当然理想状态是两个变量是相对独立的，这就需求我们进行手动的深拷贝。所谓深拷贝就是找到数据每个节点，如果为基础数据类型就直接赋值，反之为引用数据类型就是继续寻找基础数据类型节点，直到没有更多节点。

### Array 和 Object

![](https://image.liuyongzhi.cn/images/deepClone-Array%3AObject.png)

进入函数先进行判断入参是否是一个Object或Array，如果不是目标值则直接返回，反之则根据目标值类型初始化返回值。使用`for...in`来循环目标函数拿到枚举的Key值，因为不需要去关注和拷贝源数据原型的属性，所以使用 [hasOwnProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty) 进行过滤。再进行递归循环进行赋值

### 循环引用

上面版本的函数基本可以处理大部分的场景，但是如果源数据中存在循环引用的情况，那么就会导致函数调用栈爆栈的异常。例如：

![](https://image.liuyongzhi.cn/images/deepClone-Maximum.png)

对于循环引用可以使用[`WeakMap`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)或[`Map`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)，由于`WeakMap`是弱引用更利于垃圾回收，性能更好，所以此处使用`WeakMap`进行处理。远离就是当函数开始时如果是引用数据类型，那么就需要先去`WeakMap`中查询，如果已经存在那么就直接返回当前结果，反之没有的话，就在`WeakMap`中以当前克隆对象作为`Key`，当前值作为值保存

![](https://image.liuyongzhi.cn/images/deepCLone-latest.png)

![](https://image.liuyongzhi.cn/images/deepClone-result.png)

经过`WeakMap`改良过后的函数就可以解决循环引用的问题，主要是利用递归循环增加判断使在赋值的时候，复制的一定是值拷贝，而不是指针拷贝

### 源码

```js
const deepClone = (origin, map = new WeakMap()) => {
            if (typeof origin !== 'object'||origin === null) {
                return origin
            }
            if (map.get(origin)) return map.get(origin)
            const result = Array.isArray(origin) ? [] : {}
            map.set(origin, result)
            for(key in origin) {
                if (origin.hasOwnProperty(key)) {
                    result[key] = deepClone(origin[key], map)
                }
            }
            return result
        }
```

