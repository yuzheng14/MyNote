# vue 项目中菜单如何与路由结合

中后台项目中与路由结合的侧边菜单非常常见，但如何能够优雅的自动生成菜单并进行路由导航却还是需要耗费一定精力的。本文章将以组件的形式，探讨菜单（不仅仅是中后台项目）应如何与路由相结合。

## 技术栈

* vue@3
* ant-design-vue@4

## 组件

创建 `MyMenu.vue` 组件

```vue
<script setup lang="ts">
defineOptions({
  name: 'MyMenu',
})

</script>
```

## 路由

首先我们需要父组件传入需要渲染的路由进行渲染，所以我们需要一个 `routes` 属性

```vue
<script setup lang="ts">
import { PropType } from 'vue'
import { RouteRecordRaw } from 'vue-router'

const props = defineProps({
  routes: {
    type: Array as PropType<RouteRecordRaw[]>,
  },
})
</script>

<template>
  <Menu :items="routes" />
</template>
```

但是显然 `routes` 不能直接直接传给 `Menu` 的 `item` 属性，所以我们需要通过计算属性进行一点处理

```vue
<script setup lang="ts">
import { computed } from 'vue'
import { Menu, ItemType } from 'ant-design-vue'

function convertRoutesToItems(routes: RouteRecordRaw[] | undefined): ItemType[] | undefined {
  return routes?.map((route) => ({
    key: route.path,
    label: route.name ?? route.path,
    children: convertRoutesToItems(route.children),
  }))
}

const items = computed<ItemType[] | undefined>(() => convertRoutesToItems(props.routes))
</script>

<template>
  <Menu v-if="routes" :items="items" />
  <Empty v-else />
</template>

```

现在我们就可以通过路由生成一个简易的菜单了

```typescript
// 路由 packages/example/src/router/router.ts
import { createRouter, createWebHashHistory, RouteRecordRaw } from 'vue-router'

export const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: () => import('../layout/BasicLayout.vue'),
    children: [
      {
        path: 'blank',
        name: '空白',
        component: () => import('../pages/BlankPage.vue'),
      },
      {
        path: 'welcome',
        name: '欢迎',
        component: () => import('../pages/WelcomePage.vue'),
      },
      {
        path: 'submenu',
        name: '子菜单',
        children: [
          {
            path: 'submenu1',
            component: () => import('../pages/subMenu/SubMenu1.vue'),
          },
          {
            path: 'submenu2',
            component: () => import('../pages/subMenu/SubMenu2.vue'),
          },
        ],
      },
    ],
  },
]

export const router = createRouter({
  routes,
  history: createWebHashHistory(),
})
```

```vue
<!-- 布局组件 packages/example/src/layout/BasicLayout.vue  -->

<script setup lang="ts">
import { MyMenu } from 'components'
import { Layout, LayoutContent, LayoutSider } from 'ant-design-vue'
import { routes } from '../router/route'

defineOptions({
  name: 'BasicLayout',
})
</script>

<template>
  <Layout class="full-screen">
    <LayoutSider theme="light">
      <MyMenu :routes="routes[0].children" />
    </LayoutSider>
    <LayoutContent>
      <RouterView />
    </LayoutContent>
  </Layout>
</template>

<style scoped>
.full-screen {
  width: 100vw;
  height: 100vh;
}
</style>

```

在上边这个简单的实例里面我们就可以看到渲染出来的菜单了

![渲染路由](./vue%20%E9%A1%B9%E7%9B%AE%E4%B8%AD%E8%8F%9C%E5%8D%95%E5%A6%82%E4%BD%95%E4%B8%8E%E8%B7%AF%E7%94%B1%E7%BB%93%E5%90%88.assets/%E6%B8%B2%E6%9F%93%E8%B7%AF%E7%94%B1.png)

## 点击跳转

