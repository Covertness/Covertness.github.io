title: React初探——前端面向对象设计的最佳实现
date: 2018/05/21 19:48:34
categories:
- 学习记录
tags:
- React
- 面向对象

---
![](https://image.covertness.me/react_chutan/640px-React-icon.svg.png)

React 是一个用于构建用户界面的前端框架，其简单易用的特性让它逐渐成为开发前端页面的主流。本文将通过一个简单的购物车例子介绍 React 那些简单而高效的特性。
<!-- more -->

## 为什么使用 React
### 面向对象的设计理念
React 组件化的开发模型汲取了众多面向对象编程的最佳实践，通过它很容易就可以实现一个高内聚低耦合的应用。 React 通过 JSX 将要显示的元素和它们之间相互的逻辑内聚在一个 Component 里，然后通过 Props 和 Events 与其他 Component 交互，而内部状态的变化通过 State 来记录更新，这一整套流程对于有一定面向对象编程经验的开发者来说都不会陌生。

### 最少的约束
不同于 [Angular](https://angular.io)、 [Vue](https://vuejs.org) 等框架， React 没有定义太多的语法规则让开发者去遵守，这不仅使得初学者极易上手，而且提供了足够的空间让开发者进行个性化地探索。它既可以和现有的框架融合，也可以开发一些像 [Redux](https://redux.js.org/) 之类的新工具与它结合，构建更为复杂和强大的应用。

### 活跃的社区支持
React 这些简单而高效的特性吸引了很多的开发者贡献于其中，让后来者能够站在他们的肩膀上在更短的时间内开发出更加强大的应用。

## 示例——购物车
购物车是大部分拥有购买功能的应用必备的组件，在这里用户可以对所购买的商品做最后的核算。购物车大多拥有一个商品的列表，在这里陈列着用户已选好的商品，很多时候用户需要对商品的数量进行增减，所以对于每个商品条目还需要一个可以选择数量的输入框，除此之外用户一般都会关心此次购物的花销，因而这里还需要实时显示出商品总的价钱。

通过上述描绘我们大概可以脑补出这个购物车的三大组件：商品列表、数量输入框和总价。下面就通过 React 来实现这个简单的购物车，如果想先看看购物车长什么样，可以[访问这里](https://github.com/Covertness/cart)查看截图和完整的代码。

### 创建 React 应用
通过 [Create React App](https://github.com/facebookincubator/create-react-app) 可以快速地创建 React 应用，使用方法如下：
```bash
$ npm install -g create-react-app
$ create-react-app cart
```
这样我们就创建好了一个名叫 `cart` 的 React 应用，进入 `cart` 文件夹执行：
```bash
$ npm start
```
稍等片刻 React 开发环境就准备就绪了，对源文件的任何修改都会实时的反应在浏览器的页面上，非常方便。

### SpinCountInput —— 第一个 Component
打开 `src` 文件夹创建一个叫 `SpinCountInput.js` 的文件，内容如下：
```js
import React from "react";
import PropTypes from "prop-types";
import { withStyles } from "@material-ui/core/styles";
import Button from "@material-ui/core/Button";
import Input from "@material-ui/core/Input";
import AddIcon from "@material-ui/icons/Add";
import RemoveIcon from "@material-ui/icons/Remove";

const styles = theme => ({
    button: {
        margin: theme.spacing.unit
    }
});

class SpinCountInput extends React.Component {
    handleCountChange = (event) => {
        const count = event.target.value;

        if (count.length > 0 && !/^\d+$/.test(count)) return;
        this.props.onCountChange(count);
    }

    handleAdd = () => {
        this.props.onCountChange(this.props.count + 1);
    }

    handleRemove = () => {
        const count = this.props.count - 1;
        if (count < 0) return;

        this.props.onCountChange(count);
    }

    render() {
        const { classes } = this.props;
        return (
            <div>
                <Button mini={true} variant="fab" aria-label="remove" className={classes.button} onClick={this.handleRemove}>
                    <RemoveIcon />
                </Button>
                <Input value={this.props.count} onChange={this.handleCountChange} />
                <Button mini={true} variant="fab" color="primary" aria-label="add" className={classes.button} onClick={this.handleAdd}>
                    <AddIcon />
                </Button>
            </div>
        )
    }
}


SpinCountInput.propTypes = {
    classes: PropTypes.object.isRequired
};

export default withStyles(styles)(SpinCountInput);
```
从 `SpinCountInput` 这个名字中可以猜到这大概就是实现数量输入框功能的代码了，整段代码结构清晰简洁，甚至不需要阅读 React 的文档我们也可以大概知道各部分的功能。 `render` 可以看作是 React Component 的入口，阅读内部实现可以知晓这个 Component 由两个按钮和一个输入框构成，按钮实现了增减数量的功能，输入框用来直接修改数量。除此之外值得一提的是任何对数量的修改最后调用的都是 `this.props.onCountChange` ，通过这个接口将数量传到之后会提到的上层 Component 处理，这是 React 推荐的数据同步机制，通过这样的方法就可以达到数据绑定的效果，详见后文。

### 使用 Lists 填满购物车
接着在同一级目录下创建另外一个源文件 `Cart.js` ，内容如下：
```js
import React from 'react';
import PropTypes from 'prop-types';
import { withStyles } from '@material-ui/core/styles';
import List from '@material-ui/core/List';
import ListItem from '@material-ui/core/ListItem';
import ListItemText from '@material-ui/core/ListItemText';
import Avatar from '@material-ui/core/Avatar';
import SpinCountInput from './SpinCountInput';

const styles = theme => ({});

function Cart(props) {
    const { classes, items } = props;

    function handleChange(index, count) {
        props.onItemCountChange(index, count);
    }

    const listItems = items.map((item, index) =>
        <ListItem key={item.id}>
            <Avatar src={item.image} />
            <ListItemText primary={item.title} secondary={"价格： " + item.price.toFixed(2)} />
            <SpinCountInput count={item.count} onCountChange={handleChange.bind(this, index)} />
        </ListItem>
    );

    return (
        <div className={classes.root}>
            <List>{listItems}</List>
        </div>
    );
}

Cart.propTypes = {
    classes: PropTypes.object.isRequired,
};

export default withStyles(styles)(Cart);
```
这个 Component 位于之前的那个之上，用来产出商品列表，列表的每一列都会包含一个数量输入框，如何区别各个输入框呢， React 通过 `key` 来予以区分，这里我们使用 `id` 作为 `key` 。

### 数据绑定
通过前两个 Component 的交互方式可以看出 React 使用 `props` 和 `events` 实现数据传递，上层组件通过 `props` 将数据交给下层组件，下层组件透过 `events` 将改动后的数据反馈给上层组件，最终数据只会在上层组件中存在，任何对数据的改动都会汇总到上层组件，这一机制在 React 中叫做 Lifting State Up ，在其他的一些前端框架中也叫做数据绑定。

这个例子中的最上层组件在 `App.js` 中，内容如下：
```js
import React from 'react';
import { withStyles } from '@material-ui/core/styles';
import FormLabel from '@material-ui/core/FormLabel';
import Cart from './Cart';

const styles = theme => ({
    total: {
        'float': 'right',
        'margin-right': 40,
        'font-size': '2rem'
    },
});

class App extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            items: props.products
        };
    }

    handleChange = (index, count) => {
        const items = this.state.items;
        items[index].count = count;
        this.setState({ items });
    }

    render() {
        const { classes } = this.props;

        return (
            <div>
                <Cart items={this.state.items} onItemCountChange={this.handleChange} />
                <FormLabel className={classes.total}>合计： {this.state.items.reduce((acc, item) => acc + item.count * item.price, 0).toFixed(2)}</FormLabel>
            </div>
        );
    }
}

export default withStyles(styles)(App);
```
我们将数据 `props.products` 都存在了这个 Component 的 `state` 中， React 针对 `state` 进行了一些封装处理，使它能够支持局部自动更新界面，使用很方便。在这个例子中这一特性最直观的体验便是更新任何商品的数量，都会看到合计的价钱同步更新。需要说明的是这里为了让代码更存粹将具体的示例数据放到了 `index.js` 中，真实的项目里数据大多都会从后端的接口获取到。

### 调试
每次对源文件更改保存后 React 的脚手架都会自动刷新浏览器呈现最新的效果，此外还可以安装 React 的[浏览器插件](https://github.com/facebook/react-devtools)，这个工具可以在浏览器的调试窗口中显示出 Component 的对应关系方便调试。