---
title: Javascript的6种常见设计模式
date: 2019-04-22 14:16:44
tags: javascript
---

#### 构造函数模式
```js
/**
 * 构造一个动物的函数 
 */
 function Animal(name, color) {
    this.nme = name;
    this.color = color;
    this.getName = function() {
        return this.name;
    }
 }
var cat = new Animal('猫'， '白色');
```

#### 工厂模式
```js
/**
 * 工厂模式
 */
 function Animal(opts) {
    var obj = new Object();
    obj.name = opts.name;
    obj.color = opts.color;
    obj.getInfo = function() {
        return '名称：' + obj.name + ';颜色：' + obj.color;
    }
    return obj;
 }
 var cat = Animal({name: '波斯猫', color: '白色'});
 cat.getInfo();
```

#### 模块模式
```js
/**
 * 模块模式 = 封装大部分代码，只暴露必需接口
 */
var Car = (function() {
    var name = '法拉利';
    function sayName() {
        console.log(name);
    }
    function getColor(name) {
        console.log(name);    
    }
    return {
        name: sayName,
        color: geyColor
    }
})();
Car.name();
Car.color('红色');
```

#### 混合模式
```js
/**
 * 混合模式 = 原型模式 + 构造函数模式
 */
function Animal(name, color) {
    this.name = name;
    this.color = color;
}
Animal.prototype.getInfo = function() {
    console.log('名称：' + this.name);
}

function LargetCar(name, color) {
    Animal.cal(null, name. color);
    
    this.color = color;
}
LargetCar.prototype = create(Animal.prototype);
function create(parentObj) {
    function F() {};
    F.prototype = parentObj;
    return new F();
}

largetCat.prototye.getColor = function() {
    return this.color;
}

var cat = new largetCat("persian", "白色");
```

#### 单例模式
```js
/**
 * 在执行当前 Single 只获得唯一一个对象
 */
var Single = (function () {
    var instance;
    function init() {
        //define private methods and properties
        //do something
        return {
            //define public methods and properties
        }
    }
    
    return {
        // 获取实例
        getInstance: function () {
            if(!instance){
                instance = init();
            }
            return instance;
        }
    }
})();

var obj1 = Single.getInstance();
var obj2 = Single.getInstance();

console.log(obj1 === obj2);
```

#### 发布订阅模式
```js
/**
 * 发布订阅模式
 */
var EventCenter = (function () {
    var events = {};
    /*
    {
      my_event: [{handler: function(data){xxx}}, {handler: function(data){yyy}}]
    }
    */
    // 绑定事件 添加回调
    function on(evt, handler) {
        events[evt] = events[evt] || [];
        events[evt].push({
            handler: handler
        });
    }
    function fire(evt, arg){
        if (!events[evt]) {
            return
        }
        for(var i=0; i < events[evt].length; i++){
            events[evt][i].handler(arg);
        }
    }
    function off(evt){
        delete events[evt];
    }
    return {
        on:on,
        fire:fire,
        off:off
    }
})();

var number = 1;
EventCenter.on('click', function(data){
    console.log('click 事件' + data + number++ +'次');
});
EventCenter.off('click');   //  只绑定一次
EventCenter.on('click', function(data){
    console.log('click 事件' + data + number++ +'次');
});

EventCenter.fire('click', '绑定');
```