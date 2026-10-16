
## 什么是 mars3d-skill？
mars3d-skill 是面向 Mars3D 的 AI 辅助技能包，可帮助分析和修改项目代码、集成 SDK、排查地图空白或 API 使用问题。它包含 Vue/ES5 工程实践、Vue widget 模式和常见 Mars3D API 代码模式。

### 主要能力
- 分析和修改 Mars3D 项目代码。
- 排查 npm、静态文件、CDN 等 Mars3D 集成问题。
- 处理 `mars3d-cesium`、Cesium 资源路径、地图空白、打包失败等常见问题。
- 根据 `mars3d.d.ts` 和官方 API 页面确认 class、method、option 的正确用法。
- 支持 Mars3D Vue widget 机制经验：`widget-store.ts`、`index.vue`、`map.ts`、`useLifecycle(mapWork)`。
- 记录 Vue 工程实践：页面数据只保存业务状态和 graphic id，不把 Mars3D/Cesium 对象放进深度响应式状态。
- 提供常用代码模式，例如 `GraphicLayer`、`startDraw`、`centerPoint`、layer cleanup 等。

### 主要文件及目录说明
- `./SKILL.md` 是 skill 入口，负责描述触发条件、工作流程和 reference 选择规则。
- `/references/` 中存放按需加载的经验文档。文档采用“中文经验说明 + 英文 API 术语”的形式，保留 `Mars3D`、`GraphicLayer`、`startDraw`、`centerPoint`、`widget-store.ts` 等关键术语，便于和代码、类型声明、官方 API 对齐。

 

