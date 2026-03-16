---
title: compose的执行和实现方式
date: 2018-12-13
---

## compose：执行一系列任务的函数 tasks = [ step1, step2, step3… ]

  * Bulleted
  * List


  1. 执行顺序从右到左边
  2. 第一个函数的参数可以是多个，后面的函数参数只能是一个
  3. 函数的执行是同步的



let init = (…args) => args.reduce((ele1, ele2) => ele1 + ele2, 0)  
let step2 = (val) => val + 2  
let step3 = (val) => val + 3  
let step4 = (val) => val + 4  
输出： 15

## compose的实现

（1）lodash中的实现
    
    
    ```markdown
    var flow = function（funcs）{
        Var length = funcs.length
        Var index = length
    
        while (index--) {
            if (typeof funcs[index] !== 'function') {
                throw new TypeError('Expected a function');
            }
        }
            Return function(…args) {
            var index = 0
            var result = length ? funcs[index].apply(this, args) : args[0]
            while (++index < length) {
                result = funcs[index].call(this, result)
            }
            return result
            }
    }
    ```

（2）promise实现compose
    
    
    ```markdown
    const compose = function(…funcs) {
      let init = funcs.pop()
      return function(...arg) {
        return funcs.reverse().reduce(function(sequence, func) {
          return sequence.then(function(result) {
            return func.call(null, result)
          })
        }, Promise.resolve(init.apply(null, arg)))
      }
    }
    ```
