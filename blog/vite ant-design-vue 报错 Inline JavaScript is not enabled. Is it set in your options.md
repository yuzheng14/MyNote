# Vite ant-design-vue 报错 Inline JavaScript is not enabled. Is it set in your options?

## 起因

项目中导入 `ant-design-vue` 时使用其官方使用的 `css` 预处理器 `less`，但是导入后会直接报错 `Inline JavaScript is not enabled. Is it set in your options?`

![image-20221129150027848](vite%20ant-design-vue%20%E6%8A%A5%E9%94%99%20Inline%20JavaScript%20is%20not%20enabled.%20Is%20it%20set%20in%20your%20options.assets/image-20221129150027848.png)

## 解决

网上查阅方案后发现需要在配置文件中加配置 [共享配置 | Vite 官方中文文档 (vitejs.dev)](https://cn.vitejs.dev/config/shared-options.html#css-preprocessoroptions)

```typescript
// file: vite.config.ts

export default defineConfig({
  ...
  css: {
    preprocessorOptions: {
      less: {
        javascriptEnabled: true,
      },
    }
  }
})

```

