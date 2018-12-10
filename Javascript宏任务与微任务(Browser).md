## Chrome中js执行机制

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
    resolve();
}).then(() => {
    console.log('10')
})
setTimeout(( ) =>{
    new Promise(function(resolve){
        console.log('5');
        resolve();
    }).then(() => {
        console.log('7')
    })
})

```
#### 宏任务与微任务
* macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
* micro-task(微任务)：Promise

#### 执行过程：
###### 1. script 标签 -> 宏任务
```js
console.log('1')
// 1
```
```js
setTimeout(() => {
    new Promise(function(resolve){
        console.log('2');
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
    resolve();
}).then(() => {
    console.log('10')
})
// 8
// then1 -> 微任务
```

| 宏任务 | 微任务 |
|-------|-------|
|timeout1| then1 |

```js
setTimeout(( ) =>{
    new Promise(function(resolve){
        console.log('5');
        resolve();
    }).then(() => {
        console.log('7')
    })
})
// 宏任务  -> timeout2
```

| 宏任务 | 微任务 |
|-------|-------|
|timeout1| then1 |
| timeout2|  |


###### 2. 宏任务执行完毕 -> 微任务执行

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
    resolve();
}).then(() => {
    console.log('4')
})
// 2
// then2 -> 微任务
```

| 宏任务 | 微任务 |
|-------|-------|
|timeout2| then2 |

###### 4. 宏任务任务完毕 -> 微任务执行

```js
then(() => {
    console.log('4')
})
// 4
```

| 宏任务 | 微任务 |
|-------|-------|
|timeout2|  |

###### 5. 微任务任务完毕 -> 宏任务执行

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
// then3 -> 微任务
```

| 宏任务 | 微任务 |
|-------|-------|
| | then3 |

###### 6. 宏任务任务完毕 -> 微任务执行

```js
then(() => {
    console.log('7')
})
// 7
```

###### 7. 执行完毕
```js
// 1 8 10 2 4 5 7
```

##### 问题
###### Chrome中和Node.js中执行顺序区别
> Node中执行宏任务时，会将队列所有宏任务执行完毕后再去执行微任务
Chrome中一次Loop只执行一个宏任务
