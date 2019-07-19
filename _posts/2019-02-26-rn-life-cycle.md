---
layout: post
title: react-native开发笔记 - 组件的生命周期
date: 2019-02-26 18:32:24.000000000 +09:00
tag: React-Native
---


一个 `React Native` 组件从它被加载，到最终被卸载，会经历一个完整的生命周期。

在这个生命周期中，框架已经为我们定义了一些生命周期函数，用来处理在特定条件下 `React Native` 组件将要执行的操作，比如在某个时间进行数据初始化或进行网络请求等。


#### 一、组件的生命周期

组件的生命周期一般分为4个阶段：`创建阶段`、`实例化阶段`、`运行(更新)阶段`和`销毁阶段`。下面对各个阶段分别进行介绍。

##### 1. 创建阶段
 - 该阶段主要发生在创建组件类的时候，在这个阶段会初始化组件的属性类型`propTypes`和默认值`defaultProps`。当调用该组件时，只要看创建阶段的属性声明，即可知道如何调用。
 - 在 ES6 中统一使用 `static` 成员来实现。

```
    // 声明属性类型
    static propTypes = {
        name: PropTypes.string,
    };

    // 声明属性默认值
    static defaultProps = {
        name: 'xiaowang'
    };
```
> 备注1：出于性能原因，`propTypes`类型仅在`develop`模式下进行检查，`production`环境下不检查。
备注2：**RN官方推荐以组件的方式来重用代码，而不是继承**。所以，在实际开发中，应多封装组件，并写清楚组件需要传入的参数类型声明，方便调用。


##### 2. 实例化阶段

该阶段主要发生在实例化组件的时候，创建组件，准备渲染，渲染，渲染完成。
调用的函数顺序如下：
 - `constructor`：构造函数，对组件的状态进行初始化。
 - `componentWillMount`：准备加载组件，可以做一些业务初始化工作或设置组件状态。
 - `render`：生成页面需要的DOM结构，并返回。
 - `componentDidMount`： 组件加载成功并被成功渲染后执行。一般讲网络请求等获取数据比较耗时的操作放在这里进行，保证UI渲染不出现错误。

##### 3. 运行(更新)阶段

该阶段主要发生在用户操作之后或者父组件有更新的时候，此时会根据用户的操作行为进行相应的页面结构的调整。这个阶段也会触发一系列的流程，按执行顺序如下：
 - `componentWillReceiveProps`：当组件接收到新的 props 时，会触发该函数。在该函数中，通常可以调用 this.setState 方法来完成对 state 的修改。
 - `shouldComponentUpdate`：该方法用来拦截新的 props 或 state，然后根据事先设定好的判断逻辑，做出最后要不要更新组件的决定。
 - `componentWillUpdate`：当上面的方法拦截返回 true 的时候，就可以在该方法中做一些更新之前的操作。
 - `render`：根据一系列的 diff 算法，生成需要更新的虚拟 DOM 数据。（注意：在 render 中最好只做数据和模板的组合，不应进行 state 等逻辑的修改，这样组件结构会更加清晰）
 - `componentDidUpdate`：该方法在组件的更新已经同步到 DOM 中去后触发，我们常在该方法中做 DOM 操作。

##### 4. 销毁阶段
该阶段主要在组件销毁的时候触发。
 - `componentWillUnmount`：当组件要被从界面上移除时就会调用。可以在这个函数中做一些相关的清理工作，例如取消计时器、网络请求以及移除通知监听等。

流程图；
![生命周期流程图.png](/images/posts/rn-lifecycle/rn-life.png)

#### 二、生命周期函数详细介绍

##### 1. constructor
（1）函数原型
```
    constructor(props) {
        super(props);
        this.state = {}
    }
```

（2）基本介绍
 - 它是组件的构造函数。它的第一个语句必须是 `super(props)`，参数`props`值为外界传入的属性，所以这里可以直接使用`props`值来初始化`state`。
 - 构造函数将在组件被加载前最先调用，并且仅调用一次。

