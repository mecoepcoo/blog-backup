# 前言
用vue和react做开发，我们经常选择vuex，redux一类的状态管理工具来辅助管理状态，状态逻辑复杂的微信小程序，如果有状态管理工具的话，可以极大地提高开发效率和可维护性。

想象这样一个场景，一个用户修改了用户名，小程序中有十多个组件都用到了这个“用户名”这个状态，如果需要把这些显示都更新，用原生的方式来实现是很麻烦的，本文介绍一种方法，基于[rxjs](https://rxjs.dev/)来管理小程序中的各种状态。

rxjs学习资料：
- 官网文档[https://rxjs.dev/](https://rxjs.dev/)
- [学习rxjs操作符](https://rxjs-cn.github.io/learn-rxjs-operators/about/) 这篇文章介绍了低版本rxjs的操作符，很实用

# 实现原理


# 引入rxjs
我们尽量避免多余的构建工具，直接使用umd版本的rxjs，下载地址：[https://unpkg.com/rxjs/bundles/rxjs.umd.min.js](https://unpkg.com/rxjs/bundles/rxjs.umd.min.js)，下载好后，把它放在`lib`目录中。

# 实现思路
新建一个`stores`目录，把状态相关的文件都放在里面，在`stores`下新建一个`user.js`，用来管理用户的状态。

新建`services`目录，存放api相关的服务，在`services`中新建`user.js`，用来存放请求用户数据的函数。

demo：
```javascript
// services/user.js
export const userSvc = {
  getUserInfo: () => {
    console.log('执行获取用户信息');
    return 'xiaoming'; // 第一次请求返回的值
  }
};

// stores/user.js
import * as Rx from '../lib/rxjs.umd.min';
import { userSvc } from '../services/user';

const types = {
  UPDATE_USER_INFO: 'UPDATE_USER_INFO'
};

const states = {
  // BehaviorSubject把数据流中的最新值推送给订阅者
  user$$: new Rx.BehaviorSubject(userSvc.getUserInfo())
};

const actions = {
  [types.UPDATE_USER_INFO] () {
    return 'laoming';
  }
};

function dispatch(action, args) {
  switch (action) {
    case types.UPDATE_USER_INFO:
      var newStates = actions[types.UPDATE_USER_INFO]();
      states.user$$.next(newStates);
      break;
  }
}

export { types, states, actions, dispatch };
```

实现的思路是这样的：

首先创建一个`status`，用来保存用户的状态，用户状态是一个`subject`，这样就可以向各个组件中的`订阅`发送`组播`。

然后写一系列的`action`，action的名称表示要执行的动作，在action函数中完成一些异步操作，比如重新获取用户信息，我们之后会把它的返回值广播给所有的订阅。

最后是`dispatch`函数，它描述了如何修改状态值，dispatch根据传入的action名称，执行不同的action和逻辑，最终用`subject.next()`方法把数据广播给每个订阅。

有了这组代码，我们只需要在使用数据的地方订阅就可以了：
```javascript
// 在某个page中
import { states as userStates } from '../../stores/user';
Page({
  data: {
    name: '',
  },
  onLoad: function () {
    // 订阅测试，此时，name的值应该为'xiaoming'
    userStates.user$$.subscribe(data => {
      console.log(data);
      this.setData({
        name: data
      });
    });
  }
});

// 在page的某个组件中
import { states as userStates, types as userTypes, dispatch as userDispatch } from '../../stores/user';
Component({
  data: {
    name: ''
  },

  attached() {
    userStates.user$$.subscribe(data => {
      console.log(data);
      this.setData({
        name: data
      });
    });
    setTimeout(() => {
      userDispatch(userTypes.UPDATE_USER_INFO);
    }, 3000);
  }
});
```

3秒后，将会看到page和component中的name都变成了'laoming'，这样就实现了简单的状态管理。