仅仅根据路由渲染出来组件还远远不够，因为我们还需要点击菜单项的时候跳转路由。现在我们给 `Menu` 增加上 `click` 事件监听

```vue
<script setup lang="ts">
import { useRouter } from 'vue-router'
import { Menu } from 'ant-design-vue'
import { MenuInfo } from 'ant-design-vue/es/menu/src/interface'

const router = useRouter()

function handleClick(info: MenuInfo) {
  info.keyPath && router.push('/' + info.keyPath.join('/'))
}
</script>

<template>
  <Menu :items="items" @click="handleClick" mode="inline" />
</template>
```

现在点击菜单项就可以进行路由跳转了

![简单路由跳转](./vue%20%E9%A1%B9%E7%9B%AE%E4%B8%AD%E8%8F%9C%E5%8D%95%E5%A6%82%E4%BD%95%E4%B8%8E%E8%B7%AF%E7%94%B1%E7%BB%93%E5%90%88.assets/%E7%AE%80%E5%8D%95%E8%B7%AF%E7%94%B1%E8%B7%B3%E8%BD%AC.png)

## 补充菜单信息

从 `ant-design-vue` 的官网上可以查到，`ItemType` 是有很多字段的，所以我们需要对 `RouteRecordRaw` 类型进行扩充。恰好 `vue-router` 提供了一个 `RouteMeta` 的类型，可以让我们扩充 `RouterRecordRaw` 的 `meta` 字段。

在入口文件中对 `RouterMeta` 类型进行扩充：

```typescript
import { MenuItemType } from 'ant-design-vue/es/menu/src/hooks/useItems'

declare module 'vue-router' {
  interface RouterMeta {
    myMenu: Partial<MenuItemType>
  }
}

export { default as MyMenu } from './MyMenu.vue'
```

⚠️ 注意这里需要从 `ant-design-vue/es/menu/src/hooks/useItems` 中导入 `MenuItemType`，在这里我们不用 `ItemType` 的原因是 `ItemType` 是个联合类型，由菜单项、子菜单、分割线和菜单组四种类型组成，由于我们的菜单是有路由生成的，所以暂时无法支持分割线和菜单组，子菜单我们通过 `RouteRecordRaw` 的 `children` 字段实现了，所以也不需要子菜单的类型，所以我们只需要 `MenuItemType` 的类型，至于从 `ant-design-vue/es/menu/src/hooks/useItems` 导入的原因是 `ItemType` 的定义就在这个文件，另一个文件的 `MenuItemType` 的类型有些不一样。

> 扩展：`export type ItemType = MenuItemType | SubMenuType | MenuItemGroupType | MenuDividerType | null;`

现在我们可以修改下路由转菜单的逻辑了

```vue
<script>
function convertRoutesToItems(routes: RouteRecordRaw[] | undefined): ItemType[] | undefined {
  return routes?.map((route) => ({
    danger: route.meta?.myMenu?.danger,
    disabled: route.meta?.myMenu?.disabled,
    icon: route.meta?.myMenu?.icon,
    key: route.path,
    label: route.meta?.myMenu?.label ?? route.name ?? route.path,
    title: route.meta?.myMenu?.title ?? route.name?.toString() ?? route.path,
    children: convertRoutesToItems(route.children),
  }))
}
</script>
```

现在我们可以在路由里设置菜单的属性了。

![补充菜单信息](./vue%20%E9%A1%B9%E7%9B%AE%E4%B8%AD%E8%8F%9C%E5%8D%95%E5%A6%82%E4%BD%95%E4%B8%8E%E8%B7%AF%E7%94%B1%E7%BB%93%E5%90%88.assets/%E8%A1%A5%E5%85%85%E8%8F%9C%E5%8D%95%E4%BF%A1%E6%81%AF.png)

## 隐藏菜单

## 计算选中菜单并修改跳转逻辑

## 拓展应用场景（可嵌入）

## 计算展开的菜单及手风琴模式