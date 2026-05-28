# Integration And Project Setup / 集成与项目搭建

当任务涉及安装、集成、升级或排查 Mars3D 项目时，先使用这份入口文档。它只负责判断集成方式、确认本地版本、选择后续 reference；不要在未确定技术栈前加载所有集成细节。

## Integration Choices / 集成方式

Mars3D 提供三种获取 SDK 类库和集成的方式，根据项目实际情况任选其中一种。

- NPM：适合 Vue、ES5 等 Node 开发框架项目，推荐使用。确认项目使用 webpack、vue-cli 还是 vite 后，加载 `references/integration-npm.md`。
- 静态文件：适合原生项目、npm 安装存在问题时、离线内网环境，所有技术栈都可以使用。入口 HTML 通过 `script` 标签直接引入 `mars3d.js` 时，加载 `references/integration-static.md`。
- CDN：适合临时演示及快速开始使用，不适合生产环境。入口 HTML 使用远程 CDN `script` 时，加载 `references/integration-cdn.md`。

不要在同一项目中混用多套 Mars3D/Cesium 运行时。尤其注意不要同时打包 `cesium` 和 `mars3d-cesium`，除非项目明确通过配置切换到原生 `cesium` 包。

## Local Version Check / 本地版本确认

集成或排查前，先确定当前项目实际使用的 Mars3D 版本和集成方式。

优先检查：

- 最快捷的方式就是浏览器打开项目地址，进入地图页面，接着打开浏览器控制台，控制台中会打印当前使用的版本。
- `package.json`：看 `mars3d`、`mars3d-cesium`、`@turf/turf`、`vite-plugin-mars3d`、`cesium`、Mars3D 插件依赖。
- lockfiles：`package-lock.json`、`pnpm-lock.yaml`、`yarn.lock` 中的精确版本。
- `node_modules/mars3d/package.json` 和 `node_modules/mars3d-cesium/package.json`：确认真实安装版本。
- 入口 HTML：检查是否通过 `script` 引入本地 `lib/mars3d/mars3d.js` 或 CDN URL。
- 构建配置：检查 `webpack.config.js`、`vue.config.js`、`vite.config.ts` 中的 Cesium 资源复制、`externals`、`mars3dPlugin()` 配置。

常用搜索：

```powershell
rg -n "\"mars3d\"|mars3d-cesium|vite-plugin-mars3d|\"cesium\"|mars3dPlugin|CESIUM_BASE_URL|externals" -g "package.json" -g "package-lock.json" -g "pnpm-lock.yaml" -g "yarn.lock" -g "webpack.config.js" -g "vue.config.js" -g "vite.config.*" .
rg -n "mars3d\\.js|mars3d\\.css|Cesium\\.js|registry\\.npmmirror|unpkg|jsdelivr|window\\.mars3d" .
```

## Reference Routing / 引用文件分流

按识别结果只加载一个具体集成文件：

- 发现 `package.json` 安装 `mars3d`，代码里 `import * as mars3d from "mars3d"`：加载 `references/integration-npm.md`。
- 发现 `webpack.config.js`：在 `references/integration-npm.md` 中使用 webpack 配置段。
- 发现 `vue.config.js`：在 `references/integration-npm.md` 中使用 vue-cli 配置段。
- 发现 `vite.config.ts` 或 `vite.config.js`：在 `references/integration-npm.md` 中使用 vite 配置段。
- 发现入口 HTML 引入 `./lib/mars3d/mars3d.js`、`./lib/Cesium/Cesium.js`，代码通过 `window.mars3d` 使用：加载 `references/integration-static.md`。
- 发现入口 HTML 引入 `registry.npmmirror.com`、`unpkg.com`、`cdn.jsdelivr.net` 等远程资源：加载 `references/integration-cdn.md`。
- NPM 方式集成失败并改为静态 Cesium 兜底时：先看 `references/integration-npm.md` 的失败兜底段，再按需看 `references/integration-static.md` 的静态资源目录和 HTML 引入顺序。
