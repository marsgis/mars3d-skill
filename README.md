
## 什么是 mars3d-skill？
mars3d-skill是专门为Mars3D平台使用打造的 AI 辅助技能包，能帮你解决项目开发、集成、调试中的各类问题（比如地图空白、打包失败、API 使用错误等），尤其适配 Vue/ES5 技术栈和Vue通用项目模版(widget 机制)，让 AI 生成的代码更贴合 Mars3D 官方规范和工程最佳实践。

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
在开始使用前，请先确认：
1. 你已经有一个 Mars3D 项目，下面我们已 [mars3d-vue-project项目](https://gitee.com/marsgis/mars3d-vue-project)为例；
2. 已安装 Codex 或 Claude Code（二选一即可，下文会分别讲解）；
3. 网络正常，能下载 GitHub/Gitee 上的代码。

### 下载代码
- [Github](https://github.com/marsgis/mars3d-skill)

```
git clone https://github.com/marsgis/mars3d-skill.git
```

- [Gitee](https://gitee.com/marsgis/mars3d-skill)：国内码云，下载速度快些。

```
git clone https://gitee.com/marsgis/mars3d-skill.git
```

下面2个方式的对比：

|对比项|Codex（OpenAI）|Claude Code（Anthropic）|
|---|---|---|
|**skill 目录**|`\.agents\skills\`|`\.claude\skills\`|
|**调用命令**|`$mars3d-skill`|`/mars3d-skill`|
|开发公司|OpenAI|Anthropic|
|核心模型|GPT/o 系列代码专项版|Claude Sonnet/Opus/Haiku|
|运行特点|云端沙盒、并行多任务、安全严格|本地终端优先、深度项目理解|
|适合人群|全平台、多任务、注重安全|习惯命令行、大型项目、复杂调试|
|token 消耗|相对更低、更精简|内容更详细，消耗偏高|
|生态侧重|ChatGPT+云端工程|终端+本地项目+Skill 扩展|



  

## 方式一：在 Codex 中使用
Codex 是 OpenAI 推出的 AI 编程智能体，前身是 GitHub Copilot 底层代码模型，现在是面向软件工程的全流程自动化助手。
- [Codex skills 介绍](https://developers.openai.com/codex/skills)


### 一. 把 skill 复制到项目指定目录

1. 找到你的 Mars3D 项目根目录（比如 `D:\mars3d-vue-project\`）；

2. 在项目根目录下，依次新建文件夹：`.agents` → `skills`（注意 `.agents` 前面有个点）,完成后目录为`D:\mars3d-vue-project\.agents\skills\`；

3. 把下载好的 `mars3d-skill` 文件夹（里面包含SKILL.md、references等），完整复制到 `\.agents\skills\` 目录下。

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

4. 如果 skill 没出现在选择器里，重启 Codex 即可。
  
![codex使用步骤](http://mars3d.cn/docs/img/guide/skill-codex-start.jpg)


### 三. 根据实际需求输入skill指令

后续更加您实际需求，按需输入skill指令，比如：

```Plain Text
$mars3d-skill 帮我排查项目 npm run build 后地图空白的问题
$mars3d-skill 帮我新增一个 Mars3D 按钮，点击后在地图上标绘点并记录到表
$mars3d-skill 帮我新增一个 Mars3D widget，点击按钮后在地图上绘制点，并把点记录同步到表格
```




## 方式二：在 Claude Code 中使用
Claude Code 是 Anthropic 公司基于 Claude 大模型打造的 AI 编程智能体，主打深度理解项目、终端原生工作流。
- [Claude Code skills 介绍](https://docs.claude.com/en/docs/claude-code/skills)

### 一. 把 skill 复制到项目指定目录
1. 找到你的 Mars3D 项目根目录（比如 `D:\mars3d-vue-project\`）；

2. 在项目根目录下，依次新建文件夹：`.claude` → `skills`（注意 `.claude` 前面有个点）,完成后目录为`D:\mars3d-vue-project\.claude\skills\`；

3. 把下载好的 `mars3d-skill` 文件夹，完整复制到 `.claude/skills` 目录下。

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
- 检查文件夹路径是否正确（比如 Codex 是 `.agents/skills/mars3d-skill`，Claude 是 `.claude/skills/mars3d-skill`）；
- 确保 `mars3d-skill` 文件夹里包含 `SKILL.md` 和 `references` 目录，不要只复制单个文件；
- 重启 Codex/Claude Code，并重进项目目录。


### Q2：使用 skill 会额外收费吗？
- skill 本身免费，AI 平台（Codex/Claude Code）会按 token 消耗计费（比如 Claude Code 用 deepseek-v4-flash 模型，三次测试仅花费 0.22 元，费用受需求难度影响）。

![价格](http://mars3d.cn/docs/img/guide/skill-token-price.jpg)


### Q3：skill 能解决哪些问题？
- 排查 npm 安装、CDN / 静态文件集成问题；
- 解决地图空白、打包失败、API 调用报错；
- 开发 Mars3D Vue widget、标绘点 / 线 / 面、图层管理等功能；
- 规范 Vue 项目中 Mars3D 对象的使用方式（避免响应式坑）。


## 最新教程

最新更详细教程请参考：[官网教程文档](https://mars3d.cn/docs/guide/skill)





## Mars3D 是什么

> `Mars3D平台` 是一款基于 WebGL 技术实现的三维客户端开发平台，基于[Cesium](https://cesium.com/cesiumjs/)优化提升与 B/S 架构设计，支持多行业扩展的轻量级高效能 GIS 开发平台，能够免安装、无插件地在浏览器中高效运行，并可快速接入与使用多种 GIS 数据和三维模型，呈现三维空间的可视化，完成平台在不同行业的灵活应用。

> Mars3D 平台可用于构建无插件、跨操作系统、 跨浏览器的三维 GIS 应用程序。平台使用 WebGL 来进行硬件加速图形化，跨平台、跨浏览器来实现真正的动态大数据三维可视化。通过 Mars3D 产品可快速实现浏览器和移动端上美观、流畅的三维地图呈现与空间分析。

### 相关网站

- Mars3D 官网：[http://mars3d.cn](http://mars3d.cn)

- Mars3D 开源项目列表：[https://github.com/marsgis/mars3d](https://github.com/marsgis/mars3d)

## 版权说明

1. Mars3D 平台由[mars3d团队](http://mars3d.cn/)自主研发，拥有所有权利。
2. 任何个人或组织可以在遵守相关要求下可以免费无限制使用。
