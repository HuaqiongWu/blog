---
title: ES6中的proxy和应用
date: 2018-11-14
---

1、ES6 中的Proxy的使用
    
    
    ```bash
    var p = new Proxy(target, handler) // target为被包裹的对象，handler为处理方式
    handler能代理的方法有： get、set、has、construct
    var target = {
       a : ‘111’, ‘b’: ‘fasdf'
    }
    Var newObj = Proxy(target, {
        Set: function(target, key, value) {
            console.log(key, ‘被设置。。。’)
            target[key] = value
        }，
        get: function(target, key) {
            console.log(key, ‘被读取。。。’)
            return target[key]
        }
    })
    newObj.name = ‘fasdfdf’
    console.log(nameObj.name)
    ```

2、Proxy的应用  
（1）添加虚拟属性
    
    
    ```bash
    var person = {
      fisrsName: '张',
      lastName: '小白'
    };
    var proxyedPerson = new Proxy(person, {
      get: function (target, key) {
        if(key === 'fullName'){
          return [target.fisrsName, target.lastName].join(' ');
        }
        return target[key];
      },
      set: function (target, key, value) {
        if(key === 'fullName'){
          var fullNameInfo = value.split(' ');
          target.fisrsName = fullNameInfo[0];
          target.lastName = fullNameInfo[1];
        } else {
          target[key] = value;
        }
      }
    });
    console.log('姓:%s, 名:%s, 全名: %s', proxyedPerson.fisrsName, proxyedPerson.lastName, proxyedPerson.fullName);
    // 姓:张, 名:小白, 全名: 张 小白
    proxyedPerson.fullName = '李 小露';
    console.log('姓:%s, 名:%s, 全名: %s', proxyedPerson.fisrsName, proxyedPerson.lastName, proxyedPerson.fullName);
    // 姓:李, 名:小露, 全名: 李 小露
    ```

（2）用来做私有变量隐藏
    
    
    ```bash
    var api = {
      _secret: 'xxxx',
      _otherSec: 'bbb',
      ver: 'v0.0.1'
    };
    
    api = new Proxy(api, {
      get: function(target, key) {
        // 以 _ 下划线开头的都认为是 私有的
        if (key.startsWith('_')) {
          console.log('私有变量不能被访问');
          return false;
        }
        return target[key];
      },
      set: function(target, key, value) {
        if (key.startsWith('_')) {
          console.log('私有变量不能被修改');
          return false;
        }
        target[key] = value;
      },
      has: function(target, key) {
        return key.startsWith('_') ? false : (key in target);
      }
    });
    api._secret; // 私有变量不能被访问
    console.log(api.ver); // v0.0.1
    api._otherSec = 3; // 私有变量不能被修改
    console.log('_secret' in api); // true
    console.log('ver' in api); // false
    ```

（3）在代理中实现属性赋值的校验
    
    
    ```bash
    function Animal() {
      return createValidator(this, animalValidator);
    }
    var animalValidator = {
      name: function(name) {
        // 动物的名字必须是字符串类型的
        return typeof name === 'string';
      }
    };
    function createValidator(target, validator) {
      return new Proxy(target, {
        set: function(target, key, value) {
          if (validator[key]) {
            // 符合验证条件
            if (validator[key](value)) {
              target[key] = value;
            } else {
              throw Error(`Cannot set ${key} to ${value}. Invalid.`);
            }
          } else {
            target[key] = value
          }
        }
      });
    }
    var dog = new Animal();
    dog.name = 'dog';
    console.log(dog.name);
    dog.name = 123; // Uncaught Error: Cannot set name to 123. Invalid.
    ```

3、用ES5实现Proxy
    
    
    ```bash
    function clone(myObj){
        if(typeof(myObj) != 'object' || myObj == null) return myObj;
        var newObj = new Object();
        for(var i in myObj){
          newObj[i] = clone(myObj[i]);
        }
        return newObj;
    }
    /*代理实现类*/
    function ProxyCopy(target,handle){
      var targetCopy = clone(target);
      Object.keys(targetCopy).forEach(function(key){
        Object.defineProperty(targetCopy, key, {
          get: function() {
            return handle.get && handle.get(target,key);
          },
          set: function(newVal) {
            handle.set && handle.set();
            target[key] = newVal;
          }
        });
      })
      return targetCopy;
    }
    
    var person = {name:''};
    var personCopy = new ProxyCopy(person,{
      get(target,key){
        console.log('get方法被拦截。。。');
        return target[key];
      },
      set(target,key,value){
        console.log('set方法被拦截。。。')
        // return true;
      }
    })
    person.name = 'arvin';  // 未有拦截日志打出
    personCopy.name = 'arvin';  // set方法被拦截。。。
    console.log(person.name);   // 未有拦截日志打出
    console.log(personCopy.name);   // get方法被拦截。。。
    // 这个的缺点是不能检测新添加的属性
    ```

4、以上es5中的实现利用了Object.defineProperty作用
    
    
    ```bash
    Object.defineProperty(obj, prop, descriptor) // 返回 obj
    Descriptor是一个obj属性有：
    value： 属性值
    writable：是否可以重写
    enumerable: 是否可以被枚举
    configurable: 目标属性是否可以被删除或者再次修改
    除此之外，还有set和get属性，当给一个属性定义getter和setter时，这个属性称之为访问描述符， js会忽略他本身的value以及writable属性，取而代之的访问set和get函数。
    Object.defineProperty(obj, key, {
      set: function(){
        console.log(’set….')
      },
      get: function(){
        console.log(‘get...')
      }
    })
    ```

5、proxy在vue中的作用（实现双向绑定，即监听值的变化）
