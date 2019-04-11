# vue-router-invoke-webpack-plugin

根据文件格式自动生成`vue-router`的路由

## 下载

### npm

```javascript
npm install vue-router-invoke-webpack-plugin -D
```

### cnpm

```javascript
cnpm install vue-router-invoke-webpack-plugin -D
```

### yarn

```javascript
yarn add vue-router-invoke-webpack-plugin -D
```

## 为什么使用`vue-router-invoke-webpack-plugin`

在单页面我们会使用`vue-router`作为路由切换的插件,但当你的页面过多比如有 50 或者 60 个页面的时候,你的`vue`文件和最后写入的`router.js`会很复杂.为了更友好的应对我们会把页面按功能分一下,相同功能的页面放在一个文件夹中,之前我们也是这么做的,但后来发现了很多问题,特别是项目在多人合作的时候

- 命名规则

没有统一的命名,有人喜欢下划线,有人喜欢用驼峰

![image](https://github.com/Qymh/vue-router-invoke-webpack-plugin/blob/master/docs/images/index.png)

- 路由的层级

明明是二级路由或者三级路由,放在了和一级路由一个目录下,且命名完全看不出来路由的层级

![image](https://github.com/Qymh/vue-router-invoke-webpack-plugin/blob/master/docs/images/name.png)

- 新人难以接受

生成的路由太过复杂,且命名不够语义化,无法区分页面层级

其实第一二种问题完全是可以通过`code review`去解决的,但`code review`会有一定的成本,所以我们学习了 [nuxt](https://zh.nuxtjs.org/guide/routing) 中的路由构建,我们不以`页面功能`去区分路由,而是根据文件目录结构以`路由层级`去划分路由,解决了以上三个问题

## 用法

### Webpack

- 我们需要确定当前环境处于开发(`development`)还是生产(`production`)环境,所以你需要设置`process.env.NODE_ENV`在开发环境下为`development`,在生产环境下为`production`.不考虑跨平台的话环境变量是可以直接设置的,但也有很多插件实现了跨平台设置,我们推荐[cross-env](https://github.com/kentcdodds/cross-env)
- 考虑到多人开发,所以引用路由的时候不能通过你的本机绝对路由引入,所以你需要给观察的目录`dir`设置一个别名[alias](https://webpack.js.org/configuration/resolve/#resolvealias)
- 自动构建的路由是懒加载的,所以你需要引用一个 babel 插件[@babel/plugin-syntax-dynamic-import](https://babeljs.io/docs/en/next/babel-plugin-syntax-dynamic-import.html)

```javascript
const VueRouterInvokeWebpackPlugin = require('vue-router-invoke-webpack-plugin');
const path = require('path')

// 省略掉其他配置...

resolve: {
  alias: {
    '@': path.resolve(process.cwd(), 'demos')
  }
}

plugins: [
  new VueRouterInvokeWebpackPlugin(
    dir: 'demos/src',
    alias: '@/src'
  )
];
```

### VueCli3

vuecli3 会比 webpack 配置容易点

`vue.config.js`

```javascript
const VueRouterInvokeWebpackPlugin = require('vue-router-invoke-webpack-plugin');

module.exports = {
  // 省略掉其他配置...
  configureWebpack(config) {
    config.plugins.push(
      new VueRouterInvokeWebpackPlugin({
        dir: 'src/views',
        // 必须设置dir配置的别名
        alias: '@/views'
      })
    );
  }
};

// 或者采用另外一个方法

module.exports = {
  // 省略掉其他配置...
  configureWebpack: {
    plugins: [
      new VueRouterInvokeWebpackPlugin({
        dir: 'src/views',
        // 必须设置dir配置的别名
        alias: '@/views'
      })
    ]
  }
};
```

### Start

在配置好后,你可以通过`npm run serve`或者你定义的其他命令在开发模式下激活插件,当第一次运行或`dir`观察的目录发生变化时,`router.js`会被自动构建

同样的在生产环境下,你可以通过`npm run build`或者你定义的其他命令激活插件,`router.js`会被自动构建和打包

### 配置

| Prop           |   Type   | Required |  Default   |                Description |
| -------------- | :------: | :------: | :--------: | -------------------------: |
| dir            |  String  |   true   |     ''     |        观察的 vue 文件目录 |
| alias          |  String  |   true   |     ''     |  观察的 vue 文件目录的别名 |
| notFound       |  String  |  false   |     ''     |                   404 路由 |
| mode           |  String  |  false   |  history   |          hash 或者 history |
| meta           |  String  |  false   |    meta    |    定义 meta 的 yml 文件名 |
| routerDir      |  String  |  false   |    ROOT    |  构建后的 router.js 的位置 |
| language       |  String  |  false   | javascript | javascript 或者 typescript |
| ignore         |  Array   |  false   |     []     |       忽略的文件或者文件夹 |
| redirect       |  Array   |  false   |     []     |                 重定向路由 |
| modules        |  Array   |  false   |     []     |             导入的其他模块 |
| scrollBehavior | Function |  false   |     ''     |          同 scrollBehavior |
| beforeEach     | Function |  false   |     ''     |          router.beforeEach |
| beforeResolve  | Function |  false   |     ''     |       router.beforeResolve |
| afterEach      | Function |  false   |     ''     |           router.afterEach |

## 如何使用自动注入

下面将会使用 vuecli3 作为列子

`vue.config.js`

```javascript
const VueRouterInvokeWebpackPlugin = require('vue-router-invoke-webpack-plugin');

module.exports = {
  // 省略其他配置
  configureWebpack(config) {
    config.plugins.push(
      new VueRouterInvokeWebpackPlugin({
        dir: 'src/views',
        alias: '@/views'
      })
    );
  }
};
```

然后在入口文件`src/main.js`中引用构建好的`router.js`

默认生成的`router.js`的位置在项目根路由的`.invoke`文件夹中,你可以通过`routerDir`这个配置更改默认位置

```javascript
import Vue from 'vue';
import App from './App.vue';
import router from '../.invoke/router';

export default new Vue({
  el: '#app',
  router,
  render: h => h(App)
});
```

### 单路由

如果你的文件是这样的

```
src
├── views
│   ├── Login
│   │   └── Index.vue
│   └── User
│       ├── Account
│       │   └── Index.vue
│       ├── Home
│       │   └── Index.vue
│       └── Index.vue
```

那么自动生成的路由会是这样

```javascript
{
  component: () =>
    import('@/views/Login/Index.vue'),
  name: 'login',
  path: '/login'
},
{
  component: () =>
    import('@/views/User/Index.vue'),
  name: 'user',
  path: '/user'
},
{
  component: () =>
    import('@/views/User/Account/Index.vue'),
  name: 'user-account',
  path: '/user/account'
},
{
  component: () =>
    import('@/views/User/Home/Index.vue'),
  name: 'user-home',
  path: '/user/home'
}
```

### 首页

我们对首页做了特殊处理,首页直接用`Index.vue`表示

如果你的文件是这样的

```
src
├── views
│   ├── Login
│   │   └── Index.vue
│   └── Index.vue
```

那么自动生成的路由会是这样

```javascript
{
  component: () =>
    import('@/views/Index.vue'),
  name: 'index',
  path: '/'
},
{
  component: () =>
    import('@/views/Login/Index.vue'),
  name: 'login',
  path: '/login'
}
```

### 动态路由

如果你的文件是这样的

```
src
├── views
│   ├── Login
│   │   └── Index.vue
│   └── User
│       ├── :Home
│       │   └── Index.vue
│       └── Index.vue
```

那么自动生成的路由会是这样

```javascript
{
  component: () =>
    import('@/views/Login/Index.vue'),
  name: 'login',
  path: '/login'
},
{
  component: () =>
    import('@/views/User/Index.vue'),
  name: 'user',
  path: '/user'
},
{
  component: () =>
    import('@/views/User/:Home/Index.vue'),
  name: 'user-home',
  path: '/user/:home'
}
```

### 嵌套路由

如果你的文件是这样的

```
src
├── views
│   ├── Login
│   │   └── Index.vue
│   └── User
│       ├── Chart
│       │   └── Index.vue
│       ├── Home
│       │   └── Index.vue
│       └── User.vue
```

那么自动生成的路由会是这样

```javascript
{
  component: () =>
    import('@/views/Login/Index.vue'),
  name: 'login',
  path: '/login'
},
{
  component: () =>
    import('@/views/User/User.vue'),
  name: 'user',
  path: '/user',
  children: [
    {
      component: () =>
        import('@/views/User/Chart/Index.vue'),
      name: 'user-chart',
      path: 'chart'
    },
    {
      component: () =>
        import('@/views/User/Home/Index.vue'),
      name: 'user-home',
      path: 'home'
    }
  ]
}
```

### 动态嵌套路由

如果你的文件长这样

```
src
├── views
│   ├── Login
│   │   └── Index.vue
│   └── User
│       ├── :Category
│       │   ├── :Category.vue
│       │   └── Infor
│       │       └── Index.vue
│       └── Index.vue
```

那么自动生成的路由会是这样

```javascript
{
    component: () =>
      import('@/views/Login/Index.vue'),
    name: 'login',
    path: '/login'
  },
  {
    component: () =>
      import('@/views/User/Index.vue'),
    name: 'user',
    path: '/user'
  },
  {
    component: () =>
      import('@/views/User/:Category/:Category.vue'),
    name: 'user-category',
    path: '/user/:category',
    children: [
      {
        component: () =>
          import('@/views/User/:Category/Infor/Index.vue'),
        name: 'user-category-infor',
        path: 'infor'
      }
    ]
  }
```

## 命名矫正

在为什么使用`vue-router-invoke-webpack-plugin`中提到过命名不统一的问题,我们会将所有不同的命名转化为驼峰命名,这只是一个小的命名矫正,根本的方法应该是统一命名规则

举个列子

```
src
├── views
│   ├── LoginPage
│   │   └── index.vue
│   └── User-home
│       ├── account
│       │   └── Index.vue
│       ├── Home-details
│       │   └── Index.vue
│       └── Index.vue
```

矫正命名后的路由会是这样

```javascript
{
  component: () => import('@/views/LoginPage/index.vue'),
  name: 'loginPage',
  path: '/loginPage'
},
{
  component: () => import('@/views/User-home/Index.vue'),
  name: 'userHome',
  path: '/userHome'
},
{
  component: () => import('@/views/User-home/Home-details/Index.vue'),
  name: 'userHome-homeDetails',
  path: '/userHome/homeDetails'
},
{
  component: () => import('@/views/User-home/account/Index.vue'),
  name: 'userHome-account',
  path: '/userHome/account'
},
```

## meta 替代品

`meta`属性可以解决很多问题,比如定义当前页面标题或者给一个字段判断当前页面是否需要登录.

定义标题之类的可以通过[vue-meta](https://github.com/nuxt/vue-meta)解决

但如果你需要定义`meta`属性的话,需要写一个`yml`文件

举个列子

```javascript
src/views
├── Single
│   ├── Index.vue
│   └── User
│       ├── Index.vue
│       └── meta.yml
```

`meta.yml`

```yml
meta:
  - name: user
```

自动构建后的路由会是这样

```javascript
{
  component: () => import('@/views/Single/Index.vue'),
  name: 'single',
  path: 'single'
},
{
  component: () => import('@/views/Single/User/Index.vue'),
  name: 'single-user',
  meta: { name: user },
  path: 'single/user'
}
```

## 特殊的配置

### 404 路由

举个列子

```javascript
plugins: [
  new VueRouterInvokeWebpackPlugin({
    dir: 'src/views',
    alias: '@/views',
    // muse set ignore for notFound chunk
    ignore: ['NotFound.vue'],
    notFound: '@/views/NotFound.vue'
  })
];
```

```
src
├── views
│   ├── Login
│   │   └── Index.vue
│   └── Index.vue
│   └── NotFound.vue

```

自动构建后的路由会是这样

```javascript
{
  component: () =>
    import('@/views/Index.vue'),
  name: 'index',
  path: '/'
},
{
  component: () =>
    import('@/views/NotFound.vue'),
  name: 'notFound',
  path: '*'
},
{
  component: () =>
    import('@/views/Login/Index.vue'),
  name: 'login',
  path: '/login'
}
```

### 忽略文件和文件夹

假设你的设置是这样

```javascript
plugins: [
  new VueRouterInvokeWebpackPlugin({
    dir: 'src/views',
    alias: '@/views',
    language: 'javascript',
    ignore: ['images', 'components', 'template.vue']
  })
];
```

自动构建的路由将会不区分大小写的忽略掉`images` `components` `template.vue`

the directory

```
src
├── views
│   ├── Login
│   │   └── Index.vue
│   ├── Template.vue
│   └── User
│       ├── Components
│       ├── Images
│       └── Index.vue
```

```javascript
{
  component: () =>
    import('@/views/Login/Index.vue'),
  name: 'login',
  path: '/login'
},
{
  component: () =>
    import('@/views/User/Index.vue'),
  name: 'user',
  path: '/user'
}
```

### 重定向

假设你的配置如下

```javascript
plugins: [
  new VueRouterInvokeWebpackPlugin({
    dir: 'src/views',
    alias: '@/views',
    language: 'javascript',
    redirect: [
      {
        redirect: '/',
        path: '/home'
      },
      {
        redirect: '/test',
        path: '/demo'
      }
    ]
  })
];
```

自动构建的路由会长这样

```javascript
{
  path: '/home',
  redirect: '/'
},
{
  path: '/demo',
  redirect: '/test'
}
```

### 模块

自动生成的`router.js`有两个模块 `vue` `vue-router`

```javascript
import Vue from 'vue';
import Router from 'vue-router';
```

如果你需要其他模块用在`beforeEach`之类的方法中,你需要手动加上,举个列子

```javascript
new VueRouterInvokeWebpackPlugin({
  dir: 'src/views',
  alias: '@/views',
  modules: [
    {
      name: 'diyName',
      package: 'some-packages'
    }
  ]
});
```

自动生成的路由会长这样

```javascript
// 省略其它配置
import diyName from 'some-packages';
```

### vue 路由全局守卫

如果你的配置如下

```javascript
new VueRouterInvokeWebpackPlugin({
  dir: 'src/views',
  alias: '@/views',
  language: 'javascript',
  beforeEach: (to, from, next) => {
    next();
  },
  beforeResolve: (to, from, next) => {
    next();
  },
  afterEach: (to, from) => {}
});
```

自动生成的路由会是这样

```javascript
// 省略其它配置
const router = new Router({ mode: 'history', routes });
router.beforeEach((to, from, next) => {
  next();
});

router.beforeResolve((to, from, next) => {
  next();
});

router.afterEach((to, from) => {});
export default router;
```

### ScrollBehavior

如果你的配置如下

```javascript
new VueRouterInvokeWebpackPlugin({
  dir: 'src/views',
  alias: '@/views',
  language: 'javascript',
  scrollBehavior: (to, from, savedPosition) => {
    if (savedPosition) {
      return savedPosition;
    } else {
      return { x: 0, y: 0 };
    }
  }
});
```

自动生成的路由会是这样

```javascript
// 省略其他配置
const router = new Router({
  mode: 'history',
  routes,
  scrollBehavior: (to, from, savedPosition) => {
    if (savedPosition) {
      return savedPosition;
    } else {
      return { x: 0, y: 0 };
    }
  }
});
```

## Demos

详细的演示你可以`git clone`克隆我们的项目然后执行`npm run build:demos`,`demo`没有什么内容它的着重点在于`router.js`的构建,你可以看到不同文件目录对应的不同格式
