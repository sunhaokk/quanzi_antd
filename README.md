# quanzi_antd

### 框架首页

[antd首页](https://ant.design/docs/react/introduce-cn)

[antp首页](https://pro.ant.design/docs/getting-started-cn)

[dva首页](https://github.com/dvajs/dva/blob/master/README_zh-CN.md)

[mockjs教程](https://github.com/nuysoft/Mock/wiki/Getting-Started)

[roadhug文档](https://github.com/sorrycc/roadhog/blob/master/README_zh-cn.md)

[react-router](https://reacttraining.com/react-router/web/example/basic)


### 注意

- 安装使用 Yarn
 
- 环境node必须大于 8.0

### 说明

Roadhog 是一个包含 dev、build 和 test 的命令行工具，他基于 react-dev-utils，和 create-react-app 的体验保持一致。你可以想象他为可配置版的 create-react-app。

roadhog 的 webpack 部分功能是基于 af-webpack 实现的 。而 af-webpack 是支付宝团队封装的 webpack。

所以 webpack 方面，我们只需要按文档配置 roadhog

### 教程

- 先进 src/common/router.js

会看到 dynamicWrapper 这是一个对'dva/dynamic'的包装。包含所有路由。

```
const dynamicWrapper = (app, models, component) => {
  // () => require('module')
  // transformed by babel-plugin-dynamic-import-node-sync
  if (component.toString().indexOf('.then(') < 0) {
    models.forEach(model => {
      if (modelNotExisted(app, model)) {
        // eslint-disable-next-line
        app.model(require(`../models/${model}`).default);
      }
    });
    return props => {
      if (!routerDataCache) {
        routerDataCache = getRouterData(app);
      }
      return createElement(component().default, {
        ...props,
        routerData: routerDataCache,
      });
    };
  }
  // () => import('module')
  return dynamic({
    app,
    models: () =>
      models.filter(model => modelNotExisted(app, model)).map(m => import(`../models/${m}.js`)),
    // add routerData prop
    component: () => {
      if (!routerDataCache) {
        routerDataCache = getRouterData(app);
      }
      return component().then(raw => {
        const Component = raw.default || raw;
        return props =>
          createElement(Component, {
            ...props,
            routerData: routerDataCache,
          });
      });
    },
  });
};
```

这里接受三个参数 app 模型和组件。

模型存在src/routes文件夹内  组件在src/component文件夹内

其中 app.model(require(`../models/${model}`).default);这句话加载自动加载了需要配置的模型。

- 菜单

src/common/menu.js 配置菜单相关数据，菜单项的跳转链接为配置项及其所有父级配置 path 参数的拼接。

为 src/common/router.js 提供路由名称（name）等数据，根据拼接好的跳转链接来匹配相关路由。

我们通过传入 menu 到 getFlatMenuData 递归实现菜单对象树 menuData。

返回的对象结构

```
Object
/dashboard
:
{name: "dashboard", icon: "dashboard", path: "/dashboard", children: Array(3), authority: undefined}
/dashboard/analysis
:
{name: "分析页", path: "/dashboard/analysis", authority: undefined}
/dashboard/monitor
:
{name: "监控页", path: "/dashboard/monitor", authority: undefined}
/dashboard/workplace
:
{name: "工作台", path: "/dashboard/workplace", authority: undefined}
/exception
:
{name: "异常页", icon: "warning", path: "/exception", children: Array(4), authority: undefined}
/exception/403
:
{name: "403", path: "/exception/403", authority: undefined}
/exception/404
:
{name: "404", path: "/exception/404", authority: undefined}
/exception/500
:
{name: "500", path: "/exception/500", authority: undefined}
/exception/trigger
:
{name: "触发异常", path: "/exception/trigger", hideInMenu: true, authority: undefined}
/form
:
{name: "表单页", icon: "form", path: "/form", children: Array(3), authority: undefined}
/form/advanced-form
:
{name: "高级表单", authority: "admin", path: "/form/advanced-form"}
/form/basic-form
:
{name: "基础表单", path: "/form/basic-form", authority: undefined}
/form/step-form
:
{name: "分步表单", path: "/form/step-form", authority: undefined}
/list
:
{name: "列表页", icon: "table", path: "/list", children: Array(4), authority: undefined}
/list/basic-list
:
{name: "标准列表", path: "/list/basic-list", authority: undefined}
/list/card-list
:
{name: "卡片列表", path: "/list/card-list", authority: undefined}
/list/search
:
{name: "搜索列表", path: "/list/search", children: Array(3), authority: undefined}
/list/search/applications
:
{name: "搜索列表（应用）", path: "/list/search/applications", authority: undefined}
/list/search/articles
:
{name: "搜索列表（文章）", path: "/list/search/articles", authority: undefined}
/list/search/projects
:
{name: "搜索列表（项目）", path: "/list/search/projects", authority: undefined}
/list/table-list
:
{name: "查询表格", path: "/list/table-list", authority: undefined}
/profile
:
{name: "详情页", icon: "profile", path: "/profile", children: Array(2), authority: undefined}
/profile/advanced
:
{name: "高级详情页", path: "/profile/advanced", authority: "admin"}
/profile/basic
:
{name: "基础详情页", path: "/profile/basic", authority: undefined}
/result
:
{name: "结果页", icon: "check-circle-o", path: "/result", children: Array(2), authority: undefined}
/result/fail
:
{name: "失败", path: "/result/fail", authority: undefined}
/result/success
:
{name: "成功", path: "/result/success", authority: undefined}
/user
:
{name: "账户", icon: "user", path: "/user", authority: "guest", children: Array(3)}
/user/login
:
{name: "登录", path: "/user/login", authority: "guest"}
/user/register
:
{name: "注册", path: "/user/register", authority: "guest"}
/user/register-result
:
{name: "注册结果", path: "/user/register-result", authority: "guest"}
__proto__
:
Object
```
``` 
Object.keys(routerConfig).forEach(path => {

...

const menuKey = Object.keys(menuData).find(key => pathRegexp.test(`${key}`));

```
通过 Object.keys(routerConfig) 遍历路由表返回所有的对象名(其实对象名就是 path)。然后把 path 传入函数体内。 通过 Object.keys(menuData) 一个包含所有 菜单树的所有菜单名，然后通过 （find)[https://blog.csdn.net/qq_30100043/article/details/53303768] 遍历数组 返回 (pathRegexp)[https://www.npmjs.com/package/path-to-regexp]测试包裹的 path.


```
function getFlatMenuData(menus) {
  let keys = {};
  menus.forEach(item => {
    if (item.children) {
      keys[item.path] = { ...item };
      keys = { ...keys, ...getFlatMenuData(item.children) };
    } else {
      keys[item.path] = { ...item };
    }
  });
  return keys;
}

const menuData = getFlatMenuData(getMenuData());

  // Route configuration data
  // eg. {name,authority ...routerConfig }
  const routerData = {};
  // The route matches the menu
  Object.keys(routerConfig).forEach(path => {
    // Regular match item name
    // eg.  router /user/:id === /user/chen
    const pathRegexp = pathToRegexp(path);
    const menuKey = Object.keys(menuData).find(key => pathRegexp.test(`${key}`));
    let menuItem = {};
    // If menuKey is not empty
    if (menuKey) {
      menuItem = menuData[menuKey];
    }
    let router = routerConfig[path];
    // If you need to configure complex parameter routing,
    // https://github.com/ant-design/ant-design-pro-site/blob/master/docs/router-and-nav.md#%E5%B8%A6%E5%8F%82%E6%95%B0%E7%9A%84%E8%B7%AF%E7%94%B1%E8%8F%9C%E5%8D%95
    // eg . /list/:type/user/info/:id
    router = {
      ...router,
      name: router.name || menuItem.name,
      authority: router.authority || menuItem.authority,
      hideInBreadcrumb: router.hideInBreadcrumb || menuItem.hideInBreadcrumb,
    };
    routerData[path] = router;
```

总结: 如果我们要添加新的 模块 **首先要卸载路由表上** 。也可以写在菜单表上。 如果重复  优先使用路由表上的配置。

即 如果你要添加个菜单  路由表上的 对象名 一定要和 菜单里的 path 相对应。

添加一个菜单的步骤

在路由表新建路由 =》 在菜单表新建菜单  完毕。很简单







