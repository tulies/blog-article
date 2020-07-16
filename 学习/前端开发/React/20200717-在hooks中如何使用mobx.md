> 如果你想在项目用Mobx，又用到Hooks，那么可以参考看看。

## 创建store

```javascript
import { action, observable } from 'mobx';

class Store {
    @observable
    count = 1;
    
    @action
    setCount = () => {
        this.count++;
    }
}
export const store = new Store();
```

## 在class组件中使用

- 注入store

```javascript
// 注入store
import { Provider } from 'mobx-react';
import {store} from './store';

<Provider store={store}>
  <Demo />
</Provider>
```

- 在class中使用

```javascript
import React, { Component } from 'react';
import { inject, observer } from 'mobx-react';

@inject('scope')
@observer
class Demo1 extends Component { 
    render() {
        return <div>{this.props.count}</div>
    }
}
```

## 在hooks中使用

- 提炼useStore 

```javascript
// @/store/uses.js
import { useLocalStore } from "mobx-react";
import store from "./store";
export const useStore = function () {
  return useLocalStore(() => store);
};
```

- 在hooks中注入

```javascript
import React from 'react';
import { useObserver, Observer} from 'mobx-react';
import { useStore } from "@/store/uses";
// 方法1
function Demo2() { 
    const localStore = useStore();
    return useObserver(() => <div onClick={localStore.setCount}>{localStore.count}</div>)
}

// 方法2，更新Observer包裹的位置，注意这里包裹的必须是一个函数
function Demo3() { 
    const localStore = useStore();
    return <Observer>{() => <span>{localStore.count}</span>}</Observer>
}
```


文中demo代码来自[这里](https://segmentfault.com/a/1190000022335345)，懒得写。我对部分代码做了改动，做了一些提炼，大家参考。


