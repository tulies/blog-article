> 可以选择 `npm run eject` 进行自定义，但是正常我们也不需要改那么多配置，看到多出那么多文件，整个人心情都不好了。还是借助插件进行重写配置吧。

## 借助插件重写react脚手架配置

- 安装 `react-app-rewired` 和 `customize-cra` 。

```shell
    npm i -D react-app-rewired customize-cra
```

- 修改 `package.json` 下的 `scripts` 如下：

```json
"scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test --env=jsdom",
    "eject": "react-app-rewired eject"
}
```

- 项目根目录下创建 `config-overrides.js` 配置文件

```javascript
/* config-overrides.js */
const { override } = require('customize-cra')
module.exports = override(
  //重写的代码
)
```

## 引入less支持

- 安装 `less` 和 `less-loader`

```shell
npm i -D less less-loader
```

- 修改 `config-overrides.js` 的配置

```javascript
/* config-overrides.js */
const { 
    override,
    addLessLoader
} = require('customize-cra')
module.exports = override(
    addLessLoader(), // less插件
)
```


## 引入antd支持，按需加载

- 安装 `antd` 和 `babel-plugin-import`

```shell
npm i -S antd
npm i -D babel-plugin-import
```

- 修改 `config-overrides.js` 的配置

```javascript
/* config-overrides.js */
const {
    override,
    addLessLoader,
    fixBabelImports //按需加载配置函数
  } = require("customize-cra");
module.exports = override(
     fixBabelImports('import', {
      libraryName: 'antd',
      libraryDirectory: 'es',
      style: true  //自动打包相关的样式 默认为 style:'css'
    }),
    // addLessLoader(),
    // 自定义样式
    addLessLoader({
      lessOptions: {
        javascriptEnabled: true,
        modifyVars: { '@primary-color': '#1DA57A' },
      },
    }),
)
```

## 引入mobx支持

- 安装 `mobx` 和 `mobx-react`

```shell
npm i -D mobx mobx-react
```

如果要用装饰器，还需要引入装饰器支持，如下。

## 引入装饰器支持

- 修改 `config-overrides.js` 的配置

```javascript
/* config-overrides.js */
const {
    override,
    addDecoratorsLegacy, // 装饰器支持
  } = require("customize-cra");
module.exports = override(
    addDecoratorsLegacy(), // 装饰器支持
)
```
> 网上说需要安装 `@babel/plugin-proposal-decorators` ， 不过我没有提示需要安装，可能是最新的脚手架自动安装了。
----------

## 总结

本次使用`create-react-app` 脚手架创建了项目，集成了react-router、mobx、antd、less。具体配置如下：

```javascript
// config-overrides.js
// @babel/plugin-proposal-decorators
const {
    override,
    addDecoratorsLegacy,
    addLessLoader,
    fixBabelImports //按需加载配置函数
    // disableEsLint,
    // addBundleVisualizer,
    // addWebpackAlias,
    // adjustWorkbox
  } = require("customize-cra");
//   const path = require("path");
  
  module.exports = override(
    // enable legacy decorators babel plugin
    addDecoratorsLegacy(),
    fixBabelImports('import', {
      libraryName: 'antd',
      libraryDirectory: 'es',
      style: true  //自动打包相关的样式 默认为 style:'css'
    }),
    // addLessLoader(),
    addLessLoader({
      lessOptions: {
        javascriptEnabled: true,
        modifyVars: { '@primary-color': '#1DA57A' },
      },
    }),
    // disable eslint in webpack
    // disableEsLint(),
  
    // add webpack bundle visualizer if BUNDLE_VISUALIZE flag is enabled
    // process.env.BUNDLE_VISUALIZE == 1 && addBundleVisualizer(),
  
    // add an alias for "ag-grid-react" imports
    // addWebpackAlias({
    //   ["ag-grid-react$"]: path.resolve(__dirname, "src/shared/agGridWrapper.js")
    // }),
  
    // adjust the underlying workbox
    // adjustWorkbox(wb =>
    //   Object.assign(wb, {
    //     skipWaiting: true,
    //     exclude: (wb.exclude || []).concat("index.html")
    //   })
    // )
  );

```

```json
// package.json
{
  "name": "piano-ms-client",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^4.2.4",
    "@testing-library/react": "^9.5.0",
    "@testing-library/user-event": "^7.2.1",
    "antd": "^4.3.5",
    "mobx": "^5.15.4",
    "mobx-react": "^6.2.2",
    "react": "^16.13.1",
    "react-app-rewire-mobx": "^1.0.9",
    "react-dom": "^16.13.1",
    "react-router-dom": "^5.2.0",
    "react-scripts": "3.4.1"
  },
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test --env=jsdom",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": "react-app"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "devDependencies": {
    "babel-plugin-import": "^1.13.0",
    "customize-cra": "^1.0.0",
    "less": "^3.11.3",
    "less-loader": "^6.1.3",
    "react-app-rewired": "^2.1.6"
  }
}

```