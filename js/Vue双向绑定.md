#### 数据双向绑定是什么？

在MVVM框架shu中有三层，Model、ModelView与View。

数据双向绑定指的是数据（Model）与视图（View）互相绑定，某一方更新会触发另一方的更新，由ModelView负责实现。

#### 数据双向绑定解决了什么?

取代了jq和原生的事件绑定开发方式，开发者只需要关注和维护数据即可。

#### Vue的实现

Vue通过**数据劫持**和**订阅/发布者模式**实现数据的监听和双向绑定。



##### 数据更新

整理下思路：

- 实现发布者、订阅者和数据劫持。
- 维护一个发布者对象，需要监听的属性在此对象中都有对应的属性（发布者）
- 维护一个数据监听对象（Vue 3.0前由Object.defineProperty进行劫持，3.0后由proxy实现），对象的属性值改变时，触发发布者对象里对应的发布者的notify方法，触发已注册的回调

```js
// 发布者
function announcer(){
    this.subscriber=[];
    
    this.addSubscriber = function(cb){
        this.subscriber.push(cb);
    };

    this.notify = function (val){
        this.subscriber.forEach(cb => {
            cb(val);
        });
    };
};

// 订阅者
function Subscriber(queue,key,cb){
    queue[key].addSubscriber(cb);
}

// 数据劫持
function dataHijacking(data,queue){
    return new Proxy(data,{
        set(target,key,value){
            target[key]=value;
            queue[key].notify(value);
        }
    })
}

var myData=dataHijacking({test:""},queue);
var queue={};
for (let key in myData) {
  queue[key] = new announcer();
}
Subscriber(queue,"test",function(val){console.log("test is change:",myData["test"])});
myData.test="hhh";
myData.test="123";
myData.test="...";

```



##### 视图更新

Vue中使用专门的语法属性（如v-model、v-html、v-text等）来实现，其实从视图到数据的绑定只是多了把模板中的语法属性挑出来并注册成为相应发布者的订阅者。

如下面这个模板

```vue
<div id="app">
  <input v-model="value" />
  <p v-text="value"></p>
</div><div id="app">
  <input v-model="value" />
  <p v-text="value"></p>
</div>
```

整理思路：

- 写一个**Compile**方法，将模板中包含的语法属性挑出来，注册成为订阅者