（3）常见用途
构造函数最大的作用，就是在这里定义状态机变量。


##### 2. componentWillMount
（1）函数原型
```
   componentWillMount() {
        console.log('componentWillMount');
    }
```

（2）基本介绍
 - 在组件的生命周期中，这个函数只会被执行一次，从下个页面返回到当前页面也不会再调用。
 - 它在初始渲染（`render` 函数被 `React Native` 框架调用执行）前被执行，当它执行完后，`render` 函数会马上被 `React Native` 框架调用执行。
 - 如果子组件也有 `componentWillMount` 函数，它会在父组件的 `componentWillMount` 函数之后被调用。

（3）常见用途
如果我们需要从本地存储中读取数据用于显示，那么在这个函数里进行读取是一个很好的时机。

> 注意：如果在这个函数里调用 `setstate` 函数改变了某些状态机变量的值， `React Native` 框架不会执行渲染操作，而是等待这个函数执行完成后再执行初始渲染。

##### 3. render
（1）函数原型
```
render() {
        return (
            <View style={styles.container}>
                <Text>{this.state.name}</Text>
            </View>
        )
    }
```
（2）基本介绍
 - `render` 是一个组件必须有的方法，用于界面渲染。
 - 这个函数无参数，返回 `JSX` 或者其他组件来构成 `DOM`。注意：只能返回一个顶级元素。

##### 4. componentDidMount

（1）函数原型
```
componentDidMount() {
        console.log('componentDidMount');
    }
```

（2）基本介绍
 - 在组件的生命周期中，这个函数只会被执行一次。
 - 它在初始渲染执行完成后会马上被调用。在组件生命周期的这个时间点之后，开发者可以通过子组件的引用来访问、操作任何子组件。
 - 如果子组件也有 `componentDidMount` 函数，它会在父组件的 `componentDidMount` 函数之前被调用。

（3）常见用途
如果 React Native 应用需要在程序启动并显示初始界面后从网络侧获取数据，那么把从网络侧获取数据的代码放在这个函数里是一个不错的选择。

##### 5. componentWillReceiveProps
（1）函数原型
```
componentWillReceiveProps(nextProps) {
        if (nextProps.name != this.state.name) {
            this.setState({
                name: nextProps.name
            })
        }
    }
```

（2）基本介绍
 - 组件的初始渲染执行完成后，当组件接收到新的 `props` 时，这个函数将被调用。
 - 这个函数不需要返回值。接收一个 `object` 参数， `object` 里是新的 `props`。
 - 如果新的 `props` 会导致界面重新渲染，这个函数将在渲染前被执行。在这个函数中，老的 `props` 可以通过 `this.props` 访问，新的 `props` 在传入的 `object` 中。
 - 如果在这个函数中通过调用 `this.setState` 函数改变某些状态机变量的值， `React Native` 框架不会执行对这些状态机变量改变的渲染，而是等 `componentWillReceiveProps` 函数执行完成后一起渲染。

>注意：当 `React Native` 初次被渲染时，`componentWillReceiveProps` 函数并不会被触发，这种机制是故意设计的。

##### 6. shouldComponentUpdate

（1）函数原型
```
    shouldComponentUpdate(nextProps, nextState) {
        // nextProps 最新的属性
        // nextState 将要渲染的状态
        return true;
    }
```

（2）基本介绍
 - 组件的初始渲染执行完成后，当组件接收到新的 `state` 或者 `props` 时这个函数将被调用。
 - 该函数接收两个 `object`参数，其中第一个是新的 `props`，第二个是新的 `state`。
 - 该函数需要返回一个布尔值，告诉 `React Native` 框架针对这次改变，是否需要重新渲染本组件。默认返回`true`。如果此函数返回 `false`，`React Native` 将不会重新渲染本组件，相应的，该组件的 `componentWillUpdate` 和 `componentDidUpdate` 函数也不会被调用。

