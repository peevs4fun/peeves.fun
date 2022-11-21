---
title: 自定义组件库及 NPM 发包流程（以 Vue 为例）
date: 2022-11-22 01:32:38
tags: npm
excerpt: 记录一次发布 npm 包的过程体验，仅供参考
---

## 准备  
### NPM 官网  
1. [申请 NPM 账户](https://www.npmjs.com/signup)  
2. 如有多人参与项目可以创建或加入组织方便管理。发布时在包的命名上加入组织名为前缀，发包后会自动归入组织下。组织的另一个好处是可以规避包名与已有 npm 包重名导致发布不了。  
## 项目的建立与配置  
为快速新建项目，建议使用 @vue/cli 3.x 以上版本创建项目，之后的打包也会使用 vue/cli 的库打包模式。  
### 创建项目  
使用命令`vue create my-component`创建新项目，根据需要选择特性。  
*参考：[vue/cli 关于新建项目的部分](https://cli.vuejs.org/zh/guide/creating-a-project.html#vue-create)*  
### package 文件的设置  
新建完成后的项目需要在 `package.json` 文件需要加入或修改以下字段：  
```json
// 包名：@[组织名]/[包名]  (注意组织名和包名不能有大写字母,可以使用连字符 - )
"name": "@my-organization/my-component",
// 包的简介
"description": "一个用于测试 npm 发包的 vue 组件",
// 版本号，详见下方说明
"version": "0.1.0",
// 包作者，原则上与发包 npm 账号保持一致
"author": "laicanwen",
// 设置包为共有，否则无法免费发布
"private": false,
// 设置当前包的文件入口，这个设置决定了从何处引入模块，默认为 src/App.vue，此处指向编译后的模块文件。
// 也可以指向源码的模块输出文件，如'src/index.js'.但该情况下不能在 .npmignore 排除源码。
"main": "dist/my-component.umd.js",
// 开源协议
"license": "GPL-3.0"
```

### 开源协议声明  
将项目所选开源协议声明全文保存为 `LICENSE` 文件。  
以 GPL-3.0 为例：  
*协议全文：[GPL-3.0](https://www.gnu.org/licenses/gpl-3.0-standalone.html)*  
*使用细节参考：[GUN使用指南](https://www.gnu.org/licenses/gpl-howto.html)*  
### 打包工具的选择及配置  
由于使用了`vue/cli`生成项目，此处基本零配置。  
### 项目代码  
#### 项目目录结构建议  

建议目录结构如下： 
```
my-component
│  .gitignore                      // 需要 git 忽略提交的文件，当前目录无 .npmignore 时该文件同样为 npm 忽略发布的规则
│  .LICENSE                        // 项目所使用的的开源协议声明
│  .npmignore                      // 需要 npm 忽略发布的文件,语法与 .gitignore 相同
│  babel.config.js
│  package-lock.json
│  package.json                    // 记录了 npm 的依赖管理,同时记录了该包的名字、作者和入口等信息
│  README.md                       // 请在此处说明包的安装、功能和 API
│  vue.config.js
│  
├─dist
│  │  my-component.common.js       // 遵循 CommonJs 规范的打包文件
│  │  my-component.umd.js          // 遵循 CommonJs、CMD、AMD 规范的打包文件
│  │  my-component.umd.min.js      // 遵循 CommonJs、CMD、AMD 标准的压缩打包文件
│  │  demo.html
│  │  
│  └─static
│      └─fonts
│              element-icons.535877f5.woff
│              element-icons.732389de.ttf
│              
├─public
│      favicon.ico
│      index.html
│      
└─src
    │  App.vue                      // 创建项目时生成的 App 界面入口，可以作为组件的测试界面，但不参与打包上传
    │  index.js                     // 组件的输出文件，同时也是打包指定的入口
    │  main.js                      // 创建项目时生成的 App 相关文件，同样不参与打包
    │  
    ├─assets
    │      logo.png
    │      
    └─components                    // 建议把所有组件都放入该文件夹下统一管理
            Component1.vue          // 编写的组件
            Component2.vue          // 编写的组件
            Component3.vue          // 编写的组件
            

```
#### 组件的测试界面  
可以使用项目生成的 `APP.vue` 用作编写组件或模块的测试界面。  
#### 编写模块出口  
在根目录新建文件 `index.js` 作为编写完成的组件或者模块的出口。  
参考写法：  
``` javascript
export { default as MyComponent } from './components/MyComponent'
```  
#### 如何支持 VUE 插件注册  
依旧是在上文提及的 `index.js` 文件中，写法为：  
```javascript
import { default as MyComponent } from './components/MyComponent'
// 此处的 install 方法会在使用 Vue.use(myProject) 时调用
MyComponent.install = function(Vue) {
  // 判断全局 Vue 对象是否存在
  if (typeof window !== 'undefined' && window.Vue) {
      Vue = window.Vue
  }
  // 将组件在全局注册，然后可以在项目范围内以 <MyComponent> 或 <my-component> 标签方式引用
  Vue.component('MyComponent', MyComponent)
}
export default MyComponent
```  
#### 单项目多模块的写法  
`index.js` 文件的写法改为：  
```javascript
import { default as Component1 } from 'src/components/Component1'
import { default as Component2 } from 'src/components/Component2'
import { default as Component3 } from 'src/components/Component3'
const components = [
  Component1,
  Component2,
  Component3,
]
const install = function(Vue) {
  components.map(component => {
      Vue.use(component)
  })
  // 如果需要注册全局方法也在这里写入 Vue 对象原型
  // Vue.prototype.$message = message
}
/* istanbul ignore if */
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue)
}
export {
  install,
  component1,
  component2,
  component3,
}
```  
## 如何构建包  
### 打包命令  
命令：`vue-cli-service build --target lib --name my-component src/index.js`  
说明：`vue-cli-service build --target [打包模式] --name [包名] [包入口地址]`  
*参见： [vue/cli 官方文档关于库的打包](https://cli.vuejs.org/zh/guide/build-targets.html#%E5%BA%93)*  
### 打包后文件  
建议将打包后的 `my-component.umd.js` 或者 `my-component.umd.min.js` 在 `package.json` 中指定为包的入口。
## 如何发布包  
### package 的配置  
注意需要检查 `main` 字段是否指向打包好的入口文件。 
关于模块的加载规则参考 [webpack 中文文档](https://www.webpackjs.com/concepts/module-resolution/#%E6%A8%A1%E5%9D%97%E8%B7%AF%E5%BE%84)  
### 发布命令  
在终端进入当前项目根目录  
1. 登录 npmjs 账号：`npm login`  
2. 发布命令：`npm publish`  
3. 如果你需要发布一个**位于组织下**的包，则需要在发布命令**显示声明**该包为公开包：  
`npm publish --access public`  
4. 发布后即可在 [npmjs 官网](https://www.npmjs.com/)搜索到刚发布的包或者在新项目中使用 `npm install` 命令安装发布出来的包。  
###  如何彻底撤销已发布的包  
使用命令 `npm unpublish [<@scope>/]<pkg> --force`。  
或者在根目录使用命令 `npm unpublish --force` 可以强制撤销当前包的**所有版本**，请慎用。  
## 关于包的更新  
### 版本号说明  
```json
// package.json
"version":"x.y.z"
```  
`version` 字段结构是有三位的版本号, 对应为 major, minor, patch。  
如版本号为`0.0.1`的包，发布大版本更新（major）的时候会升级为 `1.0.0`，小版本更新（minor）是`0.1.0`，一些小修复更新（patch）是`0.0.2`。  
### 如何更新包  
- 使用命令更新版本： 
1. 在本地更新这个包的版本,使用命令 `npm version <update_type>`，如 `npm version major`。此时 `package.json` 的 `version` 字段已变更；
2. 提交到远端 npm 中，使用命令 `npm publish`  
- 直接修改 `package.json` 文件
1. 每次更新前 `package.json` 文件的 `version` 字段更改版本号。  
2. 提交到远端 npm 中，使用命令 `npm publish`  
### 如何撤销指定版本的包  
命令：`npm unpublish [<@scope>/]<pkg>[@<version>]`  
如： `npm unpublish @my-organization/my-component@0.0.1`  
参考 [npm 文档](https://docs.npmjs.com/cli/unpublish)关于 `unpublish` 命令的部分  
## 参考资料
[npm 文档原文](https://docs.npmjs.com/cli-documentation/)
[vue/cli 文档](https://cli.vuejs.org/zh/guide/)
[webpack 中文文档](https://www.webpackjs.com/)  

