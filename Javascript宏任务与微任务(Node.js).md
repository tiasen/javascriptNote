## Node.js 中js执行机制

```js
console.log('1');
setTimeout(() => {
    new Promise(function(resolve){
        console.log('2');
        process.nextTick(function(){
            console.log('3')
        })
        resolve();
    }).then(() => {
        console.log('4')
    })
})
new Promise(function(resolve){
    console.log('8');
    process.nextTick(function(){
        console.log('9')
    })
    resolve();
}).then(() => {
    console.log('10')
})
setTimeout(( ) =>{
    new Promise(function(resolve){
        console.log('5');
        process.nextTick(function(){
            console.log('6')
        })
        resolve();
    }).then(() => {
        console.log('7')
    })
})

```
#### 宏任务与微任务
* macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
* micro-task(微任务)：Promise，process.nextTick

#### 执行过程：
###### 1. 宏任务
```js
console.log('1')
// 1
```
```js
setTimeout(() => {
    new Promise(function(resolve){
        console.log('2');
        process.nextTick(function(){
            console.log('3')
        })
        resolve();
    }).then(() => {
        console.log('4')
    })
})
// 添加到宏任务 -> timeout1
```
| 宏任务 | 微任务 |
|-------|-------|
|timeout1| |


```js
new Promise(function(resolve){
    console.log('8');
    process.nextTick(function(){
        console.log('9')
    })
    resolve();
}).then(() => {
    console.log('10')
})
// 8
// process.nextTick1 -> 微任务
// then1 -> 微任务
```

| 宏任务 | 微任务 |
|-------|-------|
|timeout1| process1 |
| | then1 |

```js
setTimeout(( ) =>{
    new Promise(function(resolve){
        console.log('5');
        process.nextTick(function(){
            console.log('6')
        })
        resolve();
    }).then(() => {
        console.log('7')
    })
})
// 宏任务  -> timeout2
```

| 宏任务 | 微任务 |
|-------|-------|
|timeout1| process1 |
| timeout2| then1 |


###### 2. 宏任务执行完毕 -> 微任务执行
```js
process.nextTick(function(){
    console.log('9')
})
// 9
```

| 宏任务 | 微任务 |
|-------|-------|
|timeout1|  |
| timeout2| then1 |

```js
then(() => {
    console.log('10')
})
// 10
```

| 宏任务 | 微任务 |
|-------|-------|
|timeout1|  |
| timeout2|  |


###### 3. 微任务任务完毕 -> 宏任务执行

```js
new Promise(function(resolve){
    console.log('2');
    process.nextTick(function(){
        console.log('3')
    })
    resolve();
}).then(() => {
    console.log('4')
})
// 2
// process.nextTick2 -> 微任务
// then2 -> 微任务
```

| 宏任务 | 微任务 |
|-------|-------|
|| process2 |
| timeout2| then2 |

```js
new Promise(function(resolve){
    console.log('5');
    process.nextTick(function(){
        console.log('6')
    })
    resolve();
}).then(() => {
    console.log('7')
})
// 5
// process.nextTick3 -> 微任务
// then3 -> 微任务
```

| 宏任务 | 微任务 |
|-------|-------|
|| process2 |
| | then2 |
|| process3 |
| | then3 |

###### 4. 宏任务任务完毕 -> 微任务执行

```js
process.nextTick(function(){
    console.log('3')
})
// 3
```

| 宏任务 | 微任务 |
|-------|-------|
| | then2 |
|| process3 |
| | then3 |

```js
process.nextTick(function(){
    console.log('6')
})
// 6
```


| 宏任务 | 微任务 |
|-------|-------|
| | then2 |
| | then3 |

```js
then(() => {
    console.log('4')
})
// 4
```

| 宏任务 | 微任务 |
|-------|-------|
| | then3 |

```js
then(() => {
    console.log('7')
})
// 7
```

###### 5. 执行完毕
```js
// 1 8 9 10 2 5 3 6 4 7
```

##### 问题
###### process.nextTick也会放入microtask quque，为什么优先级比promise.then高呢？

>“process.nextTick 永远大于 promise.then，原因其实很简单。。。在Node中，_tickCallback在每一次执行完TaskQueue中的一个任务后被调用，而这个_tickCallback中实质上干了两件事：1. nextTickQueue中所有任务执行掉(长度最大1e4，Node版本v6.9.1)2.第一步执行完后执行_runMicrotasks函数，执行microtask中的部分(promise.then注册的回调)所以很明显process.nextTick > promise.then”