## 准备工作
开始使用前，请准备：
1. 已安装支持 Agent Skills 的 Codex 或 Claude Code（下文分别说明）；
2. 一个 Mars3D 项目（可选，下面以 [mars3d-vue-project](https://gitee.com/marsgis/mars3d-vue-project) 为例）。

也可以先安装 skill，再在之后打开 Mars3D 项目使用。

### 下载代码
- [Github](https://github.com/marsgis/mars3d-skill)

```
git clone https://github.com/marsgis/mars3d-skill.git
```

- [Gitee](https://gitee.com/marsgis/mars3d-skill)：国内码云，下载速度快些。

```
git clone https://gitee.com/marsgis/mars3d-skill.git
```

本仓库同时提供 Codex 和 Claude Code 的安装说明。两个工具的配置目录和调用方式不同，具体行为及计费请以各自官方文档和账户设置为准。



  

## 方式一：在 Codex 中使用
Codex 是 OpenAI 提供的编程智能体，支持通过 Agent Skills 扩展工作流。
- [Codex skills 介绍](https://developers.openai.com/codex/skills)


### 一. 把 skill 复制到项目指定目录

1. 找到你的 Mars3D 项目根目录（比如 `D:\mars3d-vue-project\`）；

2. 在项目根目录下创建 `.agents/skills/`（注意 `.agents` 前面有点），例如 `D:\mars3d-vue-project\.agents\skills\`；

3. 将仓库内容复制到 `.agents/skills/mars3d-skill/`，确认 `SKILL.md` 位于该目录根部，`references/` 和 `agents/` 等文件夹也一并复制。

最终你的项目结构应该是这样（对照看，确保没错）：

```Plain Text
D:\mars3d-vue-project\
├─ .agents/
│  └─ skills/
│     └─ mars3d-skill/
│        ├─ SKILL.md
│        ├─ agents/
│        │  └─ openai.yaml
│        └─ references/
│           ├─ api-errors.md
│           ├─ integration.md
│           └─ ...（其他参考文档）
├─ src/
├─ package.json
└─ ...（项目其他文件）
```


### 二. 启动 Codex 并使用 skill
1. 打开命令行工具，切换到你的 Mars3D 项目根目录（比如执行 `cd D:\mars3d-vue-project\`）；
2. 启动 Codex（按 Codex 官方方式启动，比如双击 Codex 客户端，或执行启动命令）；
3. 在 Codex 的输入框中，以 `$mars3d-skill` 开头，输入一个测试需求看看功能是否正常，比如：

    ```Plain Text
    $mars3d-skill 帮我分析这个 Mars3D Vue 项目是否使用 widget 机制
    ```
>Codex的调用名`$mars3d-skill`来自 `SKILL.md` 里的 `name: mars3d-skill`

4. 如果 skill 没有出现，先确认当前打开的是包含 `.agents/skills/mars3d-skill/SKILL.md` 的项目，并检查目录层级和文件名；再按 Codex 当前版本的说明刷新或重新打开项目。
  
![codex使用步骤](http://mars3d.cn/docs/img/guide/skill-codex-start.jpg)


### 三. 根据实际需求输入skill指令

后续更加您实际需求，按需输入skill指令，比如：

```Plain Text
$mars3d-skill 帮我排查项目 npm run build 后地图空白的问题
$mars3d-skill 帮我新增一个 Mars3D 按钮，点击后在地图上标绘点并记录到表
$mars3d-skill 帮我新增一个 Mars3D widget，点击按钮后在地图上绘制点，并把点记录同步到表格
```




## 方式二：在 Claude Code 中使用
Claude Code 是 Anthropic 提供的编程智能体，支持通过项目级 skills 扩展工作流。
- [Claude Code skills 介绍](https://docs.claude.com/en/docs/claude-code/skills)

### 一. 把 skill 复制到项目指定目录
1. 找到你的 Mars3D 项目根目录（比如 `D:\mars3d-vue-project\`）；

2. 在项目根目录下创建 `.claude/skills/`（注意 `.claude` 前面有点），例如 `D:\mars3d-vue-project\.claude\skills\`；

3. 将仓库内容复制到 `.claude/skills/mars3d-skill/`，确认 `SKILL.md` 位于该目录根部，`references/` 等文件夹也一并复制。

最终你的项目结构应该是这样：
```Plain Text
D:\mars3d-vue-project\
├─ .claude/
│  └─ skills/
│     └─ mars3d-skill/
│        ├─ SKILL.md
│        └─ references/
│           ├─ api-errors.md
│           ├─ integration.md
│           └─ ...（其他参考文档）
├─ src/
├─ package.json
└─ ...（项目其他文件）
```




### 二. 启动 Claude Code 并使用 skill
1. 打开 Claude Code，并确保当前工作目录是你的 Mars3D 项目根目录；
2. 在输入框中，以 `/mars3d-skill` 开头输入一个测试需求看看功能是否正常，比如：
    ```Plain Text
    /mars3d-skill 帮我分析这个 Mars3D Vue 项目是否使用 widget 机制
    ```
> Claude Code 的调用名来自skill目录名

3. Claude Code 也会根据你的需求自动加载该 skill，无需手动选择。

![claude使用skill流程](http://mars3d.cn/docs/img/guide/skill-claude-start.jpg)

 
### 三. 根据实际需求输入skill指令

后续更加您实际需求，按需输入skill指令，比如：

```Plain Text
/mars3d-skill 帮我判断项目是否使用 Mars3D Vue widget 机制
/mars3d-skill 帮我写一个按钮，点击后在地图上标绘点，表格记录并支持删除
```
 

 


## 常见问题解答（小白必看）
 
### Q1：复制文件夹后，AI 没识别到 skill 怎么办？
- 检查 skill 是否位于工具支持的目录，且 `SKILL.md` 是否在 `mars3d-skill` 目录根部；
- 确认整个目录都已复制，而不只是 `SKILL.md`；
- 确认工具当前打开的是正确项目，再按该工具当前版本的说明刷新 skills 或重新打开项目。


### Q2：使用 skill 会额外收费吗？
- skill 文件本身不收取费用。模型使用、订阅或 API 费用取决于所用平台、账户方案和模型，请查看对应平台的当前计费说明。


### Q3：skill 能解决哪些问题？
- 排查 npm 安装、CDN / 静态文件集成问题；
- 解决地图空白、打包失败、API 调用报错；
- 开发 Mars3D Vue widget、标绘点 / 线 / 面、图层管理等功能；
- 规范 Vue 项目中 Mars3D 对象的使用方式（避免响应式坑）。


## 最新教程

最新更详细教程请参考：[官网教程文档](http://mars3d.cn/docs/guide/skill)





## Mars3D 是什么

> `Mars3D平台` 是一款基于 WebGL 技术实现的三维客户端开发平台，基于[Cesium](https://cesium.com/cesiumjs/)优化提升与 B/S 架构设计，支持多行业扩展的轻量级高效能 GIS 开发平台，能够免安装、无插件地在浏览器中高效运行，并可快速接入与使用多种 GIS 数据和三维模型，呈现三维空间的可视化，完成平台在不同行业的灵活应用。

> Mars3D 平台可用于构建无插件、跨操作系统、 跨浏览器的三维 GIS 应用程序。平台使用 WebGL 来进行硬件加速图形化，跨平台、跨浏览器来实现真正的动态大数据三维可视化。通过 Mars3D 产品可快速实现浏览器和移动端上美观、流畅的三维地图呈现与空间分析。

### 相关网站

- Mars3D 官网：[http://mars3d.cn](http://mars3d.cn)

- Mars3D 开源项目列表：[https://github.com/marsgis/mars3d](https://github.com/marsgis/mars3d)

## 📄 开源协议 / License

本仓库（**mars3d-skill**）为 AI 辅助技能包，内容以文档与配置为主，基于 **Apache License 2.0** 开源，可自由使用、复制、修改与分发（含商业用途）。

**请注意**：Mars3D SDK 是独立授权的产品，其授权条款与本仓库相互独立。使用 Mars3D SDK 时请遵守其授权条款（社区免费版：Apache-2.0 ＋ 附加条款，须保留页面 Logo 与 F12 控制台版权信息；商业授权版：可去除署名），详见 <http://mars3d.cn>；购买商业授权：<http://mars3d.cn/buy.html>
