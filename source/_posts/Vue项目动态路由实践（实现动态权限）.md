---
title: Vue项目动态路由实践（实现动态权限）
tags:
  - 前端
  - Vue
categories:
  - 笔记
toc: false
date: 2020-06-18 15:10:48
---

# Vue项目动态路由实践（实现动态权限）

## 问题来源

管理端项目经常有需要根据账号角色、权限动态地修改菜单显示的需求，首先是登录时从服务端获取账号对应的角色、权限、菜单等信息，还有一个是在切换账号、切换角色的时候动态改变菜单权限

## 其他参考

Vue项目大多集成了vue-router，用于管理路由信息，用一个VueRouter对象来描述项目的所有路由关系，通过<router-view>标签来形成多层次的路由关系（单页面应用），但是VueRouter描述的routes是固定的，改变路由的API只有`addRoutes`，只能新增不能移除，会导致菜单权限多的账号切换到菜单权限少的账号时还是可以通过地址访问无权限的地址，只能通过前端页面来限制

## 折中的解决方法

1.账号层面的权限限制：通过在切换账号或角色的时候addRoutes，新增原先没有权限的页面
2.页面级的权限限制：在路由守卫判断用户权限，判断是放行还是指向401页面
3.控件级的权限限制：参考vue-element-admin,使用自定义指令来判断是否有权限

## 具体实现

### 1.在vuex中保存权限和菜单
```js
export default new Vuex.Store({
  state: {
    user: {
      info: {},
      //角色
      roles: [],
      //当前用户菜单权限
      menu: [],
      //全部菜单权限,用于比较
      allMenu: [],
      //自己维护一份动态路由表,用于菜单权限
      generatedRoutes: undefined,
    },
 }
}
```
### 2.在路由守卫中判断是否已经生成新的路由表

先判断是否登录，在判断是否已经获取用户信息（权限、菜单），注意下对登录页面的特殊判断
```js
router.beforeEach(async (to, from, next) => {
  if (await store.dispatch('isLogin') || to.path === '/login') {
    document.title = to.name
    if ( !await store.dispatch('hasGetUserInfo') && to.path !=='/login') {
      await store.dispatch('login')
      next({ ...to, replace: true })
    } else {
      //TODO 这里需要判断页面权限再放行
      next()
    }

  } else {
    next({
      path: '/login',
      query: {
        redirect: to.path,
        ...to.query,
      },
    })
  }
})
```