（3）常见用途
这个函数常常用来阻止不必要的重新渲染，提高 `React Native` 应用程序性能。
比如我们可以在该函数中比较新老版本的 `state` 和 `props`，判断是否需要进行重新渲染。下面是一个简单的使用样例：
```
shouldComponentUpdate(nextProps, nextState) {
  if(this.state.inputedNum.length < 3) return false;
  return true;
}
```
##### 7. componentWillUpdate
（1）函数原型
```
    componentWillUpdate(nextProps, nextState) {
        console.log('componentWillUpdate');
    }
```

（2）基本介绍
 - 组件的初始渲染执行完成后， `React Native` 框架在重新渲染该组件前会调用这个函数。
 - 该函数不需要返回值，接收两个 `object` 参数，其中第一个是新的 `props`，第二个是新的 `state`。
 - 我们可以在这个函数中为即将发生的重新渲染做一些准备工作，但不能在这个函数中通过 `this.setState` 再次改变状态机变量的值。如果需要改变，则在 `componentWillReceiveProps` 函数中进行改变。

##### 8. componentDidUpdate

（1）函数原型
```
componentDidUpdate(prevProps, prevState) {
        console.log('componentDidUpdate');
    }
```

（2）基本介绍
 - `React Native` 框架在重新渲染该组件完成后会调用这个函数（组件第一次渲染执行完成后不调用该函数）。
 - 该函数不需要返回值，接收两个 `object` 参数，其中第一个是渲染前的 `props`，第二个是渲染前的 `state`。

##### 9. componentWillUnmount
（1）函数原型
```
    componentWillUnmount() {
        console.log('componentWillUnmount');
    }
```

（2）基本介绍
 - 在组件被卸载前，这个函数将被执行。
 - 这个函数没有参数，也没不需要返回值。

（3）常见用途
如果组件申请了某些资源或者订阅了某些消息，那么需要在这个函数中释放资源，取消订阅。

#### 三、举例
1.示例代码：
```
'use strict';

import React, { Component, PropTypes } from 'react';
import {
    View,
    Text,
    Button,
    StyleSheet,
    AppRegistry
} from 'react-native';

class LifecycleClass extends Component {

    // 声明属性类型
    static propTypes = {
        name: PropTypes.string,
    };

    // 声明属性默认值
    static defaultProps = {
        name: 'xiaowang'
    };

    constructor(props) {
        // 初始化（只调用一次）
        super(props);

        this.state = {
            name: props.name
        }
    }
    
    componentWillMount() {
        // 组件将要加载（只调用一次）
    }

    componentDidMount() {
        // 组件已经加载（只调用一次）
    }

    componentWillReceiveProps(nextProps) {
        // 组件接收到外部属性改变（属性改变时调用，可调用多次）
        if (nextProps.name != this.state.name) {
            this.setState({
                name: nextProps.name
            })
        }
    }

    shouldComponentUpdate(nextProps, nextState) {
        // 组件是否要刷新  nextProps: 最新的属性    nextState: 将要渲染的状态
        return true;
    }

    componentWillUpdate(nextProps, nextState) {
        // 组件将要刷新
    }

    componentDidUpdate(prevProps, prevState) {
        // 组件已经刷新
    }

    componentWillUnmount() {
        // 组件将要卸载
    }

    render() {
        // 渲染组件
        return (
            <View style={styles.container}>
                <Text>{this.state.name}</Text>
            </View>
        )
    }
}
```

2.执行结果：
```
// --------------初次渲染--------------
// 打印结果：
constructor
componentWillMount
render
componentDidMount


// --------------外部修改name属性--------------
// 打印结果：
componentWillReceiveProps
shouldComponentUpdate
componentWillUpdate
render
componentDidUpdate


// --------------关闭该组件--------------
componentWillUnmount
```

demo地址：[RN组件生命周期demo](https://github.com/mengai123/jscodesotre/blob/master/Lifecycle.js)