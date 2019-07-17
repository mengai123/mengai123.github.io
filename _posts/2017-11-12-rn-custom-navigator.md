---
layout: post
title: ReactNative-Navigator组件使用总结
date: 2017-11-12 18:32:24.000000000 +09:00
tag: React-Native
---

> 文章描述本人在开发RN跨平台应用时，使用Navigator导航器的一些实践经验，以防忘记，也供他人参考。



#### 一、前言
如果你刚接触reactNative，并且想跨平台开发，可以直接选择使用[React Navigation](https://reactnavigation.org/ "React Navigation")。如果你只针对iOS平台开发，并且想和iOS原生外观一致，可以使用`NavigatorIOS`组件。

`Navigator`是官方推出的导航组件，兼容iOS与Android两端。从0.44版本开始，`Navigator`被从react native的核心组件库中剥离到了一个名为`react-native-deprecated-custom-components`的单独模块中。也就是说在0.44版本后，如果要使用`Navigator`，需要先将`react-native-deprecated-custom-components`安装到工程中，在需要使用的地方import。

在实际开发过程中，`Navigator`性能表现还是比较不错的，也很稳定，毕竟经历了那么多版本的检验。UI表现上也不错，几乎与原生相当。虽然被移除了RN核心库，但不影响使用。

> 写此文章时，我们团队使用的RN是0.44.3版本，官方已经更新到0.51版本。

#### 二、为什么我不使用React Navigation

本来也是考虑直接使用`React Navigation`，但我们是在现有Native工程基础上增加的RN功能，根据业务功能不同，分为不同的module。需要在同一个module中，根据Native不同的传值，跳转到不同的RN页面，也就是说导航器的initialRoute(根视图)是变化的，不是固定不变的。而经过研究，`React Navigation`比较适合根视图是固定的情况，所以只好放弃之。

#### 三、安装&使用
**1.安装：**

在项目根目录执行命令(与node_modules同级)：

```shell
npm install react-native-deprecated-custom-components
```
> 注意：经过本人多次踩坑得到的经验，安装前，最好先执行命令`npm install`，再执行上面的安装命令。直接安装的话，会出现很多奇怪的问题 :broken_heart:。

安装位置：
在~/node_modules/react-native-deprecated-custom-components目录下

目录截图：
![](/images/posts/rn-navigator/rn-navigator-code-screenshots.png)


**2.使用：**
`Navigator`的使用与iOS中的UINavigationController类似，一般作为根视图。将所有的子视图组件都放到`Navigator`中，再在对应render函数中返回`Navigator`。

代码如下：
```javascript
render() {
        return (
                <Navigator
                    initialRoute={{title: this._getRootConmponet().title, id: this.state.rootPageKey, component: this._getRootConmponet().component}}
                    configureScene={(route) => {
                        return Navigator.SceneConfigs.PushFromRight;
                    }}
                    renderScene={(route, navigator) => {
                        let Component = route.component;
                        return <Component {...route.params} route={route} navigator={navigator} />
                    }}
                    sceneStyle={{paddingTop: paddingTopOffset, paddingBottom:paddingBottomOffset}}
                    navigationBar={
                        <Navigator.NavigationBar
                            style={{
                                alignItems: 'center',
                                backgroundColor: '#f8f8f8',
                                borderBottomWidth:1/PixelRatio.get(),
                                borderBottomColor:'#cccccc',
                            }}
                            routeMapper={RouteMapper}
                            navigationStyles={Navigator.NavigationBar.Styles}
                        />
                    }
                />
        )
    }
```

代码解释：
`initialRoute`：为代码的根组件，也就是启动app之后会看到界面的第一屏，其中，其有三个参数，title:组件的名字，id：是组件的唯一标识(字符串类型，是为了区分组件的唯一性而自定义的)，component：根组件。
```javascript
initialRoute={{title: this._getRootConmponet().title, id: this.state.rootPageKey, component: this._getRootConmponet().component}}
```
> 注：参数个数和参数名字不是固定的，你这里怎么定义，决定后面怎么使用。

`configureScene`：这个是场景配置，决定页面之间跳转时候的动画方式，跳转方式比较多，具体可以到NavigatorSceneConfigs.js文件中查看。
`renderScene`：场景渲染，返回一个组件元素。`let Component = route.component`就是取每个route里的组件，例如initialRoute里的component，在配置完后，return该组件。
`sceneStyle`：场景样式，统一设置页面偏移量等，可以用来适配安卓和iOS导航栏高度不一致问题，也可以用来适配iPhoneX。
`navigationBar`：导航栏属性，返回一个Navigator.NavigationBar类型的组件，使用navigationBar属性优点是方便，具有类似原生的过渡动画；缺点是，需要在其属性routeMapper中统一定制页面导航栏样式，而不能在各个页面中定制导航栏样式，如果有页面的导航栏样式比较特别，这就需要使用上文提到的组件`id`进行判断，耦合性比较高。当然也可以不设置navigationBar属性，自己定义每个页面的导航栏(比较烦，下面说)。

#### 四、使用系统自带navigationBar
**1.navigationBar：**
示例代码：
```javascript
navigationBar={
    <Navigator.NavigationBar
                            style={{
                                alignItems: 'center',
                                backgroundColor: '#f8f8f8',
                                borderBottomWidth:1/PixelRatio.get(),
                                borderBottomColor:'#cccccc',
                            }}
                            routeMapper={RouteMapper}
                            navigationStyles={Navigator.NavigationBar.Styles}
                        />
}
```
这里`navigationBar`只使用了三个属性：
`style`：统一定义navigationBar的样式，背景色，底部线条，子视图位置等。
`routeMapper`：这个是navigationBar的灵魂，它决定navigationBar显示什么，如何操作等。
`navigationStyles`：安卓和iOS的导航栏样式不一样，`Navigator.NavigationBar.Styles`中会判断当前是什么系统，安卓就返回一个`NavigatorNavigationBarStylesAndroid`，iOS就返回`NavigatorNavigationBarStylesIOS`，后面提到的适配iPhoneX，就需要改动`NavigatorNavigationBarStylesIOS`文件。

**2.routeMapper：**
由于`routeMapper`内容比较多，可以单独抽出到一个js文件中管理。
示例代码：
```javascript
module.exports = {

    //左边按钮
    LeftButton(route, navigator, index, navState) {
        if(index > 0) {
            return (
                <TouchableOpacity
                    onPress={() => {
                        if (route.backClick) {
                            route.backClick(); //如果动作被拦截，那就直接新动作
                        } else {
                            navigator.pop() //否则，pop
                        }
                    }}
                    style={styles.leftButtonStyle}>
                    <Image source={require('../images/trc_pay_pop_btn_back.png')} resizeMode='stretch'/>
                </TouchableOpacity>
            );
        } else {
            if (route.id === Config.AccountLoginPage.id) {
                return (
                    <TouchableOpacity
                        onPress={() => {
                            if (route.rootBack) { //如果传入根视图返回，就执行新动作
                                return route.rootBack()
                            }else {
                                TRCNativeBridge.dismiss();
                            }
                        }}
                        style={styles.leftButtonStyle}>
                        <Image style={{marginLeft:10}}
                               source={require('../images/trc_account_login_close.png')}
                               resizeMode='stretch' />
                    </TouchableOpacity>
                )
            } else {
                return (
                    <View/>
                );
            }
        }
    },

    //右边按钮
    RightButton(route, navigator, index, navState) {
        if(index > 0 && route.rightButtonTitle) {
            return (
                <TouchableOpacity
                    onPress={() => {
                        if (route.rightBarButtonOnPress) { //道理同上
                            route.rightBarButtonOnPress()
                        }
                    }}
                    style={styles.rightButtonStyle}>
                    <Text style={styles.rightButtonTextStyle} numberOfLines={1}>{route.rightButtonTitle}</Text>
                </TouchableOpacity>
            );
        } else {
            return <View />
        }
    },

    //标题
    Title(route, navigator, index, navState) {
        let title = route.title ? route.title : '';
        return (
            <View style={styles.titleBgStyle}>
                <Text style={styles.middleButtonTextStyle}>{title}</Text>
            </View>
        );
    }
};
```

代码解释：
`routeMapper`对象中有三个函数，`LeftButton`、`RightButton`和`Title`，分别代表左边按钮，右边按钮和中间标题，它们的参数都是(route, navigator, index, navState)，它们都需要返回一个组件元素。

其中函数每个参数含义是：
`route`:表示当前的路由。
`navigator`:表示当前的导航器。
`index`:表示当前的页面的在导航栈中的位置索引。
`navState`:表示当前的导航状态。

**3.LeftButton：**
解释一下`LeftButton`：
```javascript
//左边按钮
    LeftButton(route, navigator, index, navState) {
        if(index > 0) {
            return (
                <TouchableOpacity
                    onPress={() => {
                        if (route.backClick) {
                            route.backClick(); //如果动作被拦截，那就直接新动作
                        } else {
                            navigator.pop() //否则，pop
                        }
                    }}
                    style={styles.leftButtonStyle}>
                    <Image source={require('../images/trc_pay_pop_btn_back.png')} resizeMode='stretch'/>
                    {
                        iOS
                            ?
                            <Text style={{marginLeft:-6, fontSize:accessoryFontSize}}>返回</Text>
                            :
                            null
                    }
                </TouchableOpacity>
            );
        } else {
            if (route.id === Config.AccountLoginPage.id) {
                return (
                    <TouchableOpacity
                        onPress={() => {
                            if (route.rootBack) { //如果传入根视图返回，就执行新动作
                                return route.rootBack()
                            }else {
                                TRCNativeBridge.dismiss();
                            }
                        }}
                        style={styles.leftButtonStyle}>
                        <Image style={{marginLeft:10}}
                               source={require('../images/trc_account_login_close.png')}
                               resizeMode='stretch' />
                    </TouchableOpacity>
                )
            } else {
                return (
                    <View/>
                );
            }
        }
    },
```
代码解释：
- 如果index > 0，表示当前页面不是根视图，返回按钮基本上都是一个返回箭头"<"，或者"<返回"，点击进行返回。
所以这里定了一个`TouchableOpacity`按钮，上面有一个`Image`。点击按钮执行onPress时：
①如果某个页面需要拦截返回事件，可以在其componentWillMount中给route定义一个`backClick`函数，进行拦截。代码如下：
```javascript
componentWillMount(){
        this.props.route.backClick = () => {
            Keyboard.dismiss();
            const { navigator } = this.props;
            navigator.pop();
        };
    }
```
②如果不需要拦截，则会直接执行`navigator.pop() `，开发者就不需要感知返回事件。

- 如果index = 0，就判断当前视图的`id`是不是根视图。如果是，同上也渲染一个按钮，点击按钮执行onPress时：
①如果某个页面需要拦截返回事件，可以在其componentWillMount中给route定义一个`rootBack`函数，进行返回拦截。
②如果不需要拦截，则会直接执行`TRCNativeBridge.dismiss()`，告诉Native关闭RN模块。

> RightButton与TItle的原理与`LeftButton`类似，就不在赘述。

#### 五、自定义navigationBar
由于使用系统自带的`navigationBar`，会增加代码耦合性，也不利于后期维护，所以只适合页面导航栏定制化较少，功能比较简单的项目。如果导航栏定制化较多，比如需要隐藏导航栏，导航栏上加搜索框等功能时，使用自定义的navigationBar会比较好。

自定义导航栏，也就是写一个公共的导航栏组件，定义好组件样式，为各种情况提供属性和事件callBack，在需要的页面进行引用(几乎每个页面都需要 :flushed:)。
使用示例：
```javascript
import  NavigatorBar  from './NavigatorBar';
export default class PageClass extends Component {
    render() {
        return (
            <View style={{flex:1}}>
                <NavigatorBar navigator={this.props.navigator}
                              title='SecretGarden'
                              hiddenLeftButton={true}  />
            </View>
        )
    }
}
```
优点：真的freeStyle，想怎么定制就怎么定制。
缺点：①使用时比较烦，每次使用都要import；②过渡动画不是很好。


#### 六、适配iPhoneX
网上很多适配iPhoneX的方法。我的方法是：如果是iPhoneX，就把状态栏高度增加24像素，也就是在上面说到的`NavigatorNavigationBarStylesIOS`文件中，修改STATUS_BAR_HEIGHT，如下：
```javascript
var STATUS_BAR_HEIGHT = 20 + (Dimensions.get('window').height === 812 ? 24 : 0); //change by meng, note:适配iPhone X
```

上面只是把状态栏增高，但是还需要将页面顶部向下偏移24像素，页面底部向上偏移34像素。注意：安卓不需要偏移。
```javascript
//顶部偏移
const iOSPaddingTop = 64 + (SCREEN_HEIGHT === 812 ? 24 : 0); //适配iPhone X
const androidPaddingTop = 56;
const paddingTopOffset = global.Android ? androidPaddingTop : iOSPaddingTop;

//底部偏移
const iOSPaddingBottomOffset = SCREEN_HEIGHT === 812 ? 34 : 0; //适配iPhone X
const androidPaddingBottomOffset = 0;
const paddingBottomOffset = global.Android ? androidPaddingBottomOffset : iOSPaddingBottomOffset;

```
这样就完成了iPhoneX的适配。


#### 七、适配安卓沉浸式
安卓沉浸式不是属于Navigator部分，但一般讨论导航栏都会与状态栏联系起来，所以在此顺便说一下。

沉浸式是安卓5.0系统上的新功能。即5.0以上系统，可以设置状态栏透明，页面布局从状态栏顶部开始。
5.0以下系统，状态栏是黑底白字。

适配方法如下：
```javascript
import { StatusBar } from 'react-native';
componentWillMount() {
        if (Android) {
            StatusBar.setBackgroundColor('#f8f8f8');
            StatusBar.setBarStyle('dark-content', true);
        }
}
```
其中背景色设为与`navigationBar`背景色一致。


#### 八、防止快速点击多次push同一界面
在原生iOS上，经常会遇到快速点击一次按钮，同一个页面会push出两次或多次，在RN上也会有这个问题。我的解决方法是：修改`Navigator`导航器源码，在`Navigator`进行push的时候，判断要push的页面与当前栈顶的页面的`id`是不是相同，如果不相同，就push；如果相同，就return。

代码如下：
```javascript
push: function(route) {
      //----------【修改源码开始】change by meng----------
	  const currentRoutes = this.getCurrentRoutes();
      if (currentRoutes.length > 0) {
          let lastRoute = currentRoutes[currentRoutes.length - 1];
          let oldId = lastRoute.id;
          let newId = route.id;
          if (oldId && newId && oldId === newId) {
              //如果是连续push到同一个页面，就直接返回
              return;
          }
      }
      //----------【修改源码结束】----------
	  ...
  }
```

以上就是我在开发过程中使用Navigator的一点心得体会，技术水平有限，若有发现不合理或不准确的地方，欢迎交流指正。

