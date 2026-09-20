# OpenWorkBuddy 源代码说明

**文档编号**：SOURCE.md  
**扫描范围**：仓库工作区（不含 `.git` 对象、不含未安装的 `node_modules`）  
**扫描结果摘要**（以生成本文档时为准）：

| 项 | 值 |
|---|---|
| 项目名 | openworkbuddy |
| 版本 | 0.6.8（package.json） |
| 主语言 | JavaScript（Node.js CommonJS，`"type": "commonjs"`） |
| 运行时要求 | Node.js >= 18 |
| 桌面壳 | Electron 43 + electron-builder 26 |
| 模块系统 | CommonJS `require` / `module.exports`；前端为经典 `<script>` 顺序加载，非 ESM 打包 |
| 开发工具 | 任意编辑器；无 webpack/vite；`node --check`；`npm test`；GitHub Actions |
| 辅助语言 | Python（技能内 Office/公众号脚本等）、Bash（安装与打包）、HTML/CSS |
| 工作区内文件总数 | 355 |
| JavaScript 文件数 | 172 |
| Python 文件数 | 18 |
| Markdown 文件数 | 68 |
| JS 总行数 | 117605 |
| Python 总行数 | 4429 |
| HTML 总行数 | 3513 |
| CSS 总行数 | 1217 |
| 代码类行数（js+py+html+css+sh） | 127302 |
| 扫描到的 JS 函数声明（function/箭头，含测试与前端） | 2731 |
| 名为 test* 的测试函数 | 151 |
| 顶层运行时 JS（仓库根 *.js） | 约 97 |
| 测试套件文件 | 44（test/*.js） |
| 内置技能目录 | 35 |

行数为物理行，含空行与注释。技能内大量 OOXML XSD 未全部按「业务代码」计，但计入工作区文件总数。

---

## 1. 源代码整体介绍

### 1.1 这是一份怎样的代码

OpenWorkBuddy 的源码性格是：**可读的 Node 脚本集合，而不是框架生成物**。业务入口是少数几个文件（`server.js`、`agent.js`、`tools.js`、`cli.js`、`electron-main.js`），其余文件几乎都是单一职责模块，文件头用中文注释讲「为什么存在、踩过什么坑」。前端不引入 React，Agent 循环不引入 LangGraph，持久化不引入 ORM。这使得「从 clone 到改提示词再刷新」可以在几分钟内完成，也使得本说明可以把每一个 `.js` 文件当成稳定的教学单元。

### 1.2 语言与运行时细节

- **JavaScript**：业务代码避免依赖实验性语法到无法在 Node 18 运行的程度。大量 `async/await`。错误对用户是中文句子。  
- **严格模式**：多数文件 `'use strict'`。  
- **依赖**：见 package.json。运行时包括 express、各办公生成库、mermaid、echarts、lark sdk、anthropic sdk、qrcode、nodemailer、joint/dagre 等。Electron 与 electron-builder 在 devDependencies，桌面打包才需要。  
- **Python**：docx 技能下有完整 Office 处理脚本与 XSD；公众号技能有 format/publish。不是主运行时。  
- **Shell**：`install.sh`、`install-mac.sh`、`deploy.sh`、`scripts/*.sh`。  
- **HTML/CSS**：`public/` 无构建。

### 1.3 开发工具与命令

| 命令 | 作用 |
|---|---|
| npm start | `node server.js` 浏览器模式 |
| npm run app | 桌面 Electron |
| npm test | `node test/all.js` 全套件 |
| npm run test:e2e | 只跑最大套件 |
| npm run eval | 黑盒评测 |
| npm run stats | 刷新 docs/stats.json |
| npm run dist:mac / dist:win | 打安装包 |
| node cli.js / npm run cli | CLI |
| node cli.js doctor | 体检 |

CI：`.github/workflows/test.yml` 与 `release.yml`。

### 1.4 目录地图

```
/（仓库根）
  *.js                 运行时模块（Agent、HTTP、IM、安全、记忆…）
  engines/             外部执行引擎适配
  public/              前端
  skills/              内置技能 Markdown 与脚本
  test/                测试
  eval/                评测题与运行器
  docs/                产品文档
  deploy/              反代与环境示例
  scripts/             维护与演示
  build/               图标
  .github/workflows    CI
```

用户数据**不在**仓库里，在 `~/OpenWorkBuddy`。

### 1.5 依赖图（概念）

electron-main → server.js → account/security/org  
server.js → agent.js → tools.js / llm.js / skills.js / memory.js / engines  
tools.js → security / diagram / media-models / gen-cache / preview / cdp …  
cli.js → 同一套 agent 与 sessions，经 cli-live 与 UI 汇合  
im.js → agent  
scheduler.js → agent  

禁止出现的环：llmFactory 可注入以避免 agent↔llm 硬环。测试利用这一点。

### 1.6 如何阅读源码（推荐路径）

1. `store.js`（100 行级）理解落盘哲学。  
2. `modes.js` + `rbac.js` 理解两套「档位」。  
3. `agent.js` 的 `createAgentRuntime` 与步循环。  
4. `tools.js` 的 `TOOL_DEFS` 与 `executeTool`。  
5. `server.js` 的 `/api/chat` 与会话落盘。  
6. `public/js/app-02.js` 看直播事件如何画成过程卡。  
7. `test/e2e.js` 某一条负对照，学习如何钉行为。

### 1.7 编码约定

- 用户可见字符串中文；标识符英文。  
- 密钥打码；日志不写明文 Key。  
- 路径 `insideRoot`。  
- JSON 人读的 pretty，流水账压缩。  
- 测试隔离环境变量与数据目录。  
- 改提示词必跑 eval 的规则写在 CONTRIBUTING。

### 1.8 许可证与第三方

根协议 PolyForm Noncommercial。生态目录 MIT 见 LICENSE-ECOSYSTEM.md。NOTICE.md 列出借鉴。第三方商标不授予。JointJS MPL-2.0。

### 1.9 规模解读

约 11 万行 JS 对「无框架 Agent 工作台」偏多，原因是：办公文件、短剧画布、IM、多租户、评测、自进化都在同一仓库，而不是拆成微服务。`server.js` 与 `tools.js`、`agent.js`、`test/e2e.js` 是最大文件，阅读时按函数检索，不要指望一页看完。

## 2. 源代码文件清单

下列每一条包含：路径、行数、字节、主要功能。功能简介结合文件头注释与模块职责。

### 2.1 `.dockerignore`

- **所在目录**：`.`
- **文件名**：`.dockerignore`
- **类型**：`无扩展名`
- **行数**：0
- **大小**：1779 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check .dockerignore`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.2 `.github/workflows/release.yml`

- **所在目录**：`.github/workflows`
- **文件名**：`release.yml`
- **类型**：`.yml`
- **行数**：169
- **大小**：9691 字节
- **主要功能**：CI：测试与发版工作流。
- **维护建议**：改此文件前先跑 `node --check .github/workflows/release.yml`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.3 `.github/workflows/test.yml`

- **所在目录**：`.github/workflows`
- **文件名**：`test.yml`
- **类型**：`.yml`
- **行数**：53
- **大小**：3063 字节
- **主要功能**：CI：测试与发版工作流。
- **维护建议**：改此文件前先跑 `node --check .github/workflows/test.yml`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.4 `.gitignore`

- **所在目录**：`.`
- **文件名**：`.gitignore`
- **类型**：`无扩展名`
- **行数**：0
- **大小**：3291 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check .gitignore`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.5 `CHANGELOG.en.md`

- **所在目录**：`.`
- **文件名**：`CHANGELOG.en.md`
- **类型**：`.md`
- **行数**：169
- **大小**：99980 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check CHANGELOG.en.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.6 `CHANGELOG.md`

- **所在目录**：`.`
- **文件名**：`CHANGELOG.md`
- **类型**：`.md`
- **行数**：167
- **大小**：89971 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check CHANGELOG.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.7 `COMMERCIAL-LICENSE.md`

- **所在目录**：`.`
- **文件名**：`COMMERCIAL-LICENSE.md`
- **类型**：`.md`
- **行数**：106
- **大小**：5789 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check COMMERCIAL-LICENSE.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.8 `CONTRIBUTING.md`

- **所在目录**：`.`
- **文件名**：`CONTRIBUTING.md`
- **类型**：`.md`
- **行数**：204
- **大小**：12676 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check CONTRIBUTING.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.9 `Dockerfile`

- **所在目录**：`.`
- **文件名**：`Dockerfile`
- **类型**：`无扩展名`
- **行数**：53
- **大小**：2771 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check Dockerfile`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.10 `LICENSE`

- **所在目录**：`.`
- **文件名**：`LICENSE`
- **类型**：`无扩展名`
- **行数**：0
- **大小**：5514 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check LICENSE`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.11 `LICENSE-ECOSYSTEM.md`

- **所在目录**：`.`
- **文件名**：`LICENSE-ECOSYSTEM.md`
- **类型**：`.md`
- **行数**：108
- **大小**：5624 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check LICENSE-ECOSYSTEM.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.12 `NOTICE.md`

- **所在目录**：`.`
- **文件名**：`NOTICE.md`
- **类型**：`.md`
- **行数**：62
- **大小**：4471 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check NOTICE.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.13 `README.en.md`

- **所在目录**：`.`
- **文件名**：`README.en.md`
- **类型**：`.md`
- **行数**：357
- **大小**：25717 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check README.en.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.14 `README.md`

- **所在目录**：`.`
- **文件名**：`README.md`
- **类型**：`.md`
- **行数**：366
- **大小**：24546 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check README.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.15 `account.js`

- **所在目录**：`.`
- **文件名**：`account.js`
- **类型**：`.js`
- **行数**：1959
- **大小**：103465 字节
- **主要功能**：账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。 导出或内部函数包括 `ttlMsFor`、`readStore`、`writeStoreAtomic`、`loadUsers`、`openRegister`、`creditsEnabled`、`saveUsers`、`loadUsage`、`saveUsage`、`localDay`、`hashPassword`、`passwordProblem` 等共 91 个函数。
- **维护建议**：改此文件前先跑 `node --check account.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.16 `admin.js`

- **所在目录**：`.`
- **文件名**：`admin.js`
- **类型**：`.js`
- **行数**：815
- **大小**：48441 字节
- **主要功能**：企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。 导出或内部函数包括 `platformAdmin`、`platformOnly`、`platformOwnerOnly`、`guarded`、`setDeployment`、`isSoloDesktop`、`ownsGlobalWorkspace`、`platformGuard`、`redactSecrets`、`redactGuard`、`tenantScope`、`createAdminRouter` 等共 13 个函数。
- **维护建议**：改此文件前先跑 `node --check admin.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.17 `agent.js`

- **所在目录**：`.`
- **文件名**：`agent.js`
- **类型**：`.js`
- **行数**：2816
- **大小**：200386 字节
- **主要功能**：Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果核验与死循环硬停。 导出或内部函数包括 `workspaceIndex`、`sizeOf`、`missingDeliverables`、`unseenVisualClaims`、`unfinishedMilestones`、`entryChars`、`historyChars`、`dropRendererParams`、`hasRenderer`、`trimHistory`、`envToday`、`safeWorkspaceDir` 等共 30 个函数。
- **维护建议**：改此文件前先跑 `node --check agent.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.18 `awake.js`

- **所在目录**：`.`
- **文件名**：`awake.js`
- **类型**：`.js`
- **行数**：129
- **大小**：5593 字节
- **主要功能**：睡眠检测与 keep-awake，合盖顺延时限。 导出或内部函数包括 `suspendedFromTick`、`startTicker`、`watch`、`totalSuspendedMs`、`acquireAssertion`、`releaseAssertion`、`hold`。
- **维护建议**：改此文件前先跑 `node --check awake.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.19 `boot-check.js`

- **所在目录**：`.`
- **文件名**：`boot-check.js`
- **类型**：`.js`
- **行数**：105
- **大小**：5014 字节
- **主要功能**：启动自检。 导出或内部函数包括 `bootProblem`、`findMissing`、`enforce`。
- **维护建议**：改此文件前先跑 `node --check boot-check.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.20 `browser-render.js`

- **所在目录**：`.`
- **文件名**：`browser-render.js`
- **类型**：`.js`
- **行数**：138
- **大小**：6255 字节
- **主要功能**：隐藏窗口渲染动态页。 导出或内部函数包括 `available`、`withHiddenWindow`、`loadHtml`、`renderMermaid`、`svgSize`、`svgToPng`、`delay`。
- **维护建议**：改此文件前先跑 `node --check browser-render.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.21 `budget.js`

- **所在目录**：`.`
- **文件名**：`budget.js`
- **类型**：`.js`
- **行数**：328
- **大小**：17004 字节
- **主要功能**：额度预扣与释放，防止并发超卖。 导出或内部函数包括 `monthKey`、`spentOf`、`round6`、`limitsOf`、`estimate`、`reserve`、`settle`、`release`、`sweep`、`status`、`exhausted`、`record` 等共 13 个函数。
- **维护建议**：改此文件前先跑 `node --check budget.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.22 `build/icon.icns`

- **所在目录**：`build`
- **文件名**：`icon.icns`
- **类型**：`.icns`
- **行数**：0
- **大小**：687348 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check build/icon.icns`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.23 `build/icon.ico`

- **所在目录**：`build`
- **文件名**：`icon.ico`
- **类型**：`.ico`
- **行数**：0
- **大小**：372526 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check build/icon.ico`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.24 `build/icon.png`

- **所在目录**：`build`
- **文件名**：`icon.png`
- **类型**：`.png`
- **行数**：0
- **大小**：446974 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check build/icon.png`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.25 `build/icon.svg`

- **所在目录**：`build`
- **文件名**：`icon.svg`
- **类型**：`.svg`
- **行数**：0
- **大小**：7575 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check build/icon.svg`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.26 `callout.js`

- **所在目录**：`.`
- **文件名**：`callout.js`
- **类型**：`.js`
- **行数**：25
- **大小**：1006 字节
- **主要功能**：正文提示条：网页画图标，终端/IM 换文字。 导出或内部函数包括 `line`、`strip`。
- **维护建议**：改此文件前先跑 `node --check callout.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.27 `cdp.js`

- **所在目录**：`.`
- **文件名**：`cdp.js`
- **类型**：`.js`
- **行数**：353
- **大小**：22771 字节
- **主要功能**：Chrome DevTools Protocol 真浏览器控制。 导出或内部函数包括 `endpointHost`、`getJson`、`idleMs`、`clearIdle`、`touchIdle`、`killTree`、`close`、`hookExit`、`findChrome`、`probe`、`readPortFile`、`profileDir` 等共 22 个函数。
- **维护建议**：改此文件前先跑 `node --check cdp.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.28 `chat-models.js`

- **所在目录**：`.`
- **文件名**：`chat-models.js`
- **类型**：`.js`
- **行数**：286
- **大小**：16367 字节
- **主要功能**：对话渠道表、选型、健康账本。 导出或内部函数包括 `chanKeyOf`、`nameForKind`、`wantsChannel`、`envKeyFor`、`pruneSeededPresets`、`templates`、`planTemplate`、`commitTemplate`、`legacyRows`、`normalize`、`modelsOf`、`isLocalBase` 等共 14 个函数。
- **维护建议**：改此文件前先跑 `node --check chat-models.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.29 `checkpoints.js`

- **所在目录**：`.`
- **文件名**：`checkpoints.js`
- **类型**：`.js`
- **行数**：320
- **大小**：14450 字节
- **主要功能**：文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。 导出或内部函数包括 `insideRoot`、`relOf`、`putObject`、`readObject`、`readLedger`、`appendLedger`、`record`、`fileHash`、`list`、`rewind`、`gc`、`splitLines` 等共 18 个函数。
- **维护建议**：改此文件前先跑 `node --check checkpoints.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.30 `cli-approve.js`

- **所在目录**：`.`
- **文件名**：`cli-approve.js`
- **类型**：`.js`
- **行数**：141
- **大小**：6833 字节
- **主要功能**：终端/手机审批危险操作。 导出或内部函数包括 `render`、`hint`、`parse`、`run`、`waitText`、`card`。
- **维护建议**：改此文件前先跑 `node --check cli-approve.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.31 `cli-args.js`

- **所在目录**：`.`
- **文件名**：`cli-args.js`
- **类型**：`.js`
- **行数**：403
- **大小**：21962 字节
- **主要功能**：CLI 参数表。 导出或内部函数包括 `editDistance`、`nearestFlag`、`nearestSub`、`looksLikeProse`、`problem`、`usableValue`、`parse`、`helpText`、`completionScript`、`problemText`、`sh`、`z`。
- **维护建议**：改此文件前先跑 `node --check cli-args.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.32 `cli-ask.js`

- **所在目录**：`.`
- **文件名**：`cli-ask.js`
- **类型**：`.js`
- **行数**：186
- **大小**：8832 字节
- **主要功能**：终端回答 agent 提问。 导出或内部函数包括 `wrap`、`render`、`hint`、`normalize`、`parse`、`retryText`、`run`、`waitText`。
- **维护建议**：改此文件前先跑 `node --check cli-ask.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.33 `cli-attach.js`

- **所在目录**：`.`
- **文件名**：`cli-attach.js`
- **类型**：`.js`
- **行数**：414
- **大小**：19307 字节
- **主要功能**：CLI 带文件和图片。 导出或内部函数包括 `tokenize`、`fromFileUrl`、`pathLike`、`expandHome`、`escPath`、`parseLine`、`atToken`、`isInside`、`cleanName`、`freeName`、`stampName`、`collect` 等共 20 个函数。
- **维护建议**：改此文件前先跑 `node --check cli-attach.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.34 `cli-live.js`

- **所在目录**：`.`
- **文件名**：`cli-live.js`
- **类型**：`.js`
- **行数**：399
- **大小**：17474 字节
- **主要功能**：终端与网页直播桥。 导出或内部函数包括 `dir`、`ensureDir`、`fileOf`、`pidAlive`、`readMeta`、`isLive`、`list`、`rowOf`、`get`、`drop`、`read`、`readLines` 等共 17 个函数。
- **维护建议**：改此文件前先跑 `node --check cli-live.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.35 `cli.js`

- **所在目录**：`.`
- **文件名**：`cli.js`
- **类型**：`.js`
- **行数**：2120
- **大小**：122034 字节
- **主要功能**：openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair、worktree 等子命令。 导出或内部函数包括 `paintDiff`、`goalThink`、`printGoalCard`、`listCliSessions`、`newSessionId`、`saveSess`、`makeEmit`、`printSummary`、`contextLine`、`noteChanged`、`drawOutputs`、`hintOutputs` 等共 40 个函数。
- **维护建议**：改此文件前先跑 `node --check cli.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.36 `config-lint.js`

- **所在目录**：`.`
- **文件名**：`config-lint.js`
- **类型**：`.js`
- **行数**：127
- **大小**：6858 字节
- **主要功能**：配置体检：Key 形态、地址、互斥字段。 导出或内部函数包括 `distance`、`nearest`、`lint`、`lines`、`KIND`。
- **维护建议**：改此文件前先跑 `node --check config-lint.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.37 `config-merge.js`

- **所在目录**：`.`
- **文件名**：`config-merge.js`
- **类型**：`.js`
- **行数**：89
- **大小**：3456 字节
- **主要功能**：配置深合并与热加载。 导出或内部函数包括 `mtimeOf`、`snapshot`、`changedPaths`、`applyAt`、`mergeOnto`、`isPlain`。
- **维护建议**：改此文件前先跑 `node --check config-merge.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.38 `config.example.json`

- **所在目录**：`.`
- **文件名**：`config.example.json`
- **类型**：`.json`
- **行数**：87
- **大小**：3866 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check config.example.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.39 `deploy.sh`

- **所在目录**：`.`
- **文件名**：`deploy.sh`
- **类型**：`.sh`
- **行数**：187
- **大小**：8039 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check deploy.sh`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.40 `deploy/Caddyfile`

- **所在目录**：`deploy`
- **文件名**：`Caddyfile`
- **类型**：`无扩展名`
- **行数**：21
- **大小**：954 字节
- **主要功能**：部署配置：反向代理、环境变量示例、运维说明。
- **维护建议**：改此文件前先跑 `node --check deploy/Caddyfile`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.41 `deploy/README.md`

- **所在目录**：`deploy`
- **文件名**：`README.md`
- **类型**：`.md`
- **行数**：373
- **大小**：16021 字节
- **主要功能**：部署配置：反向代理、环境变量示例、运维说明。
- **维护建议**：改此文件前先跑 `node --check deploy/README.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.42 `deploy/env.example`

- **所在目录**：`deploy`
- **文件名**：`env.example`
- **类型**：`.example`
- **行数**：0
- **大小**：1602 字节
- **主要功能**：部署配置：反向代理、环境变量示例、运维说明。
- **维护建议**：改此文件前先跑 `node --check deploy/env.example`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.43 `deploy/nginx.conf`

- **所在目录**：`deploy`
- **文件名**：`nginx.conf`
- **类型**：`.conf`
- **行数**：0
- **大小**：7022 字节
- **主要功能**：部署配置：反向代理、环境变量示例、运维说明。
- **维护建议**：改此文件前先跑 `node --check deploy/nginx.conf`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.44 `diagram.js`

- **所在目录**：`.`
- **文件名**：`diagram.js`
- **类型**：`.js`
- **行数**：362
- **大小**：18562 字节
- **主要功能**：四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。 导出或内部函数包括 `diagramCfg`、`plantumlEncode`、`renderECharts`、`renderDot`、`renderPlantuml`、`krokiRender`、`svgToPngAnyhow`、`looksLikeSvg`、`fixLabelBrackets`、`fixSubgraph`、`fixTimelineColon`、`fixGitBranch` 等共 15 个函数。
- **维护建议**：改此文件前先跑 `node --check diagram.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.45 `docker-compose.yml`

- **所在目录**：`.`
- **文件名**：`docker-compose.yml`
- **类型**：`.yml`
- **行数**：64
- **大小**：2904 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check docker-compose.yml`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.46 `docs/Agent效率借鉴.md`

- **所在目录**：`docs`
- **文件名**：`Agent效率借鉴.md`
- **类型**：`.md`
- **行数**：37
- **大小**：3215 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/Agent效率借鉴.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.47 `docs/IM与定时任务.md`

- **所在目录**：`docs`
- **文件名**：`IM与定时任务.md`
- **类型**：`.md`
- **行数**：63
- **大小**：5114 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/IM与定时任务.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.48 `docs/images/case-canvas.jpg`

- **所在目录**：`docs/images`
- **文件名**：`case-canvas.jpg`
- **类型**：`.jpg`
- **行数**：0
- **大小**：389352 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/case-canvas.jpg`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.49 `docs/images/case-hunan-site.jpg`

- **所在目录**：`docs/images`
- **文件名**：`case-hunan-site.jpg`
- **类型**：`.jpg`
- **行数**：0
- **大小**：272225 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/case-hunan-site.jpg`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.50 `docs/images/case-photoreal.jpg`

- **所在目录**：`docs/images`
- **文件名**：`case-photoreal.jpg`
- **类型**：`.jpg`
- **行数**：0
- **大小**：510052 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/case-photoreal.jpg`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.51 `docs/images/case-schedule-feishu.jpg`

- **所在目录**：`docs/images`
- **文件名**：`case-schedule-feishu.jpg`
- **类型**：`.jpg`
- **行数**：0
- **大小**：124477 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/case-schedule-feishu.jpg`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.52 `docs/images/demo-canvas.gif`

- **所在目录**：`docs/images`
- **文件名**：`demo-canvas.gif`
- **类型**：`.gif`
- **行数**：0
- **大小**：172390 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/demo-canvas.gif`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.53 `docs/images/demo.gif`

- **所在目录**：`docs/images`
- **文件名**：`demo.gif`
- **类型**：`.gif`
- **行数**：0
- **大小**：344664 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/demo.gif`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.54 `docs/images/feishu-group.png`

- **所在目录**：`docs/images`
- **文件名**：`feishu-group.png`
- **类型**：`.png`
- **行数**：0
- **大小**：168035 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/feishu-group.png`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.55 `docs/images/holo-card-ui.png`

- **所在目录**：`docs/images`
- **文件名**：`holo-card-ui.png`
- **类型**：`.png`
- **行数**：0
- **大小**：196192 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/holo-card-ui.png`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.56 `docs/images/how-it-works.en.svg`

- **所在目录**：`docs/images`
- **文件名**：`how-it-works.en.svg`
- **类型**：`.svg`
- **行数**：0
- **大小**：4112 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/how-it-works.en.svg`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.57 `docs/images/how-it-works.svg`

- **所在目录**：`docs/images`
- **文件名**：`how-it-works.svg`
- **类型**：`.svg`
- **行数**：0
- **大小**：4144 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/how-it-works.svg`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.58 `docs/images/local-claude-code.png`

- **所在目录**：`docs/images`
- **文件名**：`local-claude-code.png`
- **类型**：`.png`
- **行数**：0
- **大小**：404971 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/local-claude-code.png`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.59 `docs/images/openworkbuddy-overview.svg`

- **所在目录**：`docs/images`
- **文件名**：`openworkbuddy-overview.svg`
- **类型**：`.svg`
- **行数**：0
- **大小**：3470 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/openworkbuddy-overview.svg`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.60 `docs/images/short-drama-canvas-overview.png`

- **所在目录**：`docs/images`
- **文件名**：`short-drama-canvas-overview.png`
- **类型**：`.png`
- **行数**：0
- **大小**：1368022 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/short-drama-canvas-overview.png`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.61 `docs/images/star-guide.svg`

- **所在目录**：`docs/images`
- **文件名**：`star-guide.svg`
- **类型**：`.svg`
- **行数**：0
- **大小**：4663 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/images/star-guide.svg`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.62 `docs/index.html`

- **所在目录**：`docs`
- **文件名**：`index.html`
- **类型**：`.html`
- **行数**：54
- **大小**：18401 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/index.html`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.63 `docs/stats.json`

- **所在目录**：`docs`
- **文件名**：`stats.json`
- **类型**：`.json`
- **行数**：10
- **大小**：247 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/stats.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.64 `docs/功能清单.md`

- **所在目录**：`docs`
- **文件名**：`功能清单.md`
- **类型**：`.md`
- **行数**：81
- **大小**：22200 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/功能清单.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.65 `docs/发版.md`

- **所在目录**：`docs`
- **文件名**：`发版.md`
- **类型**：`.md`
- **行数**：82
- **大小**：4360 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/发版.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.66 `docs/命令行用法.md`

- **所在目录**：`docs`
- **文件名**：`命令行用法.md`
- **类型**：`.md`
- **行数**：408
- **大小**：30132 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/命令行用法.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.67 `docs/多人协作.md`

- **所在目录**：`docs`
- **文件名**：`多人协作.md`
- **类型**：`.md`
- **行数**：150
- **大小**：15023 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/多人协作.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.68 `docs/安全.md`

- **所在目录**：`docs`
- **文件名**：`安全.md`
- **类型**：`.md`
- **行数**：79
- **大小**：8825 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/安全.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.69 `docs/安全基线.md`

- **所在目录**：`docs`
- **文件名**：`安全基线.md`
- **类型**：`.md`
- **行数**：205
- **大小**：16547 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/安全基线.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.70 `docs/安装与启动.md`

- **所在目录**：`docs`
- **文件名**：`安装与启动.md`
- **类型**：`.md`
- **行数**：170
- **大小**：13285 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/安装与启动.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.71 `docs/实现细节.md`

- **所在目录**：`docs`
- **文件名**：`实现细节.md`
- **类型**：`.md`
- **行数**：80
- **大小**：7976 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/实现细节.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.72 `docs/对标学习与服务器运行.md`

- **所在目录**：`docs`
- **文件名**：`对标学习与服务器运行.md`
- **类型**：`.md`
- **行数**：74
- **大小**：4517 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/对标学习与服务器运行.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.73 `docs/开源与商业版边界.md`

- **所在目录**：`docs`
- **文件名**：`开源与商业版边界.md`
- **类型**：`.md`
- **行数**：327
- **大小**：21108 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/开源与商业版边界.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.74 `docs/扩展.md`

- **所在目录**：`docs`
- **文件名**：`扩展.md`
- **类型**：`.md`
- **行数**：163
- **大小**：10126 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/扩展.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.75 `docs/推广/发布日文案.md`

- **所在目录**：`docs/推广`
- **文件名**：`发布日文案.md`
- **类型**：`.md`
- **行数**：249
- **大小**：12747 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/推广/发布日文案.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.76 `docs/数据同步与搬家.md`

- **所在目录**：`docs`
- **文件名**：`数据同步与搬家.md`
- **类型**：`.md`
- **行数**：181
- **大小**：10583 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/数据同步与搬家.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.77 `docs/架构优化路线_2026-08.md`

- **所在目录**：`docs`
- **文件名**：`架构优化路线_2026-08.md`
- **类型**：`.md`
- **行数**：402
- **大小**：51668 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/架构优化路线_2026-08.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.78 `docs/案例.md`

- **所在目录**：`docs`
- **文件名**：`案例.md`
- **类型**：`.md`
- **行数**：33
- **大小**：2290 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/案例.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.79 `docs/模型与Key管理_调研与改法.md`

- **所在目录**：`docs`
- **文件名**：`模型与Key管理_调研与改法.md`
- **类型**：`.md`
- **行数**：240
- **大小**：11619 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/模型与Key管理_调研与改法.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.80 `docs/短剧画布选型.md`

- **所在目录**：`docs`
- **文件名**：`短剧画布选型.md`
- **类型**：`.md`
- **行数**：113
- **大小**：8689 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/短剧画布选型.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.81 `docs/评测方法论.md`

- **所在目录**：`docs`
- **文件名**：`评测方法论.md`
- **类型**：`.md`
- **行数**：101
- **大小**：7004 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/评测方法论.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.82 `docs/路线图.md`

- **所在目录**：`docs`
- **文件名**：`路线图.md`
- **类型**：`.md`
- **行数**：95
- **大小**：7273 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/路线图.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.83 `docs/远程访问.md`

- **所在目录**：`docs`
- **文件名**：`远程访问.md`
- **类型**：`.md`
- **行数**：191
- **大小**：10939 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/远程访问.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.84 `docs/部署.md`

- **所在目录**：`docs`
- **文件名**：`部署.md`
- **类型**：`.md`
- **行数**：39
- **大小**：1786 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/部署.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.85 `docs/配置模型.md`

- **所在目录**：`docs`
- **文件名**：`配置模型.md`
- **类型**：`.md`
- **行数**：112
- **大小**：6623 字节
- **主要功能**：产品与架构文档，给人读，不参与运行时。
- **维护建议**：改此文件前先跑 `node --check docs/配置模型.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.86 `doctor.js`

- **所在目录**：`.`
- **文件名**：`doctor.js`
- **类型**：`.js`
- **行数**：463
- **大小**：23852 字节
- **主要功能**：openworkbuddy doctor 体检。 导出或内部函数包括 `item`、`verdictNode`、`verdictDeps`、`verdictDataDir`、`verdictConfig`、`verdictConfigLint`、`lintConfig`、`verdictModels`、`verdictPort`、`verdictWorkspace`、`verdictEngine`、`verdictTools` 等共 23 个函数。
- **维护建议**：改此文件前先跑 `node --check doctor.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.87 `drama-compose.js`

- **所在目录**：`.`
- **文件名**：`drama-compose.js`
- **类型**：`.js`
- **行数**：461
- **大小**：30160 字节
- **主要功能**：短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。 导出或内部函数包括 `payloadOf`、`baseOf`、`isBlank`、`round`、`safeName`、`orderKeyOf`、`sortShots`、`pickFile`、`freeName`、`srtTime`、`buildSrt`、`musicPick` 等共 14 个函数。
- **维护建议**：改此文件前先跑 `node --check drama-compose.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.88 `drama-pipeline.js`

- **所在目录**：`.`
- **文件名**：`drama-pipeline.js`
- **类型**：`.js`
- **行数**：208
- **大小**：12763 字节
- **主要功能**：短剧进度与产出路径盘点。 文件头摘录：短剧制片进度。 画布上早就有剧本、角色、场次、镜头、生图生视频这些节点了，缺的是「这部戏做到哪了」—— 一张摆了三十个镜头的画布，光靠眼睛看不出还差几张首帧、哪一镜卡着生不出来、 剩下的活儿大概要跑多久。用户只能一个节点一个节点点开看，那不叫工作流，那叫一堆卡片。 这里把画布算成一条**有次序的产线**：剧本 → 定妆 → 分镜 → 首帧 → 镜头视频 → 配音 → 成片。 每一档都给「几个做完了 / 一共几个」，并且回答三个问题：   ① 下一步该干什么（next）；   ② 现在卡在哪、卡的是哪几个节点（blockers，带 ids，界面上能直接跳过去）；   ③ 还剩多少活儿、按这张画布自己跑过的速度大概要多久（pending / eta）。 三条硬规矩：   · 「有路径」不等于「做完了」。盘上没有那个文件，这一格就是没done——     以前那种「字段里写着 first_fra 导出或内部函数包括 `base`、`isPlaceholder`、`payloadOf`、`kindOf`、`outputOf`、`outputPaths`、`medianMs`、`collectRuns`、`dramaProgress`。
- **维护建议**：改此文件前先跑 `node --check drama-pipeline.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.89 `electron-builder.config.js`

- **所在目录**：`.`
- **文件名**：`electron-builder.config.js`
- **类型**：`.js`
- **行数**：201
- **大小**：11289 字节
- **主要功能**：项目支撑文件。 导出或内部函数包括 `skillPatterns`、`findAppDir`、`afterPack`、`adhocSign`。
- **维护建议**：改此文件前先跑 `node --check electron-builder.config.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.90 `electron-main.js`

- **所在目录**：`.`
- **文件名**：`electron-main.js`
- **类型**：`.js`
- **行数**：486
- **大小**：27766 字节
- **主要功能**：Electron 主进程：起本地服务、开窗口、系统托盘、桌面宠物、睡眠阻断、隐藏窗口渲染。 导出或内部函数包括 `pickBootLog`、`bootLog`、`fatal`、`waitForServer`、`attachContextMenu`、`bootHint`、`showBootFailure`、`registerShortcuts`。
- **维护建议**：改此文件前先跑 `node --check electron-main.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.91 `engines/bridge.js`

- **所在目录**：`engines`
- **文件名**：`bridge.js`
- **类型**：`.js`
- **行数**：159
- **大小**：8101 字节
- **主要功能**：把本项目工具借给外部 CLI（MCP）。 导出或内部函数包括 `nodeLauncher`、`buildServers`、`writeMcpConfig`、`codexArgs`、`writeShim`、`attach`。
- **维护建议**：改此文件前先跑 `node --check engines/bridge.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.92 `engines/claude-code.js`

- **所在目录**：`engines`
- **文件名**：`claude-code.js`
- **类型**：`.js`
- **行数**：278
- **大小**：15853 字节
- **主要功能**：本机 Claude Code 引擎适配。 导出或内部函数包括 `purposeOf`、`textOfToolResult`、`explain`、`probeThinking`、`probeAddDir`、`pickAddDirs`、`detect`、`run`。
- **维护建议**：改此文件前先跑 `node --check engines/claude-code.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.93 `engines/codex.js`

- **所在目录**：`engines`
- **文件名**：`codex.js`
- **类型**：`.js`
- **行数**：254
- **大小**：13150 字节
- **主要功能**：本机 Codex 引擎适配。 导出或内部函数包括 `shorten`、`sourceCodexHome`、`configuredModels`、`openWorkBuddyCodexHome`、`toolOf`、`explain`、`detect`、`run`。
- **维护建议**：改此文件前先跑 `node --check engines/codex.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.94 `engines/index.js`

- **所在目录**：`engines`
- **文件名**：`index.js`
- **类型**：`.js`
- **行数**：196
- **大小**：9063 字节
- **主要功能**：执行引擎选择：内置循环 / Claude Code / Codex。 导出或内部函数包括 `list`、`get`、`detectAll`、`detectAllUncached`、`resolve`、`testConnect`、`ask`。
- **维护建议**：改此文件前先跑 `node --check engines/index.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.95 `engines/jsonl.js`

- **所在目录**：`engines`
- **文件名**：`jsonl.js`
- **类型**：`.js`
- **行数**：201
- **大小**：9312 字节
- **主要功能**：引擎 JSONL 事件流解析。 导出或内部函数包括 `runJsonl`、`probeVersion`、`probeOption`、`probeHelp`、`firstVersionLine`。
- **维护建议**：改此文件前先跑 `node --check engines/jsonl.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.96 `engines/tool-bridge.js`

- **所在目录**：`engines`
- **文件名**：`tool-bridge.js`
- **类型**：`.js`
- **行数**：274
- **大小**：11766 字节
- **主要功能**：外部引擎工具桥细节。 导出或内部函数包括 `loadConfig`、`lentDefs`、`listTools`、`callTool`、`send`、`exitIfIdle`、`log`、`handle`、`main`、`readArgs`、`cliList`、`cli` 等共 13 个函数。
- **维护建议**：改此文件前先跑 `node --check engines/tool-bridge.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.97 `engines/which.js`

- **所在目录**：`engines`
- **文件名**：`which.js`
- **类型**：`.js`
- **行数**：179
- **大小**：7749 字节
- **主要功能**：探测本机引擎可执行文件。 导出或内部函数包括 `subdirs`、`extraDirs`、`runnable`、`findIn`、`searchDirs`、`augmentedPath`、`askLoginShell`、`forget`、`resolveBin`。
- **维护建议**：改此文件前先跑 `node --check engines/which.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.98 `engines/win.js`

- **所在目录**：`engines`
- **文件名**：`win.js`
- **类型**：`.js`
- **行数**：150
- **大小**：7491 字节
- **主要功能**：Windows 引擎路径。 导出或内部函数包括 `shimScript`、`pickNode`、`escapeArg`、`launchPlan`、`killTree`、`isWin`、`isBatch`。
- **维护建议**：改此文件前先跑 `node --check engines/win.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.99 `eval/baseline.json`

- **所在目录**：`eval`
- **文件名**：`baseline.json`
- **类型**：`.json`
- **行数**：56
- **大小**：909 字节
- **主要功能**：评测题库与运行器。
- **维护建议**：改此文件前先跑 `node --check eval/baseline.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.100 `eval/run.js`

- **所在目录**：`eval`
- **文件名**：`run.js`
- **类型**：`.js`
- **行数**：416
- **大小**：23828 字节
- **主要功能**：黑盒评测运行器：pass@k、AI 评委、基线对比。 导出或内部函数包括 `failCode`、`seedMemories`、`gitCommit`、`artifactExcerpts`、`judgeOne`、`main`、`STAMP`、`argOf`、`hasFlag`、`turnsOf`、`stepsFor`、`timeoutFor`。
- **维护建议**：改此文件前先跑 `node --check eval/run.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.101 `eval/tasks.js`

- **所在目录**：`eval`
- **文件名**：`tasks.js`
- **类型**：`.js`
- **行数**：520
- **大小**：29226 字节
- **主要功能**：评测题库。 导出或内部函数包括 `runNode`、`htmlIntact`、`exists`、`read`、`ck`。
- **维护建议**：改此文件前先跑 `node --check eval/tasks.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.102 `evolve.js`

- **所在目录**：`.`
- **文件名**：`evolve.js`
- **类型**：`.js`
- **行数**：655
- **大小**：42970 字节
- **主要功能**：自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。 导出或内部函数包括 `classifyToolError`、`classifyEvent`、`readSessions`、`mineSignals`、`readFeedback`、`recordFeedback`、`feedbackSummary`、`activeRules`、`ruleDigest`、`rulesChars`、`promptBlock`、`gateProposal` 等共 27 个函数。
- **维护建议**：改此文件前先跑 `node --check evolve.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.103 `experts-lib.js`

- **所在目录**：`.`
- **文件名**：`experts-lib.js`
- **类型**：`.js`
- **行数**：84
- **大小**：4575 字节
- **主要功能**：专家与专家团加载。 导出或内部函数包括 `validateExperts`、`mergeBuiltinExperts`。
- **维护建议**：改此文件前先跑 `node --check experts-lib.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.104 `experts.json`

- **所在目录**：`.`
- **文件名**：`experts.json`
- **类型**：`.json`
- **行数**：585
- **大小**：38802 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check experts.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.105 `feishu-doc.js`

- **所在目录**：`.`
- **文件名**：`feishu-doc.js`
- **类型**：`.js`
- **行数**：337
- **大小**：14537 字节
- **主要功能**：飞书 convert API 写云文档。 导出或内部函数包括 `feishuFetch`、`getToken`、`inlineEls`、`mdToBlocks`、`splitMarkdownImages`、`convertViaApi`、`appendDescendants`、`appendImage`、`sleepAbortable`、`createFeishuDoc`、`isPermissionError`。
- **维护建议**：改此文件前先跑 `node --check feishu-doc.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.106 `gen-cache.js`

- **所在目录**：`.`
- **文件名**：`gen-cache.js`
- **类型**：`.js`
- **行数**：231
- **大小**：9443 字节
- **主要功能**：生成结果内容寻址缓存，同一格重跑不二次扣费。 导出或内部函数包括 `endpointOf`、`fileFingerprint`、`key`、`load`、`save`、`prune`、`get`、`put`、`stats`、`clear`。
- **维护建议**：改此文件前先跑 `node --check gen-cache.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.107 `goal.js`

- **所在目录**：`.`
- **文件名**：`goal.js`
- **类型**：`.js`
- **行数**：282
- **大小**：16117 字节
- **主要功能**：Goal 目标模式：拆验收标准、机器实测、Jev 判断模型、最多三轮补跑。 导出或内部函数包括 `parseJsonLoose`、`createGoalEngine`。
- **维护建议**：改此文件前先跑 `node --check goal.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.108 `htmlshot.js`

- **所在目录**：`.`
- **文件名**：`htmlshot.js`
- **类型**：`.js`
- **行数**：70
- **大小**：2985 字节
- **主要功能**：HTML 离屏截图串行队列。 导出或内部函数包括 `renderHtmlToPng`、`doRender`。
- **维护建议**：改此文件前先跑 `node --check htmlshot.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.109 `icons.js`

- **所在目录**：`.`
- **文件名**：`icons.js`
- **类型**：`.js`
- **行数**：34
- **大小**：1096 字节
- **主要功能**：图标资源。 导出或内部函数包括 `iconNames`、`isIconName`。
- **维护建议**：改此文件前先跑 `node --check icons.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.110 `im-ilink.js`

- **所在目录**：`.`
- **文件名**：`im-ilink.js`
- **类型**：`.js`
- **行数**：482
- **大小**：19966 字节
- **主要功能**：微信 iLink 扫码登录通道。 导出或内部函数包括 `randomUin`、`splitText`、`hasCdn`、`describeItems`、`dedupKey`、`fetchQrcode`、`pollQrStatus`、`createIlinkConnection`。
- **维护建议**：改此文件前先跑 `node --check im-ilink.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.111 `im-media.js`

- **所在目录**：`.`
- **文件名**：`im-media.js`
- **类型**：`.js`
- **行数**：258
- **大小**：11970 字节
- **主要功能**：IM 入站媒体（图片语音文件）落盘与转交 agent。 导出或内部函数包括 `sniffExt`、`safeBaseName`、`stamp`、`defaultName`、`saveInbound`、`fetchBuffer`、`fileNameFromHeaders`、`parseAesKey`、`decryptAesEcb`、`buildCdnDownloadUrl`、`downloadWechatCdn`、`encryptAesEcb` 等共 16 个函数。
- **维护建议**：改此文件前先跑 `node --check im-media.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.112 `im-qq.js`

- **所在目录**：`.`
- **文件名**：`im-qq.js`
- **类型**：`.js`
- **行数**：316
- **大小**：10540 字节
- **主要功能**：QQ 机器人通道。 导出或内部函数包括 `getWS`、`splitText`、`createQQConnection`。
- **维护建议**：改此文件前先跑 `node --check im-qq.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.113 `im-store.js`

- **所在目录**：`.`
- **文件名**：`im-store.js`
- **类型**：`.js`
- **行数**：133
- **大小**：5414 字节
- **主要功能**：IM 会话独立落盘，重启不失忆。 导出或内部函数包括 `createImSessionStore`。
- **维护建议**：改此文件前先跑 `node --check im-store.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.114 `im-wechat.js`

- **所在目录**：`.`
- **文件名**：`im-wechat.js`
- **类型**：`.js`
- **行数**：381
- **大小**：17304 字节
- **主要功能**：微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。 导出或内部函数包括 `aesKeyOf`、`pkcs7Strip`、`pkcs7Pad`、`msgSignature`、`decryptMsg`、`encryptMsg`、`xmlField`、`splitBytes`、`mediaKindOf`、`uploadWechatMedia`、`makeTokenCache`、`mediaFields` 等共 14 个函数。
- **维护建议**：改此文件前先跑 `node --check im-wechat.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.115 `im.js`

- **所在目录**：`.`
- **文件名**：`im.js`
- **类型**：`.js`
- **行数**：1682
- **大小**：88223 字节
- **主要功能**：IM 远程指挥总控：飞书、企微、钉钉、Webhook 入站出站、会话持久化、成果回传。 导出或内部函数包括 `dropVectorTwins`、`unwrapFeishuInbound`、`feishuDedupeKeys`、`createImRouter`。
- **维护建议**：改此文件前先跑 `node --check im.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.116 `install-mac.sh`

- **所在目录**：`.`
- **文件名**：`install-mac.sh`
- **类型**：`.sh`
- **行数**：95
- **大小**：5209 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check install-mac.sh`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.117 `install.sh`

- **所在目录**：`.`
- **文件名**：`install.sh`
- **类型**：`.sh`
- **行数**：93
- **大小**：3470 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check install.sh`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.118 `intranet.js`

- **所在目录**：`.`
- **文件名**：`intranet.js`
- **类型**：`.js`
- **行数**：41
- **大小**：2253 字节
- **主要功能**：内网探测与绑定。 导出或内部函数包括 `isIntranet`。
- **维护建议**：改此文件前先跑 `node --check intranet.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.119 `jev.js`

- **所在目录**：`.`
- **文件名**：`jev.js`
- **类型**：`.js`
- **行数**：166
- **大小**：9138 字节
- **主要功能**：Jev 协议拼装。 导出或内部函数包括 `trim`、`keyOfProvider`、`pickRoute`、`status`、`ask`、`pick`、`selftest`。
- **维护建议**：改此文件前先跑 `node --check jev.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.120 `json-compress.js`

- **所在目录**：`.`
- **文件名**：`json-compress.js`
- **类型**：`.js`
- **行数**：108
- **大小**：4912 字节
- **主要功能**：JSON 压缩。 导出或内部函数包括 `compress`、`createJsonCompress`。
- **维护建议**：改此文件前先跑 `node --check json-compress.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.121 `lanes.js`

- **所在目录**：`.`
- **文件名**：`lanes.js`
- **类型**：`.js`
- **行数**：106
- **大小**：4797 字节
- **主要功能**：办公/工程两条泳道。 导出或内部函数包括 `normalize`、`get`、`laneOf`、`engineSessionFor`、`rememberEngineSession`。
- **维护建议**：改此文件前先跑 `node --check lanes.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.122 `lark-cli.js`

- **所在目录**：`.`
- **文件名**：`lark-cli.js`
- **类型**：`.js`
- **行数**：75
- **大小**：4449 字节
- **主要功能**：飞书全家桶 CLI 绑定。 导出或内部函数包括 `parseConfigShow`、`verifyUrlOf`、`explainLarkError`、`usableSecret`、`appConsoleUrl`。
- **维护建议**：改此文件前先跑 `node --check lark-cli.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.123 `lifecycle.js`

- **所在目录**：`.`
- **文件名**：`lifecycle.js`
- **类型**：`.js`
- **行数**：291
- **大小**：16027 字节
- **主要功能**：入职离职：一次关权限不删数据，出交接回执。 导出或内部函数包括 `reassignTo`、`offboard`、`onboard`、`deptTemplate`、`setDeptTemplate`、`listDeptTemplates`、`receiptText`。
- **维护建议**：改此文件前先跑 `node --check lifecycle.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.124 `llm.js`

- **所在目录**：`.`
- **文件名**：`llm.js`
- **类型**：`.js`
- **行数**：869
- **大小**：43413 字节
- **主要功能**：多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、失败重试与缓存命中字段统一。 导出或内部函数包括 `repairToolPairs`、`toAnthropicMessages`、`anthropicBase`、`channelEnvName`、`hostOf`、`resolveKey`、`cleanKey`、`headerKey`、`anthropicChat`、`openaiUsage`、`toOpenAIMessages`、`rescueLeakedToolCalls` 等共 25 个函数。
- **维护建议**：改此文件前先跑 `node --check llm.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.125 `llms.txt`

- **所在目录**：`.`
- **文件名**：`llms.txt`
- **类型**：`.txt`
- **行数**：36
- **大小**：2189 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check llms.txt`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.126 `log.js`

- **所在目录**：`.`
- **文件名**：`log.js`
- **类型**：`.js`
- **行数**：149
- **大小**：7201 字节
- **主要功能**：日志。 导出或内部函数包括 `today`、`fileOf`、`ensureDir`、`pruneOld`、`log`、`tail`、`days`、`setLevel`、`debug`、`info`、`warn`、`error`。
- **维护建议**：改此文件前先跑 `node --check log.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.127 `mailer.js`

- **所在目录**：`.`
- **文件名**：`mailer.js`
- **类型**：`.js`
- **行数**：233
- **大小**：8926 字节
- **主要功能**：SMTP 发信与收件人白名单。 导出或内部函数包括 `configured`、`fromAddr`、`parseAddrs`、`allowRules`、`addrAllowed`、`checkRecipients`、`checkAttachments`、`fmtBytes`、`scrub`、`transportOf`、`verify`、`send`。
- **维护建议**：改此文件前先跑 `node --check mailer.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.128 `mcp-catalog.js`

- **所在目录**：`.`
- **文件名**：`mcp-catalog.js`
- **类型**：`.js`
- **行数**：205
- **大小**：19436 字节
- **主要功能**：连接器广场目录与探测。 导出或内部函数包括 `findCmd`、`onPath`、`resolve`、`catalog`、`std`、`uvx`、`http`。
- **维护建议**：改此文件前先跑 `node --check mcp-catalog.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.129 `mcp.js`

- **所在目录**：`.`
- **文件名**：`mcp.js`
- **类型**：`.js`
- **行数**：443
- **大小**：18706 字节
- **主要功能**：MCP 客户端：stdio / Streamable HTTP，工具注入 agent，生命周期管理。 导出或内部函数包括 `whyFailed`。
- **维护建议**：改此文件前先跑 `node --check mcp.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.130 `md-tty.js`

- **所在目录**：`.`
- **文件名**：`md-tty.js`
- **类型**：`.js`
- **行数**：218
- **大小**：10564 字节
- **主要功能**：终端 Markdown 渲染。 导出或内部函数包括 `safeCut`、`inline`、`undecided`、`blockOf`、`createRenderer`。
- **维护建议**：改此文件前先跑 `node --check md-tty.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.131 `media-health.js`

- **所在目录**：`.`
- **文件名**：`media-health.js`
- **类型**：`.js`
- **行数**：154
- **大小**：8866 字节
- **主要功能**：媒体渠道熔断：连挂硬错后暂停，提示词里写明。 导出或内部函数包括 `fingerprint`、`statusOf`、`looksNetwork`、`looksBroke`、`looksNoModel`、`now`、`gate`、`record`、`reset`、`list`。
- **维护建议**：改此文件前先跑 `node --check media-health.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.132 `media-models.js`

- **所在目录**：`.`
- **文件名**：`media-models.js`
- **类型**：`.js`
- **行数**：705
- **大小**：43870 字节
- **主要功能**：生图/生视频/配音/转写/看图多模型与五家视频协议识别。 文件头摘录：图像 / 视频 / 语音 / 视觉 / 转写这五路模型的「多模型 + 共用 Key」层。 老配置长这样，一路只能配一个模型，而且 Key 要一路填一遍：   config.media = { image: {base_url, api_key, model}, video: {...}, tts: {...}, vision: {...} } 可现实是：一把 OpenRouter 的 Key 能同时喂图、视频、视觉；一把火山方舟的 Key 也是。 同一把 Key 抄四遍，换 Key 的时候就得记着改四处——漏一处，某一路就在半年后突然 401。 所以拆成两张表：   config.providers    = [{ id, name, kind, base_url, api_key }]      一把 Key 一行   config.media_models = [{ id, cap,  导出或内部函数包括 `protoOfKind`、`guessCap`、`capOfModel`、`uniqueId`、`baseForUse`、`providerKeyOf`、`guessKind`、`videoProtoOf`、`baseOfKind`、`normalizeProviders`、`dedupeProviders`、`normalize` 等共 24 个函数。
- **维护建议**：改此文件前先跑 `node --check media-models.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.133 `memory.js`

- **所在目录**：`.`
- **文件名**：`memory.js`
- **类型**：`.js`
- **行数**：551
- **大小**：26792 字节
- **主要功能**：双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、过期能力断言拦截、命中回写淘汰。 导出或内部函数包括 `looksSecret`、`looksStaleClaim`、`noteUsed`、`flushHits`、`keepScore`、`normalize`、`load`、`save`、`setEmbedder`、`vecLoad`、`vecSave`、`ensureVectors` 等共 28 个函数。
- **维护建议**：改此文件前先跑 `node --check memory.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.134 `metrics.js`

- **所在目录**：`.`
- **文件名**：`metrics.js`
- **类型**：`.js`
- **行数**：291
- **大小**：13120 字节
- **主要功能**：运维指标与 Prometheus 导出。 导出或内部函数包括 `bump`、`observe`、`pct`、`diskFreePct`、`channelStreaks`、`auditBlocked`、`snapshot`、`shardOf`、`fileOf`、`shards`、`write`、`read` 等共 18 个函数。
- **维护建议**：改此文件前先跑 `node --check metrics.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.135 `migrate.js`

- **所在目录**：`.`
- **文件名**：`migrate.js`
- **类型**：`.js`
- **行数**：208
- **大小**：10495 字节
- **主要功能**：升级迁移。 导出或内部函数包括 `stampToday`、`readLedger`、`backupCanvases`、`tidyLooseFiles`、`runMigrations`。
- **维护建议**：改此文件前先跑 `node --check migrate.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.136 `modes.js`

- **所在目录**：`.`
- **文件名**：`modes.js`
- **类型**：`.js`
- **行数**：76
- **大小**：4603 字节
- **主要功能**：执行模式唯一真源：Ask / Plan / Craft / Goal，避免界面与 CLI 漂移。 导出或内部函数包括 `modeOf`、`normalizeMode`、`isMode`、`agentMode`、`isGoalMode`、`modeLabel`、`modeHint`。
- **维护建议**：改此文件前先跑 `node --check modes.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.137 `notify.js`

- **所在目录**：`.`
- **文件名**：`notify.js`
- **类型**：`.js`
- **行数**：51
- **大小**：1707 字节
- **主要功能**：群机器人推送。 导出或内部函数包括 `pushWecom`、`pushDingtalk`、`pushBots`。
- **维护建议**：改此文件前先跑 `node --check notify.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.138 `org.js`

- **所在目录**：`.`
- **文件名**：`org.js`
- **类型**：`.js`
- **行数**：490
- **大小**：23159 字节
- **主要功能**：多租户组织、席位、套餐。 导出或内部函数包括 `normalizeDeptTemplates`、`money`、`normalizeBudget`、`emptyDb`、`load`、`save`、`newId`、`ensureDefault`、`settingsOf`、`listOrgs`、`getOrg`、`multiTenant` 等共 28 个函数。
- **维护建议**：改此文件前先跑 `node --check org.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.139 `package-lock.json`

- **所在目录**：`.`
- **文件名**：`package-lock.json`
- **类型**：`.json`
- **行数**：7076
- **大小**：262762 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check package-lock.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.140 `package.json`

- **所在目录**：`.`
- **文件名**：`package.json`
- **类型**：`.json`
- **行数**：56
- **大小**：1832 字节
- **主要功能**：项目支撑文件。
- **维护建议**：改此文件前先跑 `node --check package.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.141 `paths.js`

- **所在目录**：`.`
- **文件名**：`paths.js`
- **类型**：`.js`
- **行数**：133
- **大小**：5906 字节
- **主要功能**：数据目录解析：OPENWORKBUDDY_HOME / 打包后用户目录 / 源码旁 data。 导出或内部函数包括 `isPackaged`、`dataPath`、`appPath`、`preferData`、`copyTree`、`copyIfMissing`、`seedDataDir`、`resolvePort`。
- **维护建议**：改此文件前先跑 `node --check paths.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.142 `pet-preload.js`

- **所在目录**：`.`
- **文件名**：`pet-preload.js`
- **类型**：`.js`
- **行数**：12
- **大小**：642 字节
- **主要功能**：宠物窗口预加载脚本。
- **维护建议**：改此文件前先跑 `node --check pet-preload.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.143 `pet-sprites.js`

- **所在目录**：`.`
- **文件名**：`pet-sprites.js`
- **类型**：`.js`
- **行数**：229
- **大小**：10283 字节
- **主要功能**：像素宠物精灵与 Codex/Petdex 兼容。 导出或内部函数包括 `pngSize`、`webpSize`、`imageSize`、`checkSheet`、`localRoot`、`petRoots`、`readPetDir`、`scanPets`、`findPet`、`sheetDataUrl`、`spriteSpec`。
- **维护建议**：改此文件前先跑 `node --check pet-sprites.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.144 `pet.js`

- **所在目录**：`.`
- **文件名**：`pet.js`
- **类型**：`.js`
- **行数**：421
- **大小**：20440 字节
- **主要功能**：桌面宠物窗口与状态机。 导出或内部函数包括 `photoDataUrl`、`spriteBundle`、`posFile`、`loadPos`、`savePos`、`defaultPos`、`sanePos`、`toggleMain`、`bindIpc`、`create`、`push`、`setState` 等共 23 个函数。
- **维护建议**：改此文件前先跑 `node --check pet.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.145 `plugins.js`

- **所在目录**：`.`
- **文件名**：`plugins.js`
- **类型**：`.js`
- **行数**：449
- **大小**：23237 字节
- **主要功能**：Agent Plugins 1.0.0：一个包同时带技能和 MCP。 导出或内部函数包括 `containedIn`、`validateManifest`、`discoverSkills`、`expandVars`、`discoverMcpServers`、`loadPlugin`、`loadPlugins`、`pluginSkills`、`pluginMcpServers`、`safePluginDirName`、`readSources`、`writeSources` 等共 18 个函数。
- **维护建议**：改此文件前先跑 `node --check plugins.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.146 `prefs.js`

- **所在目录**：`.`
- **文件名**：`prefs.js`
- **类型**：`.js`
- **行数**：247
- **大小**：11394 字节
- **主要功能**：按账号存储的引擎、思考档、外观偏好。 导出或内部函数包括 `withPrefs`、`current`、`keyOf`、`fileOf`、`read`、`write`、`merge`、`isPersonalPatch`、`split`、`agentCfg`、`agentView`、`petCfg` 等共 14 个函数。
- **维护建议**：改此文件前先跑 `node --check prefs.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.147 `preview.js`

- **所在目录**：`.`
- **文件名**：`preview.js`
- **类型**：`.js`
- **行数**：384
- **大小**：17438 字节
- **主要功能**：docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。 导出或内部函数包括 `readZip`、`decodeEntities`、`parseXml`、`findAll`、`textOf`、`docxRuns`、`docxParagraph`、`docxToDoc`、`pptxToSlides`、`xlsxToSheets`、`zipListing`、`previewData` 等共 15 个函数。
- **维护建议**：改此文件前先跑 `node --check preview.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.148 `pricing.js`

- **所在目录**：`.`
- **文件名**：`pricing.js`
- **类型**：`.js`
- **行数**：455
- **大小**：24788 字节
- **主要功能**：积分与单价表。 导出或内部函数包括 `unitTableFor`、`normalizeUnitRow`、`unitPriceOf`、`costOfUnits`、`candidates`、`tableFor`、`normalizeRow`、`priceOf`、`costOf`、`discountOf`、`r6`、`yuanText` 等共 14 个函数。
- **维护建议**：改此文件前先跑 `node --check pricing.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.149 `public/admin.html`

- **所在目录**：`public`
- **文件名**：`admin.html`
- **类型**：`.html`
- **行数**：379
- **大小**：25832 字节
- **主要功能**：前端静态资源，无构建直接由 Express 提供。
- **维护建议**：改此文件前先跑 `node --check public/admin.html`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.150 `public/css/ui.css`

- **所在目录**：`public/css`
- **文件名**：`ui.css`
- **类型**：`.css`
- **行数**：1217
- **大小**：118722 字节
- **主要功能**：前端静态资源，无构建直接由 Express 提供。
- **维护建议**：改此文件前先跑 `node --check public/css/ui.css`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.151 `public/icon.png`

- **所在目录**：`public`
- **文件名**：`icon.png`
- **类型**：`.png`
- **行数**：0
- **大小**：446974 字节
- **主要功能**：前端静态资源，无构建直接由 Express 提供。
- **维护建议**：改此文件前先跑 `node --check public/icon.png`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.152 `public/index.html`

- **所在目录**：`public`
- **文件名**：`index.html`
- **类型**：`.html`
- **行数**：2737
- **大小**：286772 字节
- **主要功能**：前端静态资源，无构建直接由 Express 提供。
- **维护建议**：改此文件前先跑 `node --check public/index.html`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.153 `public/js/admin.js`

- **所在目录**：`public/js`
- **文件名**：`admin.js`
- **类型**：`.js`
- **行数**：2717
- **大小**：161767 字节
- **主要功能**：管理后台前端。 导出或内部函数包括 `fmtTs`、`fmtDate`、`ago`、`api`、`toast`、`act`、`modal`、`confirmBox`、`statCols`、`table`、`presetRange`、`activePreset` 等共 61 个函数。
- **维护建议**：改此文件前先跑 `node --check public/js/admin.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.154 `public/js/app-00-ui.js`

- **所在目录**：`public/js`
- **文件名**：`app-00-ui.js`
- **类型**：`.js`
- **行数**：388
- **大小**：19387 字节
- **主要功能**：前端 UI 基础：主题、字号、组件。 导出或内部函数包括 `ic`、`setMsg`、`isIconName`、`ava`、`avaCell`、`avaPicks`、`markActivatable`、`onActivate`、`rszRoom`、`setPanelW`、`resetPanelW`、`applyStoredW` 等共 21 个函数。
- **维护建议**：改此文件前先跑 `node --check public/js/app-00-ui.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.155 `public/js/app-01.js`

- **所在目录**：`public/js`
- **文件名**：`app-01.js`
- **类型**：`.js`
- **行数**：4102
- **大小**：250829 字节
- **主要功能**：前端会话、消息、文件面板。 导出或内部函数包括 `renderCtxMeter`、`resetCtxMeter`、`procNote`、`setBusySendMode`、`saveSessions`、`esc`、`prettyUrl`、`autoLinkUrls`、`getList`、`amPlatformOwner`、`canOpenOnHost`、`dirOf` 等共 147 个函数。
- **维护建议**：改此文件前先跑 `node --check public/js/app-01.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.156 `public/js/app-02.js`

- **所在目录**：`public/js`
- **文件名**：`app-02.js`
- **类型**：`.js`
- **行数**：2529
- **大小**：138935 字节
- **主要功能**：前端任务直播、过程卡、模式切换。 文件头摘录：这条模型现在点下去能不能真跑起来。 本机服务（Ollama 之类）不要 Key，填不填都算能用；其余看 has_key—— 多人服务器上普通成员拿到的 api_key 是一串星号，只有这个布尔是真的。 导出或内部函数包括 `modelReady`、`renderModelMenu`、`renderGoalCard`、`setWorkspaceDir`、`renderWsMenu`、`modeInfo`、`setMode`、`loadExecModes`、`attachKind`、`markerName`、`attachmentOrder`、`removeAttachmentMarker` 等共 115 个函数。
- **维护建议**：改此文件前先跑 `node --check public/js/app-02.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.157 `public/js/app-03.js`

- **所在目录**：`public/js`
- **文件名**：`app-03.js`
- **类型**：`.js`
- **行数**：1835
- **大小**：123655 字节
- **主要功能**：前端设置与安全。 文件头摘录：缓存命中率。agent 每走一步都要把整段上下文重发一遍，真实账本里 prompt 和 completion 是 64:1 —— 这一大坨到底是全价重买还是走了缓存，光看 tokens 总数完全看不出来， 而它才是账单的大头。命中率低说明系统提示词或历史在被反复改动（换模型、改设置、 时间戳精度太细都会打断前缀），值得去看一眼。 cachedOf 是「记过这个字段的那些条的 prompt 总量」：老流水没这个字段，不参与计算， 所以这里返回空串而不是「0%」—— 没统计过不等于没命中。 导出或内部函数包括 `cacheTxt`、`renderAccount`、`showAuth`、`applyAuthMode`、`forgotHintHtml`、`submitAuth`、`submitPair`、`showAuthBlocked`、`showTwoFactorGate`、`initAuth`、`mergeServerSessions`、`keyLink` 等共 52 个函数。
- **维护建议**：改此文件前先跑 `node --check public/js/app-03.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.158 `public/js/app-04.js`

- **所在目录**：`public/js`
- **文件名**：`app-04.js`
- **类型**：`.js`
- **行数**：1558
- **大小**：115341 字节
- **主要功能**：前端专家技能连接器广场。 导出或内部函数包括 `updateEvalView`、`openEvalDetail`、`libTaskOf`、`libOwnerTask`、`libTurnOf`、`libIcon`、`libSize`、`libKindOk`、`libKindOf`、`libUrl`、`libTimeBucket`、`libGroupFiles` 等共 32 个函数。
- **维护建议**：改此文件前先跑 `node --check public/js/app-04.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.159 `public/js/app-05.js`

- **所在目录**：`public/js`
- **文件名**：`app-05.js`
- **类型**：`.js`
- **行数**：3327
- **大小**：231255 字节
- **主要功能**：前端模型表、媒体、Trace。 导出或内部函数包括 `renderHubMcp`、`renderPromptPage`、`startTaskWith`、`startTaskUsing`、`cronToHuman`、`localMin`、`whenToHuman`、`renderSettings`、`saveSettings`、`loadMediaCatalog`、`saveAllModelTables`、`saveMediaTables` 等共 90 个函数。
- **维护建议**：改此文件前先跑 `node --check public/js/app-05.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.160 `public/js/app-06.js`

- **所在目录**：`public/js`
- **文件名**：`app-06.js`
- **类型**：`.js`
- **行数**：932
- **大小**：66539 字节
- **主要功能**：前端评测、自进化、备份。 导出或内部函数包括 `renderSecurityPane`、`renderTwoFactorBox`、`renderShortcutsPane`、`renderEvolvePane`、`renderLookPane`、`renderAboutPane`。
- **维护建议**：改此文件前先跑 `node --check public/js/app-06.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.161 `public/js/app-07-canvas.js`

- **所在目录**：`public/js`
- **文件名**：`app-07-canvas.js`
- **类型**：`.js`
- **行数**：2462
- **大小**：209490 字节
- **主要功能**：无限画布 JointJS 底座。 导出或内部函数包括 `canvasStorageKey`、`canvasToast`、`canvasType`、`canvasPayload`、`canvasKind`、`canvasEndpointId`、`canvasRelationLabel`、`canvasDefaultRelation`、`canvasLinkRelation`、`canvasRelationOptions`、`canvasNodeLabel`、`canvasSafeText` 等共 145 个函数。
- **维护建议**：改此文件前先跑 `node --check public/js/app-07-canvas.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.162 `public/js/app-07-drama.js`

- **所在目录**：`public/js`
- **文件名**：`app-07-drama.js`
- **类型**：`.js`
- **行数**：421
- **大小**：26263 字节
- **主要功能**：短剧业务节点与生成回写。 导出或内部函数包括 `dramaFileUrl`、`dramaBaseName`、`dramaShotId`、`dramaChars`、`dramaAllShots`、`dramaToast`、`dramaList`、`dramaLoad`、`dramaSave`、`dramaSaveShot`、`dramaRerun`、`dramaHistDiff` 等共 28 个函数。
- **维护建议**：改此文件前先跑 `node --check public/js/app-07-drama.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.163 `public/js/i18n.js`

- **所在目录**：`public/js`
- **文件名**：`i18n.js`
- **类型**：`.js`
- **行数**：1615
- **大小**：116538 字节
- **主要功能**：中英文案。
- **维护建议**：改此文件前先跑 `node --check public/js/i18n.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.164 `public/pet.html`

- **所在目录**：`public`
- **文件名**：`pet.html`
- **类型**：`.html`
- **行数**：343
- **大小**：21585 字节
- **主要功能**：前端静态资源，无构建直接由 Express 提供。
- **维护建议**：改此文件前先跑 `node --check public/pet.html`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.165 `public/svgfig.js`

- **所在目录**：`public`
- **文件名**：`svgfig.js`
- **类型**：`.js`
- **行数**：165
- **大小**：9171 字节
- **主要功能**：流式 SVG 清洗与主题内联。
- **维护建议**：改此文件前先跑 `node --check public/svgfig.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.166 `quota.js`

- **所在目录**：`.`
- **文件名**：`quota.js`
- **类型**：`.js`
- **行数**：415
- **大小**：20758 字节
- **主要功能**：付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。 导出或内部函数包括 `emptyDb`、`load`、`loadAll`、`save`、`localDay`、`localMonth`、`normalizeCap`、`quotaTable`、`withActor`、`currentActor`、`used`、`check` 等共 20 个函数。
- **维护建议**：改此文件前先跑 `node --check quota.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.167 `rbac.js`

- **所在目录**：`.`
- **文件名**：`rbac.js`
- **类型**：`.js`
- **行数**：112
- **大小**：6406 字节
- **主要功能**：RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。 导出或内部函数包括 `roleOf`、`rankOf`、`can`、`outranks`、`assignProblem`、`upper`、`assignableBy`、`same`。
- **维护建议**：改此文件前先跑 `node --check rbac.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.168 `relay-files.js`

- **所在目录**：`.`
- **文件名**：`relay-files.js`
- **类型**：`.js`
- **行数**：162
- **大小**：7446 字节
- **主要功能**：中转站生成文件暂存。 导出或内部函数包括 `typeOf`、`ensure`、`readMeta`、`save`、`get`、`remove`、`sweep`、`list`、`binOf`、`metaOf`、`okId`。
- **维护建议**：改此文件前先跑 `node --check relay-files.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.169 `relay.js`

- **所在目录**：`.`
- **文件名**：`relay.js`
- **类型**：`.js`
- **行数**：873
- **大小**：46528 字节
- **主要功能**：OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。 导出或内部函数包括 `worthRetry`、`pickChannels`、`num`、`weightedShuffle`、`prepBody`、`usageOf`、`forward`、`createRouter`。
- **维护建议**：改此文件前先跑 `node --check relay.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.170 `repl-commands.js`

- **所在目录**：`.`
- **文件名**：`repl-commands.js`
- **类型**：`.js`
- **行数**：739
- **大小**：39575 字节
- **主要功能**：REPL 斜杠命令。 导出或内部函数包括 `find`、`nearest`、`parse`、`mergePaste`、`resolveCd`、`complete`、`menu`、`modelRows`、`modelListText`、`pickModelRow`、`ago`、`sessionRows` 等共 33 个函数。
- **维护建议**：改此文件前先跑 `node --check repl-commands.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.171 `scheduler.js`

- **所在目录**：`.`
- **文件名**：`scheduler.js`
- **类型**：`.js`
- **行数**：606
- **大小**：29741 字节
- **主要功能**：定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。 导出或内部函数包括 `describeCron`、`setActiveScheduler`、`activeScheduler`、`loadStore`、`saveStore`、`parseField`、`parseAt`、`describeWhen`、`parseCron`、`cronMatches`、`createScheduler`。
- **维护建议**：改此文件前先跑 `node --check scheduler.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.172 `scripts/check-package-files.js`

- **所在目录**：`scripts`
- **文件名**：`check-package-files.js`
- **类型**：`.js`
- **行数**：263
- **大小**：11079 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。 导出或内部函数包括 `bareRequires`、`missingDeps`、`localRequires`、`walkGraph`、`missingFrom`、`assertPackComplete`、`assertDepsRequirable`、`firstLine`、`assertSlimmed`。
- **维护建议**：改此文件前先跑 `node --check scripts/check-package-files.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.173 `scripts/demo-mask.js`

- **所在目录**：`scripts`
- **文件名**：`demo-mask.js`
- **类型**：`.js`
- **行数**：56
- **大小**：2838 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。 导出或内部函数包括 `defaultPairs`、`maskScript`。
- **维护建议**：改此文件前先跑 `node --check scripts/demo-mask.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.174 `scripts/demo-readme.js`

- **所在目录**：`scripts`
- **文件名**：`demo-readme.js`
- **类型**：`.js`
- **行数**：41
- **大小**：1840 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。 文件头摘录：录完 demo 把 GIF 挂进两份 README 的首屏（徽章 / Star 那段之后、第一条 --- 分隔线之前）。 已经挂过就不动；找不到分隔线就不猜位置，交给人手挂。 纯函数放这里不放 record-demo.js：那个文件顶部 require("electron")，node 直接测不了。 导出或内部函数包括 `wireReadme`、`wireReadmes`。
- **维护建议**：改此文件前先跑 `node --check scripts/demo-readme.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.175 `scripts/demo-timing.js`

- **所在目录**：`scripts`
- **文件名**：`demo-timing.js`
- **类型**：`.js`
- **行数**：35
- **大小**：1693 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。 文件头摘录：demo 录屏的时长整形：打字和最后停在结果上的那几秒保持原速，只把「等模型干活」那段压进目标时长。 纯函数放这里，record-demo.js 顶部 require("electron")，node 直接测不了。 导出或内部函数包括 `fitDurations`。
- **维护建议**：改此文件前先跑 `node --check scripts/demo-timing.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.176 `scripts/genlogo.py`

- **所在目录**：`scripts`
- **文件名**：`genlogo.py`
- **类型**：`.py`
- **行数**：201
- **大小**：10728 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。
- **维护建议**：改此文件前先跑 `node --check scripts/genlogo.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.177 `scripts/make-icons.sh`

- **所在目录**：`scripts`
- **文件名**：`make-icons.sh`
- **类型**：`.sh`
- **行数**：35
- **大小**：1528 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。
- **维护建议**：改此文件前先跑 `node --check scripts/make-icons.sh`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.178 `scripts/make-mac-app.sh`

- **所在目录**：`scripts`
- **文件名**：`make-mac-app.sh`
- **类型**：`.sh`
- **行数**：115
- **大小**：7287 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。
- **维护建议**：改此文件前先跑 `node --check scripts/make-mac-app.sh`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.179 `scripts/record-demo.js`

- **所在目录**：`scripts`
- **文件名**：`record-demo.js`
- **类型**：`.js`
- **行数**：303
- **大小**：19181 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。 导出或内部函数包括 `parseArgs`、`seedHome`、`waitHttp`、`ensureLoggedIn`、`ffmpeg`、`installMask`、`typeInto`、`sleep`、`log`。
- **维护建议**：改此文件前先跑 `node --check scripts/record-demo.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.180 `scripts/run-app.sh`

- **所在目录**：`scripts`
- **文件名**：`run-app.sh`
- **类型**：`.sh`
- **行数**：13
- **大小**：600 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。
- **维护建议**：改此文件前先跑 `node --check scripts/run-app.sh`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.181 `scripts/shot-card.js`

- **所在目录**：`scripts`
- **文件名**：`shot-card.js`
- **类型**：`.js`
- **行数**：84
- **大小**：3323 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。 导出或内部函数包括 `sleep`、`shoot`、`keyScript`。
- **维护建议**：改此文件前先跑 `node --check scripts/shot-card.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.182 `scripts/shot-ui.js`

- **所在目录**：`scripts`
- **文件名**：`shot-ui.js`
- **类型**：`.js`
- **行数**：172
- **大小**：10124 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。 导出或内部函数包括 `pageScript`、`serveStatic`、`shoot`。
- **维护建议**：改此文件前先跑 `node --check scripts/shot-ui.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.183 `scripts/stats.js`

- **所在目录**：`scripts`
- **文件名**：`stats.js`
- **类型**：`.js`
- **行数**：61
- **大小**：3257 字节
- **主要功能**：维护脚本：统计、图标、演示录制、打包检查。 导出或内部函数包括 `read`。
- **维护建议**：改此文件前先跑 `node --check scripts/stats.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.184 `security.js`

- **所在目录**：`.`
- **文件名**：`security.js`
- **类型**：`.js`
- **行数**：631
- **大小**：31148 字节
- **主要功能**：安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERNS、引擎档位翻译。 导出或内部函数包括 `permissionMode`、`engineGuard`、`getSecurity`、`audit`、`auditList`、`auditClear`、`auditExport`、`expandPath`、`underPrefix`、`resolvePathWithPolicy`、`splitSegments`、`stripEnvAssign` 等共 33 个函数。
- **维护建议**：改此文件前先跑 `node --check security.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.185 `server.js`

- **所在目录**：`.`
- **文件名**：`server.js`
- **类型**：`.js`
- **行数**：7248
- **大小**：416238 字节
- **主要功能**：Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被 Electron 主进程或 `node server.js` 直接拉起。 导出或内部函数包括 `fillDefaults`、`llmForSession`、`recordModelHealth`、`healthSummary`、`saveExperts`、`persistRunning`、`sweepInterruptedRuns`、`assignSessionDir`、`sessFile`、`sessStat`、`sessChangedOnDisk`、`getSession` 等共 140 个函数。
- **维护建议**：改此文件前先跑 `node --check server.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.186 `session-search.js`

- **所在目录**：`.`
- **文件名**：`session-search.js`
- **类型**：`.js`
- **行数**：199
- **大小**：9843 字节
- **主要功能**：任务历史检索：正文、产出文件名、意思相近。 导出或内部函数包括 `assistantText`、`filesOf`、`digestOf`、`embedTextOf`、`normalize`、`bigrams`、`keywordScore`、`cosine`、`snippet`、`rank`、`searchNote`。
- **维护建议**：改此文件前先跑 `node --check session-search.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.187 `shot-history.js`

- **所在目录**：`.`
- **文件名**：`shot-history.js`
- **类型**：`.js`
- **行数**：371
- **大小**：18060 字节
- **主要功能**：分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。 导出或内部函数包括 `insideRoot`、`probeFile`、`putFile`、`readLedger`、`appendLedger`、`findTarget`、`stable`、`fingerprint`、`snapshot`、`blobState`、`list`、`changed` 等共 20 个函数。
- **维护建议**：改此文件前先跑 `node --check shot-history.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.188 `skill-guard.js`

- **所在目录**：`.`
- **文件名**：`skill-guard.js`
- **类型**：`.js`
- **行数**：580
- **大小**：36924 字节
- **主要功能**：技能静态安检：三十余条规则分三档，强装留档。 导出或内部函数包括 `isBinaryName`、`visible`、`locate`、`negated`、`frontmatterEnd`、`scanText`、`taint`、`scanDir`、`scanOne`、`explain`。
- **维护建议**：改此文件前先跑 `node --check skill-guard.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.189 `skills.js`

- **所在目录**：`.`
- **文件名**：`skills.js`
- **类型**：`.js`
- **行数**：758
- **大小**：38994 字节
- **主要功能**：技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。 导出或内部函数包括 `loadSkills`、`safePluginSkills`、`safeName`、`parseFrontmatter`、`findSkillDir`、`getSkillFull`、`assertNotPluginSkill`、`samePlace`、`saveSkill`、`deleteSkill`、`copySkillFolder`、`dirSize` 等共 25 个函数。
- **维护建议**：改此文件前先跑 `node --check skills.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.190 `skills/character-photo-studio/skill.md`

- **所在目录**：`skills/character-photo-studio`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：86
- **大小**：6961 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/character-photo-studio/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.191 `skills/competitor-watch/skill.md`

- **所在目录**：`skills/competitor-watch`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：71
- **大小**：3169 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/competitor-watch/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.192 `skills/content-repurpose/skill.md`

- **所在目录**：`skills/content-repurpose`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：59
- **大小**：3173 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/content-repurpose/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.193 `skills/content-studio/skill.md`

- **所在目录**：`skills/content-studio`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：33
- **大小**：2685 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/content-studio/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.194 `skills/contract-review/skill.md`

- **所在目录**：`skills/contract-review`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：86
- **大小**：4130 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/contract-review/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.195 `skills/customer-feedback/skill.md`

- **所在目录**：`skills/customer-feedback`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：67
- **大小**：3107 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/customer-feedback/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.196 `skills/data-analysis/skill.md`

- **所在目录**：`skills/data-analysis`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：68
- **大小**：3139 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/data-analysis/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.197 `skills/data-viz/skill.md`

- **所在目录**：`skills/data-viz`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：64
- **大小**：3086 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/data-viz/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.198 `skills/deep-research/skill.md`

- **所在目录**：`skills/deep-research`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：66
- **大小**：4332 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/deep-research/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.199 `skills/docx/LICENSE.txt`

- **所在目录**：`skills/docx`
- **文件名**：`LICENSE.txt`
- **类型**：`.txt`
- **行数**：30
- **大小**：1467 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/LICENSE.txt`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.200 `skills/docx/scripts/__init__.py`

- **所在目录**：`skills/docx/scripts`
- **文件名**：`__init__.py`
- **类型**：`.py`
- **行数**：1
- **大小**：1 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/__init__.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.201 `skills/docx/scripts/accept_changes.py`

- **所在目录**：`skills/docx/scripts`
- **文件名**：`accept_changes.py`
- **类型**：`.py`
- **行数**：135
- **大小**：4051 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/accept_changes.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.202 `skills/docx/scripts/comment.py`

- **所在目录**：`skills/docx/scripts`
- **文件名**：`comment.py`
- **类型**：`.py`
- **行数**：368
- **大小**：14054 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/comment.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.203 `skills/docx/scripts/merge_runs.py`

- **所在目录**：`skills/docx/scripts`
- **文件名**：`merge_runs.py`
- **类型**：`.py`
- **行数**：310
- **大小**：9398 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/merge_runs.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.204 `skills/docx/scripts/office/helpers/__init__.py`

- **所在目录**：`skills/docx/scripts/office/helpers`
- **文件名**：`__init__.py`
- **类型**：`.py`
- **行数**：111
- **大小**：3358 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/helpers/__init__.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.205 `skills/docx/scripts/office/helpers/pptx_chart.py`

- **所在目录**：`skills/docx/scripts/office/helpers`
- **文件名**：`pptx_chart.py`
- **类型**：`.py`
- **行数**：170
- **大小**：5862 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/helpers/pptx_chart.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.206 `skills/docx/scripts/office/helpers/pptx_slide.py`

- **所在目录**：`skills/docx/scripts/office/helpers`
- **文件名**：`pptx_slide.py`
- **类型**：`.py`
- **行数**：60
- **大小**：1675 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/helpers/pptx_slide.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.207 `skills/docx/scripts/office/helpers/pptx_theme.py`

- **所在目录**：`skills/docx/scripts/office/helpers`
- **文件名**：`pptx_theme.py`
- **类型**：`.py`
- **行数**：114
- **大小**：3510 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/helpers/pptx_theme.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.208 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-chart.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`dml-chart.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：74984 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-chart.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.209 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-chartDrawing.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`dml-chartDrawing.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：6956 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-chartDrawing.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.210 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-diagram.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`dml-diagram.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：51302 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-diagram.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.211 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-lockedCanvas.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`dml-lockedCanvas.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：624 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-lockedCanvas.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.212 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-main.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`dml-main.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：152039 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-main.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.213 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-picture.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`dml-picture.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：1231 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-picture.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.214 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-spreadsheetDrawing.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`dml-spreadsheetDrawing.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：8862 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-spreadsheetDrawing.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.215 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-wordprocessingDrawing.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`dml-wordprocessingDrawing.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：14795 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-wordprocessingDrawing.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.216 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/pml.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`pml.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：83612 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/pml.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.217 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-additionalCharacteristics.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`shared-additionalCharacteristics.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：1269 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-additionalCharacteristics.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.218 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-bibliography.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`shared-bibliography.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：7328 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-bibliography.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.219 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-commonSimpleTypes.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`shared-commonSimpleTypes.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：6382 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-commonSimpleTypes.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.220 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-customXmlDataProperties.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`shared-customXmlDataProperties.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：1248 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-customXmlDataProperties.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.221 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-customXmlSchemaProperties.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`shared-customXmlSchemaProperties.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：880 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-customXmlSchemaProperties.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.222 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-documentPropertiesCustom.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`shared-documentPropertiesCustom.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：2608 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-documentPropertiesCustom.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.223 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-documentPropertiesExtended.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`shared-documentPropertiesExtended.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：3507 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-documentPropertiesExtended.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.224 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-documentPropertiesVariantTypes.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`shared-documentPropertiesVariantTypes.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：7507 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-documentPropertiesVariantTypes.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.225 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-math.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`shared-math.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：23313 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-math.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.226 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-relationshipReference.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`shared-relationshipReference.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：1367 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-relationshipReference.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.227 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/sml.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`sml.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：242277 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/sml.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.228 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-main.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`vml-main.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：26148 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-main.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.229 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-officeDrawing.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`vml-officeDrawing.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：25279 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-officeDrawing.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.230 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-presentationDrawing.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`vml-presentationDrawing.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：535 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-presentationDrawing.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.231 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-spreadsheetDrawing.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`vml-spreadsheetDrawing.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：5712 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-spreadsheetDrawing.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.232 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-wordprocessingDrawing.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`vml-wordprocessingDrawing.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：4010 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-wordprocessingDrawing.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.233 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/wml.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`wml.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：171367 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/wml.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.234 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/xml.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`
- **文件名**：`xml.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：4646 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/xml.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.235 `skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-contentTypes.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ecma/fouth-edition`
- **文件名**：`opc-contentTypes.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：1963 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-contentTypes.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.236 `skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-coreProperties.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ecma/fouth-edition`
- **文件名**：`opc-coreProperties.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：2515 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-coreProperties.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.237 `skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-digSig.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ecma/fouth-edition`
- **文件名**：`opc-digSig.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：2856 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-digSig.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.238 `skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-relationships.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/ecma/fouth-edition`
- **文件名**：`opc-relationships.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：1344 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-relationships.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.239 `skills/docx/scripts/office/schemas/mce/mc.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/mce`
- **文件名**：`mc.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：3127 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/mce/mc.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.240 `skills/docx/scripts/office/schemas/microsoft/wml-2010.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/microsoft`
- **文件名**：`wml-2010.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：26549 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/microsoft/wml-2010.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.241 `skills/docx/scripts/office/schemas/microsoft/wml-2012.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/microsoft`
- **文件名**：`wml-2012.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：3745 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/microsoft/wml-2012.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.242 `skills/docx/scripts/office/schemas/microsoft/wml-2018.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/microsoft`
- **文件名**：`wml-2018.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：901 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/microsoft/wml-2018.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.243 `skills/docx/scripts/office/schemas/microsoft/wml-cex-2018.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/microsoft`
- **文件名**：`wml-cex-2018.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：1778 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/microsoft/wml-cex-2018.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.244 `skills/docx/scripts/office/schemas/microsoft/wml-cid-2016.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/microsoft`
- **文件名**：`wml-cid-2016.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：1002 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/microsoft/wml-cid-2016.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.245 `skills/docx/scripts/office/schemas/microsoft/wml-sdtdatahash-2020.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/microsoft`
- **文件名**：`wml-sdtdatahash-2020.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：600 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/microsoft/wml-sdtdatahash-2020.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.246 `skills/docx/scripts/office/schemas/microsoft/wml-symex-2015.xsd`

- **所在目录**：`skills/docx/scripts/office/schemas/microsoft`
- **文件名**：`wml-symex-2015.xsd`
- **类型**：`.xsd`
- **行数**：0
- **大小**：745 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/schemas/microsoft/wml-symex-2015.xsd`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.247 `skills/docx/scripts/office/soffice.py`

- **所在目录**：`skills/docx/scripts/office`
- **文件名**：`soffice.py`
- **类型**：`.py`
- **行数**：192
- **大小**：5927 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/soffice.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.248 `skills/docx/scripts/office/validate.py`

- **所在目录**：`skills/docx/scripts/office`
- **文件名**：`validate.py`
- **类型**：`.py`
- **行数**：173
- **大小**：6142 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/validate.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.249 `skills/docx/scripts/office/validators/__init__.py`

- **所在目录**：`skills/docx/scripts/office/validators`
- **文件名**：`__init__.py`
- **类型**：`.py`
- **行数**：15
- **大小**：336 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/validators/__init__.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.250 `skills/docx/scripts/office/validators/base.py`

- **所在目录**：`skills/docx/scripts/office/validators`
- **文件名**：`base.py`
- **类型**：`.py`
- **行数**：875
- **大小**：33756 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/validators/base.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.251 `skills/docx/scripts/office/validators/docx.py`

- **所在目录**：`skills/docx/scripts/office/validators`
- **文件名**：`docx.py`
- **类型**：`.py`
- **行数**：466
- **大小**：17482 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/validators/docx.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.252 `skills/docx/scripts/office/validators/pptx.py`

- **所在目录**：`skills/docx/scripts/office/validators`
- **文件名**：`pptx.py`
- **类型**：`.py`
- **行数**：441
- **大小**：15966 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/validators/pptx.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.253 `skills/docx/scripts/office/validators/redlining.py`

- **所在目录**：`skills/docx/scripts/office/validators`
- **文件名**：`redlining.py`
- **类型**：`.py`
- **行数**：299
- **大小**：11231 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/office/validators/redlining.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.254 `skills/docx/scripts/templates/comments.xml`

- **所在目录**：`skills/docx/scripts/templates`
- **文件名**：`comments.xml`
- **类型**：`.xml`
- **行数**：0
- **大小**：2603 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/templates/comments.xml`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.255 `skills/docx/scripts/templates/commentsExtended.xml`

- **所在目录**：`skills/docx/scripts/templates`
- **文件名**：`commentsExtended.xml`
- **类型**：`.xml`
- **行数**：0
- **大小**：2611 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/templates/commentsExtended.xml`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.256 `skills/docx/scripts/templates/commentsExtensible.xml`

- **所在目录**：`skills/docx/scripts/templates`
- **文件名**：`commentsExtensible.xml`
- **类型**：`.xml`
- **行数**：0
- **大小**：2707 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/templates/commentsExtensible.xml`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.257 `skills/docx/scripts/templates/commentsIds.xml`

- **所在目录**：`skills/docx/scripts/templates`
- **文件名**：`commentsIds.xml`
- **类型**：`.xml`
- **行数**：0
- **大小**：2619 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/templates/commentsIds.xml`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.258 `skills/docx/scripts/templates/people.xml`

- **所在目录**：`skills/docx/scripts/templates`
- **文件名**：`people.xml`
- **类型**：`.xml`
- **行数**：0
- **大小**：115 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/scripts/templates/people.xml`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.259 `skills/docx/skill.md`

- **所在目录**：`skills/docx`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：91
- **大小**：6911 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/docx/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.260 `skills/email-draft/skill.md`

- **所在目录**：`skills/email-draft`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：74
- **大小**：3177 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/email-draft/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.261 `skills/excel-report/skill.md`

- **所在目录**：`skills/excel-report`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：47
- **大小**：1516 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/excel-report/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.262 `skills/feishu-doc/skill.md`

- **所在目录**：`skills/feishu-doc`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：80
- **大小**：4308 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/feishu-doc/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.263 `skills/financial-model/skill.md`

- **所在目录**：`skills/financial-model`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：77
- **大小**：3565 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/financial-model/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.264 `skills/html-page/skill.md`

- **所在目录**：`skills/html-page`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：121
- **大小**：9818 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/html-page/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.265 `skills/infographic/skill.md`

- **所在目录**：`skills/infographic`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：68
- **大小**：2533 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/infographic/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.266 `skills/lark-cli/skill.md`

- **所在目录**：`skills/lark-cli`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：107
- **大小**：5015 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/lark-cli/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.267 `skills/market-research/skill.md`

- **所在目录**：`skills/market-research`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：73
- **大小**：3291 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/market-research/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.268 `skills/meeting-minutes/skill.md`

- **所在目录**：`skills/meeting-minutes`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：74
- **大小**：3399 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/meeting-minutes/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.269 `skills/podcast/skill.md`

- **所在目录**：`skills/podcast`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：75
- **大小**：2822 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/podcast/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.270 `skills/ppt-design/skill.md`

- **所在目录**：`skills/ppt-design`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：128
- **大小**：6410 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/ppt-design/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.271 `skills/prd/skill.md`

- **所在目录**：`skills/prd`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：103
- **大小**：4437 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/prd/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.272 `skills/project-plan/skill.md`

- **所在目录**：`skills/project-plan`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：90
- **大小**：4280 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/project-plan/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.273 `skills/recruiting/skill.md`

- **所在目录**：`skills/recruiting`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：108
- **大小**：4261 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/recruiting/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.274 `skills/sales-outreach/skill.md`

- **所在目录**：`skills/sales-outreach`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：74
- **大小**：3218 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/sales-outreach/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.275 `skills/seo-brief/skill.md`

- **所在目录**：`skills/seo-brief`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：69
- **大小**：2798 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/seo-brief/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.276 `skills/short-drama/references/分镜表.schema.json`

- **所在目录**：`skills/short-drama/references`
- **文件名**：`分镜表.schema.json`
- **类型**：`.json`
- **行数**：85
- **大小**：5133 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/short-drama/references/分镜表.schema.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.277 `skills/short-drama/skill.md`

- **所在目录**：`skills/short-drama`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：167
- **大小**：15793 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/short-drama/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.278 `skills/skill-creator/skill.md`

- **所在目录**：`skills/skill-creator`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：32
- **大小**：1287 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/skill-creator/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.279 `skills/support-scripts/skill.md`

- **所在目录**：`skills/support-scripts`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：83
- **大小**：3643 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/support-scripts/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.280 `skills/video-compose/skill.md`

- **所在目录**：`skills/video-compose`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：31
- **大小**：3682 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/video-compose/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.281 `skills/web-styles/skill.md`

- **所在目录**：`skills/web-styles`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：124
- **大小**：7357 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/web-styles/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.282 `skills/wechat-article/scripts/format.py`

- **所在目录**：`skills/wechat-article/scripts`
- **文件名**：`format.py`
- **类型**：`.py`
- **行数**：254
- **大小**：9710 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/wechat-article/scripts/format.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.283 `skills/wechat-article/scripts/publish.py`

- **所在目录**：`skills/wechat-article/scripts`
- **文件名**：`publish.py`
- **类型**：`.py`
- **行数**：244
- **大小**：10165 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/wechat-article/scripts/publish.py`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.284 `skills/wechat-article/skill.md`

- **所在目录**：`skills/wechat-article`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：146
- **大小**：7165 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/wechat-article/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.285 `skills/wechat-article/themes/minimal.json`

- **所在目录**：`skills/wechat-article/themes`
- **文件名**：`minimal.json`
- **类型**：`.json`
- **行数**：149
- **大小**：3249 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/wechat-article/themes/minimal.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.286 `skills/wechat-article/themes/newspaper.json`

- **所在目录**：`skills/wechat-article/themes`
- **文件名**：`newspaper.json`
- **类型**：`.json`
- **行数**：151
- **大小**：3276 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/wechat-article/themes/newspaper.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.287 `skills/wechat-article/themes/tech.json`

- **所在目录**：`skills/wechat-article/themes`
- **文件名**：`tech.json`
- **类型**：`.json`
- **行数**：149
- **大小**：3239 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/wechat-article/themes/tech.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.288 `skills/wechat-article/themes/warm.json`

- **所在目录**：`skills/wechat-article/themes`
- **文件名**：`warm.json`
- **类型**：`.json`
- **行数**：149
- **大小**：3239 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/wechat-article/themes/warm.json`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.289 `skills/weekly-report/skill.md`

- **所在目录**：`skills/weekly-report`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：37
- **大小**：1150 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/weekly-report/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.290 `skills/xhs-cards/skill.md`

- **所在目录**：`skills/xhs-cards`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：43
- **大小**：3708 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/xhs-cards/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.291 `skills/xiaohongshu-topic/skill.md`

- **所在目录**：`skills/xiaohongshu-topic`
- **文件名**：`skill.md`
- **类型**：`.md`
- **行数**：126
- **大小**：7231 字节
- **主要功能**：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- **维护建议**：改此文件前先跑 `node --check skills/xiaohongshu-topic/skill.md`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.292 `static-compress.js`

- **所在目录**：`.`
- **文件名**：`static-compress.js`
- **类型**：`.js`
- **行数**：141
- **大小**：6623 字节
- **主要功能**：静态资源压缩。 导出或内部函数包括 `accepts`、`pickEncoding`、`compress`、`createStaticCompress`。
- **维护建议**：改此文件前先跑 `node --check static-compress.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.293 `store.js`

- **所在目录**：`.`
- **文件名**：`store.js`
- **类型**：`.js`
- **行数**：119
- **大小**：5547 字节
- **主要功能**：JSON 小仓库：原子写 + .bak 兜底 + 坏文件隔离；账本走 strict 模式绝不静默回滚。 导出或内部函数包括 `readJson`、`recover`、`tighten`、`writeJsonAtomic`。
- **维护建议**：改此文件前先跑 `node --check store.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.294 `sweep.js`

- **所在目录**：`.`
- **文件名**：`sweep.js`
- **类型**：`.js`
- **行数**：381
- **大小**：19041 字节
- **主要功能**：清中间物：只删本轮名单且路径必须在工作区内。 导出或内部函数包括 `isHidden`、`siteish`、`partOfSet`、`dirSize`、`scan`、`taskOf`、`taskOfDir`、`deliverablesOf`、`seqDirInfo`、`usageOf`、`plan`、`apply`。
- **维护建议**：改此文件前先跑 `node --check sweep.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.295 `systemone.js`

- **所在目录**：`.`
- **文件名**：`systemone.js`
- **类型**：`.js`
- **行数**：378
- **大小**：18700 字节
- **主要功能**：Jev 判断模型：是非/单选/打分与确定度门槛。 导出或内部函数包括 `isStr`、`txt`、`noul`、`choice`、`score`、`normalizeQuestions`、`stateOf`、`buildBody`、`sureOfNoul`、`probsOf`、`pct`、`readAnswers` 等共 19 个函数。
- **维护建议**：改此文件前先跑 `node --check systemone.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.296 `task-verdict.js`

- **所在目录**：`.`
- **文件名**：`task-verdict.js`
- **类型**：`.js`
- **行数**：263
- **大小**：16360 字节
- **主要功能**：假绿裁定：没抛异常不等于干成了。 导出或内部函数包括 `lastMatchIndex`、`judgeRun`、`explainRunError`、`verdictMessage`。
- **维护建议**：改此文件前先跑 `node --check task-verdict.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.297 `term-image.js`

- **所在目录**：`.`
- **文件名**：`term-image.js`
- **类型**：`.js`
- **行数**：178
- **大小**：9429 字节
- **主要功能**：终端画产出图。 导出或内部函数包括 `detect`、`encode`、`pickDrawable`、`openerFor`、`resolveTarget`。
- **维护建议**：改此文件前先跑 `node --check term-image.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.298 `test/admin-ui.js`

- **所在目录**：`test`
- **文件名**：`admin-ui.js`
- **类型**：`.js`
- **行数**：582
- **大小**：39458 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。 导出或内部函数包括 `call`、`openAdmin`、`ok`、`GOTO`。
- **维护建议**：改此文件前先跑 `node --check test/admin-ui.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.299 `test/agent-loop.js`

- **所在目录**：`test`
- **文件名**：`agent-loop.js`
- **类型**：`.js`
- **行数**：156
- **大小**：11131 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。 导出或内部函数包括 `ok`、`eq`、`key`、`hist`、`call`、`step`、`cycle`。
- **维护建议**：改此文件前先跑 `node --check test/agent-loop.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.300 `test/all.js`

- **所在目录**：`test`
- **文件名**：`all.js`
- **类型**：`.js`
- **行数**：126
- **大小**：8053 字节
- **主要功能**：测试调度器：按套件拉起子进程并汇总退出码。
- **维护建议**：改此文件前先跑 `node --check test/all.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.301 `test/auth-2fa.js`

- **所在目录**：`test`
- **文件名**：`auth-2fa.js`
- **类型**：`.js`
- **行数**：259
- **大小**：17157 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。 导出或内部函数包括 `ok`、`why`、`rewindStep`、`hit`、`ck`。
- **维护建议**：改此文件前先跑 `node --check test/auth-2fa.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.302 `test/cdp.js`

- **所在目录**：`test`
- **文件名**：`cdp.js`
- **类型**：`.js`
- **行数**：192
- **大小**：11850 字节
- **主要功能**：Chrome DevTools Protocol 真浏览器控制。 导出或内部函数包括 `fakeChrome`、`ok`、`eq`、`listen`、`wsFrame`。
- **维护建议**：改此文件前先跑 `node --check test/cdp.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.303 `test/chat-models.js`

- **所在目录**：`test`
- **文件名**：`chat-models.js`
- **类型**：`.js`
- **行数**：554
- **大小**：35712 字节
- **主要功能**：对话渠道表、选型、健康账本。 导出或内部函数包括 `ok`、`eq`、`legacy`。
- **维护建议**：改此文件前先跑 `node --check test/chat-models.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.304 `test/checkpoints.js`

- **所在目录**：`test`
- **文件名**：`checkpoints.js`
- **类型**：`.js`
- **行数**：155
- **大小**：10333 字节
- **主要功能**：文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。 导出或内部函数包括 `ok`、`read`、`rec`。
- **维护建议**：改此文件前先跑 `node --check test/checkpoints.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.305 `test/cli-approve.js`

- **所在目录**：`test`
- **文件名**：`cli-approve.js`
- **类型**：`.js`
- **行数**：194
- **大小**：10350 字节
- **主要功能**：终端/手机审批危险操作。 导出或内部函数包括 `fakeIO`、`run`。
- **维护建议**：改此文件前先跑 `node --check test/cli-approve.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.306 `test/cli-args.js`

- **所在目录**：`test`
- **文件名**：`cli-args.js`
- **类型**：`.js`
- **行数**：388
- **大小**：25237 字节
- **主要功能**：CLI 参数表。 导出或内部函数包括 `ok`、`eq`、`say`、`clean`。
- **维护建议**：改此文件前先跑 `node --check test/cli-args.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.307 `test/cli-ask.js`

- **所在目录**：`test`
- **文件名**：`cli-ask.js`
- **类型**：`.js`
- **行数**：123
- **大小**：6839 字节
- **主要功能**：终端回答 agent 提问。 导出或内部函数包括 `fakeIO`、`run`。
- **维护建议**：改此文件前先跑 `node --check test/cli-ask.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.308 `test/cli-attach.js`

- **所在目录**：`test`
- **文件名**：`cli-attach.js`
- **类型**：`.js`
- **行数**：469
- **大小**：32822 字节
- **主要功能**：CLI 带文件和图片。 导出或内部函数包括 `ok`、`eq`、`fakeFs`、`runner`、`P`。
- **维护建议**：改此文件前先跑 `node --check test/cli-attach.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.309 `test/cli-live.js`

- **所在目录**：`test`
- **文件名**：`cli-live.js`
- **类型**：`.js`
- **行数**：535
- **大小**：33581 字节
- **主要功能**：终端与网页直播桥。 导出或内部函数包括 `ok`、`eq`、`metaOf`、`writeMeta`、`rowOf`。
- **维护建议**：改此文件前先跑 `node --check test/cli-live.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.310 `test/deploy.js`

- **所在目录**：`test`
- **文件名**：`deploy.js`
- **类型**：`.js`
- **行数**：634
- **大小**：41512 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。 导出或内部函数包括 `ignored`、`done`、`ok`、`read`、`bare`。
- **维护建议**：改此文件前先跑 `node --check test/deploy.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.311 `test/doctor.js`

- **所在目录**：`test`
- **文件名**：`doctor.js`
- **类型**：`.js`
- **行数**：346
- **大小**：26275 字节
- **主要功能**：openworkbuddy doctor 体检。 导出或内部函数包括 `ok`、`eq`、`kt`。
- **维护建议**：改此文件前先跑 `node --check test/doctor.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.312 `test/e2e.js`

- **所在目录**：`test`
- **文件名**：`e2e.js`
- **类型**：`.js`
- **行数**：16703
- **大小**：1215207 字节
- **主要功能**：最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。 导出或内部函数包括 `reapStaleTempHomes`、`bootRealServer`、`makeFakeLLM`、`testAgentPipeline`、`testOfficeLibs`、`testPreviewExtract`、`testNodeSyntaxPrecheck`、`testShellGlobCompat`、`testMissingBinHintWired`、`testSessionFileLayout`、`testCssTokenGate`、`testMotionGate` 等共 173 个函数。
- **维护建议**：改此文件前先跑 `node --check test/e2e.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.313 `test/eval.js`

- **所在目录**：`test`
- **文件名**：`eval.js`
- **类型**：`.js`
- **行数**：231
- **大小**：14050 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。 导出或内部函数包括 `ok`、`src`、`fresh`。
- **维护建议**：改此文件前先跑 `node --check test/eval.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.314 `test/fixtures/mermaid.js`

- **所在目录**：`test/fixtures`
- **文件名**：`mermaid.js`
- **类型**：`.js`
- **行数**：124
- **大小**：4266 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- **维护建议**：改此文件前先跑 `node --check test/fixtures/mermaid.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.315 `test/frontend.js`

- **所在目录**：`test`
- **文件名**：`frontend.js`
- **类型**：`.js`
- **行数**：9874
- **大小**：738345 字节
- **主要功能**：真 Chromium 里跑前端源码切片断言。 导出或内部函数包括 `esc`、`fileIcon`、`fmtSize`、`revealFile`、`previewFile`、`startPreview`、`toast`、`onActivate`、`esc`、`canOpenOnHost`、`revealFile`、`renderFiles` 等共 65 个函数。
- **维护建议**：改此文件前先跑 `node --check test/frontend.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.316 `test/gen-cache.js`

- **所在目录**：`test`
- **文件名**：`gen-cache.js`
- **类型**：`.js`
- **行数**：380
- **大小**：24653 字节
- **主要功能**：生成结果内容寻址缓存，同一格重跑不二次扣费。 导出或内部函数包括 `ok`、`eq`、`e2e`、`finish`、`resolveFile`、`K`、`json`、`bin`。
- **维护建议**：改此文件前先跑 `node --check test/gen-cache.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.317 `test/icons.js`

- **所在目录**：`test`
- **文件名**：`icons.js`
- **类型**：`.js`
- **行数**：612
- **大小**：38126 字节
- **主要功能**：图标资源。 导出或内部函数包括 `hits`、`regexCanStart`、`stripComments`、`stripHtmlComments`、`stripEmojiRegions`、`scan`、`ok`、`eq`、`allHits`、`accepts`。
- **维护建议**：改此文件前先跑 `node --check test/icons.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.318 `test/lanes.js`

- **所在目录**：`test`
- **文件名**：`lanes.js`
- **类型**：`.js`
- **行数**：131
- **大小**：8991 字节
- **主要功能**：办公/工程两条泳道。 导出或内部函数包括 `ok`、`eq`、`backendId`。
- **维护建议**：改此文件前先跑 `node --check test/lanes.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.319 `test/lifecycle.js`

- **所在目录**：`test`
- **文件名**：`lifecycle.js`
- **类型**：`.js`
- **行数**：486
- **大小**：29369 字节
- **主要功能**：入职离职：一次关权限不删数据，出交接回执。 导出或内部函数包括 `newSched`、`call`、`ok`、`eq`、`throws`、`V`、`U`。
- **维护建议**：改此文件前先跑 `node --check test/lifecycle.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.320 `test/md-tty.js`

- **所在目录**：`test`
- **文件名**：`md-tty.js`
- **类型**：`.js`
- **行数**：207
- **大小**：12464 字节
- **主要功能**：终端 Markdown 渲染。 导出或内部函数包括 `ok`、`eq`、`plain`、`colored`、`streamed`。
- **维护建议**：改此文件前先跑 `node --check test/md-tty.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.321 `test/media-health.js`

- **所在目录**：`test`
- **文件名**：`media-health.js`
- **类型**：`.js`
- **行数**：212
- **大小**：14325 字节
- **主要功能**：媒体渠道熔断：连挂硬错后暂停，提示词里写明。 导出或内部函数包括 `eq`、`ok`、`err`、`fresh`。
- **维护建议**：改此文件前先跑 `node --check test/media-health.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.322 `test/media-models.js`

- **所在目录**：`test`
- **文件名**：`media-models.js`
- **类型**：`.js`
- **行数**：655
- **大小**：42557 字节
- **主要功能**：生图/生视频/配音/转写/看图多模型与五家视频协议识别。 导出或内部函数包括 `rest`、`asrChecks`、`done`、`ok`、`eq`、`legacy`。
- **维护建议**：改此文件前先跑 `node --check test/media-models.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.323 `test/office-tools.js`

- **所在目录**：`test`
- **文件名**：`office-tools.js`
- **类型**：`.js`
- **行数**：946
- **大小**：63706 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。 导出或内部函数包括 `clean`、`ok`、`eq`、`has`、`makeZip`、`makeFixtures`、`run`。
- **维护建议**：改此文件前先跑 `node --check test/office-tools.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.324 `test/ops.js`

- **所在目录**：`test`
- **文件名**：`ops.js`
- **类型**：`.js`
- **行数**：461
- **大小**：25601 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。 导出或内部函数包括 `ok`、`eq`、`readLog`、`wipeLog`。
- **维护建议**：改此文件前先跑 `node --check test/ops.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.325 `test/prefs.js`

- **所在目录**：`test`
- **文件名**：`prefs.js`
- **类型**：`.js`
- **行数**：1022
- **大小**：73352 字节
- **主要功能**：按账号存储的引擎、思考档、外观偏好。 导出或内部函数包括 `call`、`slice`、`runSeedCopy`、`runSourcePins`、`runConfigGates`、`ok`、`eq`、`makeConfig`、`flat`。
- **维护建议**：改此文件前先跑 `node --check test/prefs.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.326 `test/quota.js`

- **所在目录**：`test`
- **文件名**：`quota.js`
- **类型**：`.js`
- **行数**：264
- **大小**：14799 字节
- **主要功能**：付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。 导出或内部函数包括 `ok`、`eq`、`reset`、`table`、`actor`、`burn`、`ok_silent`。
- **维护建议**：改此文件前先跑 `node --check test/quota.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.327 `test/rbac.js`

- **所在目录**：`test`
- **文件名**：`rbac.js`
- **类型**：`.js`
- **行数**：321
- **大小**：23709 字节
- **主要功能**：RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。 导出或内部函数包括 `call`、`login`、`ok`、`eq`、`why`、`denied`、`allowed`、`raw`。
- **维护建议**：改此文件前先跑 `node --check test/rbac.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.328 `test/relay.js`

- **所在目录**：`test`
- **文件名**：`relay.js`
- **类型**：`.js`
- **行数**：959
- **大小**：66829 字节
- **主要功能**：OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。 导出或内部函数包括 `ok`、`eq`、`threw`、`bill`。
- **维护建议**：改此文件前先跑 `node --check test/relay.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.329 `test/remote.js`

- **所在目录**：`test`
- **文件名**：`remote.js`
- **类型**：`.js`
- **行数**：491
- **大小**：32878 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。 导出或内部函数包括 `req`、`ok`、`eq`、`ck`。
- **维护建议**：改此文件前先跑 `node --check test/remote.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.330 `test/repl-commands.js`

- **所在目录**：`test`
- **文件名**：`repl-commands.js`
- **类型**：`.js`
- **行数**：910
- **大小**：63699 字节
- **主要功能**：REPL 斜杠命令。 导出或内部函数包括 `ok`、`eq`、`tag`、`fakeInbox`、`main`。
- **维护建议**：改此文件前先跑 `node --check test/repl-commands.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.331 `test/repo-hygiene.js`

- **所在目录**：`test`
- **文件名**：`repo-hygiene.js`
- **类型**：`.js`
- **行数**：580
- **大小**：34167 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。 导出或内部函数包括 `stripComments`、`stripTemplates`、`assertStripSane`、`ignoredSkills`、`skillNameLiterals`、`shippedSkills`、`namingHits`、`scanTargets`、`DECLARED_DEPS`、`pkgOf`、`ok`、`HAS_GIT` 等共 14 个函数。
- **维护建议**：改此文件前先跑 `node --check test/repo-hygiene.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.332 `test/session-search.js`

- **所在目录**：`test`
- **文件名**：`session-search.js`
- **类型**：`.js`
- **行数**：260
- **大小**：16553 字节
- **主要功能**：任务历史检索：正文、产出文件名、意思相近。 导出或内部函数包括 `ok`、`eq`、`sess`、`one`、`titles`。
- **维护建议**：改此文件前先跑 `node --check test/session-search.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.333 `test/shot-history.js`

- **所在目录**：`test`
- **文件名**：`shot-history.js`
- **类型**：`.js`
- **行数**：305
- **大小**：19401 字节
- **主要功能**：分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。 导出或内部函数包括 `ok`、`freshBoard`、`abs`、`write`、`read`、`board`、`saveBoard`、`shotOf`。
- **维护建议**：改此文件前先跑 `node --check test/shot-history.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.334 `test/skill-guard.js`

- **所在目录**：`test`
- **文件名**：`skill-guard.js`
- **类型**：`.js`
- **行数**：433
- **大小**：29142 字节
- **主要功能**：技能静态安检：三十余条规则分三档，强装留档。 导出或内部函数包括 `ok`、`both`、`mkSrc`、`tryInstall`、`S`、`rules`、`hit`、`installedDir`。
- **维护建议**：改此文件前先跑 `node --check test/skill-guard.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.335 `test/sweep.js`

- **所在目录**：`test`
- **文件名**：`sweep.js`
- **类型**：`.js`
- **行数**：232
- **大小**：13363 字节
- **主要功能**：清中间物：只删本轮名单且路径必须在工作区内。 导出或内部函数包括 `ok`、`tree`、`fixture`、`group`、`pathsOf`、`pfx`、`frames`。
- **维护建议**：改此文件前先跑 `node --check test/sweep.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.336 `test/systemone.js`

- **所在目录**：`test`
- **文件名**：`systemone.js`
- **类型**：`.js`
- **行数**：367
- **大小**：28057 字节
- **主要功能**：Jev 判断模型：是非/单选/打分与确定度门槛。 导出或内部函数包括 `ok`、`eq`、`rest`、`src`。
- **维护建议**：改此文件前先跑 `node --check test/systemone.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.337 `test/tenant.js`

- **所在目录**：`test`
- **文件名**：`tenant.js`
- **类型**：`.js`
- **行数**：979
- **大小**：73472 字节
- **主要功能**：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。 导出或内部函数包括 `call`、`login`、`ok`、`eq`、`approvalScope`、`memScope`、`isPlatformOwner`。
- **维护建议**：改此文件前先跑 `node --check test/tenant.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.338 `test/term-image.js`

- **所在目录**：`test`
- **文件名**：`term-image.js`
- **类型**：`.js`
- **行数**：220
- **大小**：15577 字节
- **主要功能**：终端画产出图。 导出或内部函数包括 `ok`、`eq`。
- **维护建议**：改此文件前先跑 `node --check test/term-image.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.339 `test/toolward.js`

- **所在目录**：`test`
- **文件名**：`toolward.js`
- **类型**：`.js`
- **行数**：366
- **大小**：24177 字节
- **主要功能**：可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。 导出或内部函数包括 `ok`、`tryInstall`、`mkClean`、`F`、`REP`、`mkGuard`、`plan`、`calls`、`clearCalls`、`installedDir`。
- **维护建议**：改此文件前先跑 `node --check test/toolward.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.340 `test/totp.js`

- **所在目录**：`test`
- **文件名**：`totp.js`
- **类型**：`.js`
- **行数**：117
- **大小**：8372 字节
- **主要功能**：TOTP 二次验证（RFC 4226/6238）。 导出或内部函数包括 `ok`、`eq`。
- **维护建议**：改此文件前先跑 `node --check test/totp.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.341 `test/trace.js`

- **所在目录**：`test`
- **文件名**：`trace.js`
- **类型**：`.js`
- **行数**：750
- **大小**：49879 字节
- **主要功能**：本地 Trace 账本 + 可选 Langfuse 上报。 导出或内部函数包括 `fakeLangfuse`、`main`、`ok`、`eq`、`wait`。
- **维护建议**：改此文件前先跑 `node --check test/trace.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.342 `test/worktree.js`

- **所在目录**：`test`
- **文件名**：`worktree.js`
- **类型**：`.js`
- **行数**：251
- **大小**：16877 字节
- **主要功能**：两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。 导出或内部函数包括 `ok`、`mkRepo`、`eq`、`store`、`git`、`out`。
- **维护建议**：改此文件前先跑 `node --check test/worktree.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.343 `text-width.js`

- **所在目录**：`.`
- **文件名**：`text-width.js`
- **类型**：`.js`
- **行数**：19
- **大小**：863 字节
- **主要功能**：终端东西方字符宽度。 导出或内部函数包括 `cols`、`padCols`。
- **维护建议**：改此文件前先跑 `node --check text-width.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.344 `thinking.js`

- **所在目录**：`.`
- **文件名**：`thinking.js`
- **类型**：`.js`
- **行数**：155
- **大小**：8382 字节
- **主要功能**：思考档位到各家参数的映射。 导出或内部函数包括 `norm`、`planFor`、`planForEngine`、`isOn`。
- **维护建议**：改此文件前先跑 `node --check thinking.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.345 `thumb-png.js`

- **所在目录**：`.`
- **文件名**：`thumb-png.js`
- **类型**：`.js`
- **行数**：183
- **大小**：8473 字节
- **主要功能**：PNG 缩略图。 导出或内部函数包括 `crc32`、`readChunks`、`chunk`、`pngInfo`、`shrinkPng`。
- **维护建议**：改此文件前先跑 `node --check thumb-png.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.346 `thumb-worker.js`

- **所在目录**：`.`
- **文件名**：`thumb-worker.js`
- **类型**：`.js`
- **行数**：34
- **大小**：1591 字节
- **主要功能**：缩略图工作线程。
- **维护建议**：改此文件前先跑 `node --check thumb-worker.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.347 `thumb.js`

- **所在目录**：`.`
- **文件名**：`thumb.js`
- **类型**：`.js`
- **行数**：259
- **大小**：12066 字节
- **主要功能**：缩略图池。 导出或内部函数包括 `cacheName`、`makeThumb`、`thumbFile`、`hasNativeImage`、`retire`、`spawn`、`pump`、`enqueueShrink`、`thumbFileAsync`、`closeThumbPool`、`thumbPoolStats`。
- **维护建议**：改此文件前先跑 `node --check thumb.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.348 `tools.js`

- **所在目录**：`.`
- **文件名**：`tools.js`
- **类型**：`.js`
- **行数**：4197
- **大小**：249915 字节
- **主要功能**：内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写后自检与路径安全。 导出或内部函数包括 `ws`、`withWorkspace`、`enterWorkspace`、`getWorkspaceDir`、`orgPolicy`、`withPolicy`、`hostAllowed`、`orgBlocksShell`、`shellBlocked`、`netBlocked`、`getDefaultWorkspaceDir`、`setWorkspaceDir` 等共 134 个函数。
- **维护建议**：改此文件前先跑 `node --check tools.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.349 `toolward.js`

- **所在目录**：`.`
- **文件名**：`toolward.js`
- **类型**：`.js`
- **行数**：511
- **大小**：22649 字节
- **主要功能**：可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。 导出或内部函数包括 `candidateDirs`、`canRun`、`expandHome`、`findBin`、`run`、`modeOf`、`setting`、`probeBin`、`status`、`line`、`scanTargets`、`scanDir` 等共 25 个函数。
- **维护建议**：改此文件前先跑 `node --check toolward.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.350 `totp.js`

- **所在目录**：`.`
- **文件名**：`totp.js`
- **类型**：`.js`
- **行数**：129
- **大小**：5799 字节
- **主要功能**：TOTP 二次验证（RFC 4226/6238）。 文件头摘录：TOTP（RFC 6238）—— 二次验证的算术部分。 为什么自己写而不是装个库：这段东西一共就是「HMAC 一次、按最后 4 位取偏移、截 31 位、取模」， 加上一个 base32。全在 node 自带的 crypto 里，装一个包反而多一处供应链面。 但**自己写就必须拿 RFC 的标准向量对答案**——这类代码写错了不会报错， 只会变成「有时候能过有时候不能过」，用户会以为是手机时间不准，查上半天。 向量在 test/totp.js 里，RFC 4226 的 10 条 + RFC 6238 的 6 条，一条都不能少。 只做 SHA-1 / 6 位 / 30 秒这一档：Google Authenticator、1Password、微软那个， 认的都是这一档。otpauth:// 里就算写了 algorithm=SHA256，好几个 app 也直接忽略—— 于是你这边算 SHA256、 导出或内部函数包括 `base32Encode`、`base32Decode`、`hotp`、`generateSecret`、`code`、`verify`、`otpauthURL`。
- **维护建议**：改此文件前先跑 `node --check totp.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.351 `trace.js`

- **所在目录**：`.`
- **文件名**：`trace.js`
- **类型**：`.js`
- **行数**：707
- **大小**：37969 字节
- **主要功能**：本地 Trace 账本 + 可选 Langfuse 上报。 导出或内部函数包括 `localTraceFile`、`isFillerSaid`、`labelFromInput`、`archiveName`、`localFiles`、`rotateIfNeeded`、`localRecord`、`readEventsOf`、`localReadEvents`、`localTraceList`、`localTraceClear`、`capText` 等共 20 个函数。
- **维护建议**：改此文件前先跑 `node --check trace.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.352 `updater.js`

- **所在目录**：`.`
- **文件名**：`updater.js`
- **类型**：`.js`
- **行数**：101
- **大小**：5814 字节
- **主要功能**：检查 GitHub Releases 更新。 导出或内部函数包括 `currentVersion`、`installKind`、`parseVer`、`cmpVer`、`howToUpdate`、`checkUpdate`、`resetCache`。
- **维护建议**：改此文件前先跑 `node --check updater.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.353 `usage-store.js`

- **所在目录**：`.`
- **文件名**：`usage-store.js`
- **类型**：`.js`
- **行数**：252
- **大小**：11234 字节
- **主要功能**：分片用量账本。 导出或内部函数包括 `makeStore`。
- **维护建议**：改此文件前先跑 `node --check usage-store.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.354 `vkeys.js`

- **所在目录**：`.`
- **文件名**：`vkeys.js`
- **类型**：`.js`
- **行数**：284
- **大小**：12510 字节
- **主要功能**：虚拟 API Key 签发、校验、IP/模型/能力限制。 导出或内部函数包括 `hash`、`load`、`save`、`genRaw`、`maskOf`、`create`、`numOrZero`、`cleanList`、`cleanCaps`、`capAllowed`、`publicOf`、`list` 等共 23 个函数。
- **维护建议**：改此文件前先跑 `node --check vkeys.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

### 2.355 `worktree.js`

- **所在目录**：`.`
- **文件名**：`worktree.js`
- **类型**：`.js`
- **行数**：326
- **大小**：17908 字节
- **主要功能**：两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。 导出或内部函数包括 `git`、`realOf`、`repoOf`、`plan`、`seedFrom`、`open`、`sigOf`、`readMeta`、`status`、`list`、`close`、`release` 等共 20 个函数。
- **维护建议**：改此文件前先跑 `node --check worktree.js`（若为 JS）以及 `test/all.js` 中与该主题相关的套件；若文件属于 agent/tools/提示词，再跑 eval。不要把 API Key 示例写成真值。
- **与用户数据关系**：源码目录不应写入用户会话；通过 paths.js 定位 HOME。测试必须指向临时目录。

## 3. 按目录的源码说明

### 3.1 目录 `.`

含 118 个文件，合计约 57221 行。该目录在架构上的位置：运行时核心，Agent 与 HTTP 都在这里。是阅读和 Code Review 的主战场。

- `.dockerignore`（0 行）：项目支撑文件。
- `.gitignore`（0 行）：项目支撑文件。
- `CHANGELOG.en.md`（169 行）：项目支撑文件。
- `CHANGELOG.md`（167 行）：项目支撑文件。
- `COMMERCIAL-LICENSE.md`（106 行）：项目支撑文件。
- `CONTRIBUTING.md`（204 行）：项目支撑文件。
- `Dockerfile`（53 行）：项目支撑文件。
- `LICENSE`（0 行）：项目支撑文件。
- `LICENSE-ECOSYSTEM.md`（108 行）：项目支撑文件。
- `NOTICE.md`（62 行）：项目支撑文件。
- `README.en.md`（357 行）：项目支撑文件。
- `README.md`（366 行）：项目支撑文件。
- `account.js`（1959 行）：账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `admin.js`（815 行）：企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `agent.js`（2816 行）：Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果核验与死循环硬停。
- `awake.js`（129 行）：睡眠检测与 keep-awake，合盖顺延时限。
- `boot-check.js`（105 行）：启动自检。
- `browser-render.js`（138 行）：隐藏窗口渲染动态页。
- `budget.js`（328 行）：额度预扣与释放，防止并发超卖。
- `callout.js`（25 行）：正文提示条：网页画图标，终端/IM 换文字。
- `cdp.js`（353 行）：Chrome DevTools Protocol 真浏览器控制。
- `chat-models.js`（286 行）：对话渠道表、选型、健康账本。
- `checkpoints.js`（320 行）：文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `cli-approve.js`（141 行）：终端/手机审批危险操作。
- `cli-args.js`（403 行）：CLI 参数表。
- `cli-ask.js`（186 行）：终端回答 agent 提问。
- `cli-attach.js`（414 行）：CLI 带文件和图片。
- `cli-live.js`（399 行）：终端与网页直播桥。
- `cli.js`（2120 行）：openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair、worktree 
- `config-lint.js`（127 行）：配置体检：Key 形态、地址、互斥字段。
- `config-merge.js`（89 行）：配置深合并与热加载。
- `config.example.json`（87 行）：项目支撑文件。
- `deploy.sh`（187 行）：项目支撑文件。
- `diagram.js`（362 行）：四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `docker-compose.yml`（64 行）：项目支撑文件。
- `doctor.js`（463 行）：openworkbuddy doctor 体检。
- `drama-compose.js`（461 行）：短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `drama-pipeline.js`（208 行）：短剧进度与产出路径盘点。
- `electron-builder.config.js`（201 行）：项目支撑文件。
- `electron-main.js`（486 行）：Electron 主进程：起本地服务、开窗口、系统托盘、桌面宠物、睡眠阻断、隐藏窗口渲染。
- `evolve.js`（655 行）：自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `experts-lib.js`（84 行）：专家与专家团加载。
- `experts.json`（585 行）：项目支撑文件。
- `feishu-doc.js`（337 行）：飞书 convert API 写云文档。
- `gen-cache.js`（231 行）：生成结果内容寻址缓存，同一格重跑不二次扣费。
- `goal.js`（282 行）：Goal 目标模式：拆验收标准、机器实测、Jev 判断模型、最多三轮补跑。
- `htmlshot.js`（70 行）：HTML 离屏截图串行队列。
- `icons.js`（34 行）：图标资源。
- `im-ilink.js`（482 行）：微信 iLink 扫码登录通道。
- `im-media.js`（258 行）：IM 入站媒体（图片语音文件）落盘与转交 agent。
- `im-qq.js`（316 行）：QQ 机器人通道。
- `im-store.js`（133 行）：IM 会话独立落盘，重启不失忆。
- `im-wechat.js`（381 行）：微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `im.js`（1682 行）：IM 远程指挥总控：飞书、企微、钉钉、Webhook 入站出站、会话持久化、成果回传。
- `install-mac.sh`（95 行）：项目支撑文件。
- `install.sh`（93 行）：项目支撑文件。
- `intranet.js`（41 行）：内网探测与绑定。
- `jev.js`（166 行）：Jev 协议拼装。
- `json-compress.js`（108 行）：JSON 压缩。
- `lanes.js`（106 行）：办公/工程两条泳道。
- `lark-cli.js`（75 行）：飞书全家桶 CLI 绑定。
- `lifecycle.js`（291 行）：入职离职：一次关权限不删数据，出交接回执。
- `llm.js`（869 行）：多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、失败重试与缓存命中字
- `llms.txt`（36 行）：项目支撑文件。
- `log.js`（149 行）：日志。
- `mailer.js`（233 行）：SMTP 发信与收件人白名单。
- `mcp-catalog.js`（205 行）：连接器广场目录与探测。
- `mcp.js`（443 行）：MCP 客户端：stdio / Streamable HTTP，工具注入 agent，生命周期管理。
- `md-tty.js`（218 行）：终端 Markdown 渲染。
- `media-health.js`（154 行）：媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `media-models.js`（705 行）：生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `memory.js`（551 行）：双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、过期能力断言拦截、命
- `metrics.js`（291 行）：运维指标与 Prometheus 导出。
- `migrate.js`（208 行）：升级迁移。
- `modes.js`（76 行）：执行模式唯一真源：Ask / Plan / Craft / Goal，避免界面与 CLI 漂移。
- `notify.js`（51 行）：群机器人推送。
- `org.js`（490 行）：多租户组织、席位、套餐。
- `package-lock.json`（7076 行）：项目支撑文件。
- `package.json`（56 行）：项目支撑文件。
- `paths.js`（133 行）：数据目录解析：OPENWORKBUDDY_HOME / 打包后用户目录 / 源码旁 data。
- `pet-preload.js`（12 行）：宠物窗口预加载脚本。
- `pet-sprites.js`（229 行）：像素宠物精灵与 Codex/Petdex 兼容。
- `pet.js`（421 行）：桌面宠物窗口与状态机。
- `plugins.js`（449 行）：Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `prefs.js`（247 行）：按账号存储的引擎、思考档、外观偏好。
- `preview.js`（384 行）：docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `pricing.js`（455 行）：积分与单价表。
- `quota.js`（415 行）：付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `rbac.js`（112 行）：RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `relay-files.js`（162 行）：中转站生成文件暂存。
- `relay.js`（873 行）：OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `repl-commands.js`（739 行）：REPL 斜杠命令。
- `scheduler.js`（606 行）：定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `security.js`（631 行）：安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERNS、引擎档位翻译。
- `server.js`（7248 行）：Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被 Electron 
- `session-search.js`（199 行）：任务历史检索：正文、产出文件名、意思相近。
- `shot-history.js`（371 行）：分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `skill-guard.js`（580 行）：技能静态安检：三十余条规则分三档，强装留档。
- `skills.js`（758 行）：技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `static-compress.js`（141 行）：静态资源压缩。
- `store.js`（119 行）：JSON 小仓库：原子写 + .bak 兜底 + 坏文件隔离；账本走 strict 模式绝不静默回滚。
- `sweep.js`（381 行）：清中间物：只删本轮名单且路径必须在工作区内。
- `systemone.js`（378 行）：Jev 判断模型：是非/单选/打分与确定度门槛。
- `task-verdict.js`（263 行）：假绿裁定：没抛异常不等于干成了。
- `term-image.js`（178 行）：终端画产出图。
- `text-width.js`（19 行）：终端东西方字符宽度。
- `thinking.js`（155 行）：思考档位到各家参数的映射。
- `thumb-png.js`（183 行）：PNG 缩略图。
- `thumb-worker.js`（34 行）：缩略图工作线程。
- `thumb.js`（259 行）：缩略图池。
- `tools.js`（4197 行）：内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写后自检与路径安全。
- `toolward.js`（511 行）：可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `totp.js`（129 行）：TOTP 二次验证（RFC 4226/6238）。
- `trace.js`（707 行）：本地 Trace 账本 + 可选 Langfuse 上报。
- `updater.js`（101 行）：检查 GitHub Releases 更新。
- `usage-store.js`（252 行）：分片用量账本。
- `vkeys.js`（284 行）：虚拟 API Key 签发、校验、IP/模型/能力限制。
- `worktree.js`（326 行）：两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。

### 3.2 目录 `.github/workflows`

含 2 个文件，合计约 222 行。该目录在架构上的位置：支撑打包、部署或评测，不在热路径也要保持可重复。

- `.github/workflows/release.yml`（169 行）：CI：测试与发版工作流。
- `.github/workflows/test.yml`（53 行）：CI：测试与发版工作流。

### 3.3 目录 `build`

含 4 个文件，合计约 0 行。该目录在架构上的位置：支撑打包、部署或评测，不在热路径也要保持可重复。

- `build/icon.icns`（0 行）：项目支撑文件。
- `build/icon.ico`（0 行）：项目支撑文件。
- `build/icon.png`（0 行）：项目支撑文件。
- `build/icon.svg`（0 行）：项目支撑文件。

### 3.4 目录 `deploy`

含 4 个文件，合计约 394 行。该目录在架构上的位置：支撑打包、部署或评测，不在热路径也要保持可重复。

- `deploy/Caddyfile`（21 行）：部署配置：反向代理、环境变量示例、运维说明。
- `deploy/README.md`（373 行）：部署配置：反向代理、环境变量示例、运维说明。
- `deploy/env.example`（0 行）：部署配置：反向代理、环境变量示例、运维说明。
- `deploy/nginx.conf`（0 行）：部署配置：反向代理、环境变量示例、运维说明。

### 3.5 目录 `docs`

含 25 个文件，合计约 3490 行。该目录在架构上的位置：人话文档。与代码冲突时以代码为准并应修文档。

- `docs/Agent效率借鉴.md`（37 行）：产品与架构文档，给人读，不参与运行时。
- `docs/IM与定时任务.md`（63 行）：产品与架构文档，给人读，不参与运行时。
- `docs/index.html`（54 行）：产品与架构文档，给人读，不参与运行时。
- `docs/stats.json`（10 行）：产品与架构文档，给人读，不参与运行时。
- `docs/功能清单.md`（81 行）：产品与架构文档，给人读，不参与运行时。
- `docs/发版.md`（82 行）：产品与架构文档，给人读，不参与运行时。
- `docs/命令行用法.md`（408 行）：产品与架构文档，给人读，不参与运行时。
- `docs/多人协作.md`（150 行）：产品与架构文档，给人读，不参与运行时。
- `docs/安全.md`（79 行）：产品与架构文档，给人读，不参与运行时。
- `docs/安全基线.md`（205 行）：产品与架构文档，给人读，不参与运行时。
- `docs/安装与启动.md`（170 行）：产品与架构文档，给人读，不参与运行时。
- `docs/实现细节.md`（80 行）：产品与架构文档，给人读，不参与运行时。
- `docs/对标学习与服务器运行.md`（74 行）：产品与架构文档，给人读，不参与运行时。
- `docs/开源与商业版边界.md`（327 行）：产品与架构文档，给人读，不参与运行时。
- `docs/扩展.md`（163 行）：产品与架构文档，给人读，不参与运行时。
- `docs/数据同步与搬家.md`（181 行）：产品与架构文档，给人读，不参与运行时。
- `docs/架构优化路线_2026-08.md`（402 行）：产品与架构文档，给人读，不参与运行时。
- `docs/案例.md`（33 行）：产品与架构文档，给人读，不参与运行时。
- `docs/模型与Key管理_调研与改法.md`（240 行）：产品与架构文档，给人读，不参与运行时。
- `docs/短剧画布选型.md`（113 行）：产品与架构文档，给人读，不参与运行时。
- `docs/评测方法论.md`（101 行）：产品与架构文档，给人读，不参与运行时。
- `docs/路线图.md`（95 行）：产品与架构文档，给人读，不参与运行时。
- `docs/远程访问.md`（191 行）：产品与架构文档，给人读，不参与运行时。
- `docs/部署.md`（39 行）：产品与架构文档，给人读，不参与运行时。
- `docs/配置模型.md`（112 行）：产品与架构文档，给人读，不参与运行时。

### 3.6 目录 `docs/images`

含 14 个文件，合计约 0 行。该目录在架构上的位置：人话文档。与代码冲突时以代码为准并应修文档。

- `docs/images/case-canvas.jpg`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/case-hunan-site.jpg`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/case-photoreal.jpg`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/case-schedule-feishu.jpg`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/demo-canvas.gif`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/demo.gif`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/feishu-group.png`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/holo-card-ui.png`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/how-it-works.en.svg`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/how-it-works.svg`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/local-claude-code.png`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/openworkbuddy-overview.svg`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/short-drama-canvas-overview.png`（0 行）：产品与架构文档，给人读，不参与运行时。
- `docs/images/star-guide.svg`（0 行）：产品与架构文档，给人读，不参与运行时。

### 3.7 目录 `docs/推广`

含 1 个文件，合计约 249 行。该目录在架构上的位置：人话文档。与代码冲突时以代码为准并应修文档。

- `docs/推广/发布日文案.md`（249 行）：产品与架构文档，给人读，不参与运行时。

### 3.8 目录 `engines`

含 8 个文件，合计约 1691 行。该目录在架构上的位置：把外部 CLI 当发动机，工具经 MCP 回流才能走本仓库安全闸。档位翻译在 security.engineGuard。

- `engines/bridge.js`（159 行）：把本项目工具借给外部 CLI（MCP）。
- `engines/claude-code.js`（278 行）：本机 Claude Code 引擎适配。
- `engines/codex.js`（254 行）：本机 Codex 引擎适配。
- `engines/index.js`（196 行）：执行引擎选择：内置循环 / Claude Code / Codex。
- `engines/jsonl.js`（201 行）：引擎 JSONL 事件流解析。
- `engines/tool-bridge.js`（274 行）：外部引擎工具桥细节。
- `engines/which.js`（179 行）：探测本机引擎可执行文件。
- `engines/win.js`（150 行）：Windows 引擎路径。

### 3.9 目录 `eval`

含 3 个文件，合计约 992 行。该目录在架构上的位置：支撑打包、部署或评测，不在热路径也要保持可重复。

- `eval/baseline.json`（56 行）：评测题库与运行器。
- `eval/run.js`（416 行）：黑盒评测运行器：pass@k、AI 评委、基线对比。
- `eval/tasks.js`（520 行）：评测题库。

### 3.10 目录 `public`

含 5 个文件，合计约 3624 行。该目录在架构上的位置：用户看得见的界面。无构建，顺序依赖 script 标签，禁止改成乱序 ESM 除非先做依赖图审计。

- `public/admin.html`（379 行）：前端静态资源，无构建直接由 Express 提供。
- `public/icon.png`（0 行）：前端静态资源，无构建直接由 Express 提供。
- `public/index.html`（2737 行）：前端静态资源，无构建直接由 Express 提供。
- `public/pet.html`（343 行）：前端静态资源，无构建直接由 Express 提供。
- `public/svgfig.js`（165 行）：流式 SVG 清洗与主题内联。

### 3.11 目录 `public/css`

含 1 个文件，合计约 1217 行。该目录在架构上的位置：用户看得见的界面。无构建，顺序依赖 script 标签，禁止改成乱序 ESM 除非先做依赖图审计。

- `public/css/ui.css`（1217 行）：前端静态资源，无构建直接由 Express 提供。

### 3.12 目录 `public/js`

含 11 个文件，合计约 21886 行。该目录在架构上的位置：用户看得见的界面。无构建，顺序依赖 script 标签，禁止改成乱序 ESM 除非先做依赖图审计。

- `public/js/admin.js`（2717 行）：管理后台前端。
- `public/js/app-00-ui.js`（388 行）：前端 UI 基础：主题、字号、组件。
- `public/js/app-01.js`（4102 行）：前端会话、消息、文件面板。
- `public/js/app-02.js`（2529 行）：前端任务直播、过程卡、模式切换。
- `public/js/app-03.js`（1835 行）：前端设置与安全。
- `public/js/app-04.js`（1558 行）：前端专家技能连接器广场。
- `public/js/app-05.js`（3327 行）：前端模型表、媒体、Trace。
- `public/js/app-06.js`（932 行）：前端评测、自进化、备份。
- `public/js/app-07-canvas.js`（2462 行）：无限画布 JointJS 底座。
- `public/js/app-07-drama.js`（421 行）：短剧业务节点与生成回写。
- `public/js/i18n.js`（1615 行）：中英文案。

### 3.13 目录 `scripts`

含 12 个文件，合计约 1379 行。该目录在架构上的位置：支撑打包、部署或评测，不在热路径也要保持可重复。

- `scripts/check-package-files.js`（263 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/demo-mask.js`（56 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/demo-readme.js`（41 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/demo-timing.js`（35 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/genlogo.py`（201 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/make-icons.sh`（35 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/make-mac-app.sh`（115 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/record-demo.js`（303 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/run-app.sh`（13 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/shot-card.js`（84 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/shot-ui.js`（172 行）：维护脚本：统计、图标、演示录制、打包检查。
- `scripts/stats.js`（61 行）：维护脚本：统计、图标、演示录制、打包检查。

### 3.14 目录 `skills/character-photo-studio`

含 1 个文件，合计约 86 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/character-photo-studio/skill.md`（86 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.15 目录 `skills/competitor-watch`

含 1 个文件，合计约 71 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/competitor-watch/skill.md`（71 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.16 目录 `skills/content-repurpose`

含 1 个文件，合计约 59 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/content-repurpose/skill.md`（59 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.17 目录 `skills/content-studio`

含 1 个文件，合计约 33 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/content-studio/skill.md`（33 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.18 目录 `skills/contract-review`

含 1 个文件，合计约 86 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/contract-review/skill.md`（86 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.19 目录 `skills/customer-feedback`

含 1 个文件，合计约 67 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/customer-feedback/skill.md`（67 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.20 目录 `skills/data-analysis`

含 1 个文件，合计约 68 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/data-analysis/skill.md`（68 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.21 目录 `skills/data-viz`

含 1 个文件，合计约 64 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/data-viz/skill.md`（64 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.22 目录 `skills/deep-research`

含 1 个文件，合计约 66 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/deep-research/skill.md`（66 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.23 目录 `skills/docx`

含 2 个文件，合计约 121 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/docx/LICENSE.txt`（30 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/skill.md`（91 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.24 目录 `skills/docx/scripts`

含 4 个文件，合计约 814 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/docx/scripts/__init__.py`（1 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/accept_changes.py`（135 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/comment.py`（368 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/merge_runs.py`（310 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.25 目录 `skills/docx/scripts/office`

含 2 个文件，合计约 365 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/docx/scripts/office/soffice.py`（192 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/validate.py`（173 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.26 目录 `skills/docx/scripts/office/helpers`

含 4 个文件，合计约 455 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/docx/scripts/office/helpers/__init__.py`（111 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/helpers/pptx_chart.py`（170 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/helpers/pptx_slide.py`（60 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/helpers/pptx_theme.py`（114 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.27 目录 `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016`

含 27 个文件，合计约 0 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-chart.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-chartDrawing.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-diagram.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-lockedCanvas.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-main.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-picture.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-spreadsheetDrawing.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/dml-wordprocessingDrawing.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/pml.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-additionalCharacteristics.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-bibliography.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-commonSimpleTypes.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-customXmlDataProperties.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-customXmlSchemaProperties.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-documentPropertiesCustom.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-documentPropertiesExtended.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-documentPropertiesVariantTypes.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-math.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/shared-relationshipReference.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/sml.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-main.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-officeDrawing.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-presentationDrawing.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-spreadsheetDrawing.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/vml-wordprocessingDrawing.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/wml.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ISO-IEC29500-4_2016/xml.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.28 目录 `skills/docx/scripts/office/schemas/ecma/fouth-edition`

含 4 个文件，合计约 0 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-contentTypes.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-coreProperties.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-digSig.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/ecma/fouth-edition/opc-relationships.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.29 目录 `skills/docx/scripts/office/schemas/mce`

含 1 个文件，合计约 0 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/docx/scripts/office/schemas/mce/mc.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.30 目录 `skills/docx/scripts/office/schemas/microsoft`

含 7 个文件，合计约 0 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/docx/scripts/office/schemas/microsoft/wml-2010.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/microsoft/wml-2012.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/microsoft/wml-2018.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/microsoft/wml-cex-2018.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/microsoft/wml-cid-2016.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/microsoft/wml-sdtdatahash-2020.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/schemas/microsoft/wml-symex-2015.xsd`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.31 目录 `skills/docx/scripts/office/validators`

含 5 个文件，合计约 2096 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/docx/scripts/office/validators/__init__.py`（15 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/validators/base.py`（875 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/validators/docx.py`（466 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/validators/pptx.py`（441 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/office/validators/redlining.py`（299 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.32 目录 `skills/docx/scripts/templates`

含 5 个文件，合计约 0 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/docx/scripts/templates/comments.xml`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/templates/commentsExtended.xml`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/templates/commentsExtensible.xml`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/templates/commentsIds.xml`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/docx/scripts/templates/people.xml`（0 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.33 目录 `skills/email-draft`

含 1 个文件，合计约 74 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/email-draft/skill.md`（74 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.34 目录 `skills/excel-report`

含 1 个文件，合计约 47 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/excel-report/skill.md`（47 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.35 目录 `skills/feishu-doc`

含 1 个文件，合计约 80 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/feishu-doc/skill.md`（80 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.36 目录 `skills/financial-model`

含 1 个文件，合计约 77 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/financial-model/skill.md`（77 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.37 目录 `skills/html-page`

含 1 个文件，合计约 121 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/html-page/skill.md`（121 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.38 目录 `skills/infographic`

含 1 个文件，合计约 68 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/infographic/skill.md`（68 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.39 目录 `skills/lark-cli`

含 1 个文件，合计约 107 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/lark-cli/skill.md`（107 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.40 目录 `skills/market-research`

含 1 个文件，合计约 73 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/market-research/skill.md`（73 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.41 目录 `skills/meeting-minutes`

含 1 个文件，合计约 74 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/meeting-minutes/skill.md`（74 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.42 目录 `skills/podcast`

含 1 个文件，合计约 75 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/podcast/skill.md`（75 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.43 目录 `skills/ppt-design`

含 1 个文件，合计约 128 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/ppt-design/skill.md`（128 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.44 目录 `skills/prd`

含 1 个文件，合计约 103 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/prd/skill.md`（103 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.45 目录 `skills/project-plan`

含 1 个文件，合计约 90 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/project-plan/skill.md`（90 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.46 目录 `skills/recruiting`

含 1 个文件，合计约 108 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/recruiting/skill.md`（108 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.47 目录 `skills/sales-outreach`

含 1 个文件，合计约 74 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/sales-outreach/skill.md`（74 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.48 目录 `skills/seo-brief`

含 1 个文件，合计约 69 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/seo-brief/skill.md`（69 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.49 目录 `skills/short-drama`

含 1 个文件，合计约 167 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/short-drama/skill.md`（167 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.50 目录 `skills/short-drama/references`

含 1 个文件，合计约 85 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/short-drama/references/分镜表.schema.json`（85 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.51 目录 `skills/skill-creator`

含 1 个文件，合计约 32 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/skill-creator/skill.md`（32 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.52 目录 `skills/support-scripts`

含 1 个文件，合计约 83 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/support-scripts/skill.md`（83 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.53 目录 `skills/video-compose`

含 1 个文件，合计约 31 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/video-compose/skill.md`（31 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.54 目录 `skills/web-styles`

含 1 个文件，合计约 124 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/web-styles/skill.md`（124 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.55 目录 `skills/wechat-article`

含 1 个文件，合计约 146 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/wechat-article/skill.md`（146 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.56 目录 `skills/wechat-article/scripts`

含 2 个文件，合计约 498 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/wechat-article/scripts/format.py`（254 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/wechat-article/scripts/publish.py`（244 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.57 目录 `skills/wechat-article/themes`

含 4 个文件，合计约 598 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/wechat-article/themes/minimal.json`（149 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/wechat-article/themes/newspaper.json`（151 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/wechat-article/themes/tech.json`（149 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。
- `skills/wechat-article/themes/warm.json`（149 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.58 目录 `skills/weekly-report`

含 1 个文件，合计约 37 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/weekly-report/skill.md`（37 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.59 目录 `skills/xhs-cards`

含 1 个文件，合计约 43 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/xhs-cards/skill.md`（43 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.60 目录 `skills/xiaohongshu-topic`

含 1 个文件，合计约 126 行。该目录在架构上的位置：给模型看的说明书加少量脚本。安装前静态扫描。作者体验必须保持「存盘即生效」。

- `skills/xiaohongshu-topic/skill.md`（126 行）：技能资源：Markdown 指令、脚本或模板，装入后由 agent 按 skill.md 执行。

### 3.61 目录 `test`

含 44 个文件，合计约 44438 行。该目录在架构上的位置：回归网。e2e 最重。新行为没有负对照等于没钉住。

- `test/admin-ui.js`（582 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `test/agent-loop.js`（156 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `test/all.js`（126 行）：测试调度器：按套件拉起子进程并汇总退出码。
- `test/auth-2fa.js`（259 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `test/cdp.js`（192 行）：Chrome DevTools Protocol 真浏览器控制。
- `test/chat-models.js`（554 行）：对话渠道表、选型、健康账本。
- `test/checkpoints.js`（155 行）：文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `test/cli-approve.js`（194 行）：终端/手机审批危险操作。
- `test/cli-args.js`（388 行）：CLI 参数表。
- `test/cli-ask.js`（123 行）：终端回答 agent 提问。
- `test/cli-attach.js`（469 行）：CLI 带文件和图片。
- `test/cli-live.js`（535 行）：终端与网页直播桥。
- `test/deploy.js`（634 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `test/doctor.js`（346 行）：openworkbuddy doctor 体检。
- `test/e2e.js`（16703 行）：最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `test/eval.js`（231 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `test/frontend.js`（9874 行）：真 Chromium 里跑前端源码切片断言。
- `test/gen-cache.js`（380 行）：生成结果内容寻址缓存，同一格重跑不二次扣费。
- `test/icons.js`（612 行）：图标资源。
- `test/lanes.js`（131 行）：办公/工程两条泳道。
- `test/lifecycle.js`（486 行）：入职离职：一次关权限不删数据，出交接回执。
- `test/md-tty.js`（207 行）：终端 Markdown 渲染。
- `test/media-health.js`（212 行）：媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `test/media-models.js`（655 行）：生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `test/office-tools.js`（946 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `test/ops.js`（461 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `test/prefs.js`（1022 行）：按账号存储的引擎、思考档、外观偏好。
- `test/quota.js`（264 行）：付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `test/rbac.js`（321 行）：RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `test/relay.js`（959 行）：OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `test/remote.js`（491 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `test/repl-commands.js`（910 行）：REPL 斜杠命令。
- `test/repo-hygiene.js`（580 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `test/session-search.js`（260 行）：任务历史检索：正文、产出文件名、意思相近。
- `test/shot-history.js`（305 行）：分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `test/skill-guard.js`（433 行）：技能静态安检：三十余条规则分三档，强装留档。
- `test/sweep.js`（232 行）：清中间物：只删本轮名单且路径必须在工作区内。
- `test/systemone.js`（367 行）：Jev 判断模型：是非/单选/打分与确定度门槛。
- `test/tenant.js`（979 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `test/term-image.js`（220 行）：终端画产出图。
- `test/toolward.js`（366 行）：可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `test/totp.js`（117 行）：TOTP 二次验证（RFC 4226/6238）。
- `test/trace.js`（750 行）：本地 Trace 账本 + 可选 Langfuse 上报。
- `test/worktree.js`（251 行）：两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。

### 3.62 目录 `test/fixtures`

含 1 个文件，合计约 124 行。该目录在架构上的位置：回归网。e2e 最重。新行为没有负对照等于没钉住。

- `test/fixtures/mermaid.js`（124 行）：自动化测试套件。失败退出码非 0，由 test/all.js 汇总。

## 4. 函数总索引（扫描自 JS）

格式：`函数` — 文件:行 — 模块职责一句。用于 Code Review 检索。

- `aesKeyOf` — `im-wechat.js:26` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `pkcs7Strip` — `im-wechat.js:32` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `pkcs7Pad` — `im-wechat.js:38` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `msgSignature` — `im-wechat.js:44` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `decryptMsg` — `im-wechat.js:50` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `encryptMsg` — `im-wechat.js:60` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `xmlField` — `im-wechat.js:75` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `splitBytes` — `im-wechat.js:81` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `mediaKindOf` — `im-wechat.js:99` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `uploadWechatMedia` — `im-wechat.js:108` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `makeTokenCache` — `im-wechat.js:123` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `mediaFields` — `im-wechat.js:137` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `createWecomApp` — `im-wechat.js:150` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `createWechatMp` — `im-wechat.js:260` — 微信公众号 / iLink 通道：AES 加解密、消息验签、被动回复与主动推送。
- `payloadOf` — `drama-compose.js:61` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `baseOf` — `drama-compose.js:62` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `isBlank` — `drama-compose.js:63` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `round` — `drama-compose.js:64` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `safeName` — `drama-compose.js:66` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `orderKeyOf` — `drama-compose.js:78` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `sortShots` — `drama-compose.js:91` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `pickFile` — `drama-compose.js:103` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `freeName` — `drama-compose.js:116` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `srtTime` — `drama-compose.js:123` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `buildSrt` — `drama-compose.js:135` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `musicPick` — `drama-compose.js:161` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `musicArgv` — `drama-compose.js:200` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `composePlan` — `drama-compose.js:248` — 短剧拼片计划：分镜排序、字幕、配乐、ffmpeg 参数。
- `pngSize` — `pet-sprites.js:74` — 像素宠物精灵与 Codex/Petdex 兼容。
- `webpSize` — `pet-sprites.js:89` — 像素宠物精灵与 Codex/Petdex 兼容。
- `imageSize` — `pet-sprites.js:108` — 像素宠物精灵与 Codex/Petdex 兼容。
- `checkSheet` — `pet-sprites.js:123` — 像素宠物精灵与 Codex/Petdex 兼容。
- `localRoot` — `pet-sprites.js:138` — 像素宠物精灵与 Codex/Petdex 兼容。
- `petRoots` — `pet-sprites.js:146` — 像素宠物精灵与 Codex/Petdex 兼容。
- `readPetDir` — `pet-sprites.js:160` — 像素宠物精灵与 Codex/Petdex 兼容。
- `scanPets` — `pet-sprites.js:182` — 像素宠物精灵与 Codex/Petdex 兼容。
- `findPet` — `pet-sprites.js:201` — 像素宠物精灵与 Codex/Petdex 兼容。
- `sheetDataUrl` — `pet-sprites.js:207` — 像素宠物精灵与 Codex/Petdex 兼容。
- `spriteSpec` — `pet-sprites.js:214` — 像素宠物精灵与 Codex/Petdex 兼容。
- `available` — `browser-render.js:14` — 隐藏窗口渲染动态页。
- `withHiddenWindow` — `browser-render.js:24` — 隐藏窗口渲染动态页。
- `loadHtml` — `browser-render.js:41` — 隐藏窗口渲染动态页。
- `renderMermaid` — `browser-render.js:85` — 隐藏窗口渲染动态页。
- `svgSize` — `browser-render.js:103` — 隐藏窗口渲染动态页。
- `svgToPng` — `browser-render.js:122` — 隐藏窗口渲染动态页。
- `delay` — `browser-render.js:52` — 隐藏窗口渲染动态页。
- `reassignTo` — `lifecycle.js:36` — 入职离职：一次关权限不删数据，出交接回执。
- `offboard` — `lifecycle.js:56` — 入职离职：一次关权限不删数据，出交接回执。
- `onboard` — `lifecycle.js:198` — 入职离职：一次关权限不删数据，出交接回执。
- `deptTemplate` — `lifecycle.js:214` — 入职离职：一次关权限不删数据，出交接回执。
- `setDeptTemplate` — `lifecycle.js:227` — 入职离职：一次关权限不删数据，出交接回执。
- `listDeptTemplates` — `lifecycle.js:255` — 入职离职：一次关权限不删数据，出交接回执。
- `receiptText` — `lifecycle.js:263` — 入职离职：一次关权限不删数据，出交接回执。
- `sniffExt` — `im-media.js:26` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `safeBaseName` — `im-media.js:47` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `stamp` — `im-media.js:60` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `defaultName` — `im-media.js:67` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `saveInbound` — `im-media.js:75` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `fetchBuffer` — `im-media.js:105` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `fileNameFromHeaders` — `im-media.js:124` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `parseAesKey` — `im-media.js:138` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `decryptAesEcb` — `im-media.js:148` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `buildCdnDownloadUrl` — `im-media.js:153` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `downloadWechatCdn` — `im-media.js:158` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `encryptAesEcb` — `im-media.js:173` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `aesEcbPaddedSize` — `im-media.js:179` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `buildCdnUploadUrl` — `im-media.js:183` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `uploadCdnCiphertext` — `im-media.js:191` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `inboundNote` — `im-media.js:223` — IM 入站媒体（图片语音文件）落盘与转交 agent。
- `find` — `repl-commands.js:62` — REPL 斜杠命令。
- `nearest` — `repl-commands.js:73` — REPL 斜杠命令。
- `parse` — `repl-commands.js:95` — REPL 斜杠命令。
- `mergePaste` — `repl-commands.js:133` — REPL 斜杠命令。
- `resolveCd` — `repl-commands.js:150` — REPL 斜杠命令。
- `complete` — `repl-commands.js:160` — REPL 斜杠命令。
- `menu` — `repl-commands.js:191` — REPL 斜杠命令。
- `modelRows` — `repl-commands.js:234` — REPL 斜杠命令。
- `modelListText` — `repl-commands.js:262` — REPL 斜杠命令。
- `pickModelRow` — `repl-commands.js:287` — REPL 斜杠命令。
- `ago` — `repl-commands.js:319` — REPL 斜杠命令。
- `sessionRows` — `repl-commands.js:341` — REPL 斜杠命令。
- `sessionListText` — `repl-commands.js:357` — REPL 斜杠命令。
- `pickSessionRow` — `repl-commands.js:383` — REPL 斜杠命令。
- `pickerRowsOf` — `repl-commands.js:417` — REPL 斜杠命令。
- `filterPickerRows` — `repl-commands.js:426` — REPL 斜杠命令。
- `pickerWindow` — `repl-commands.js:442` — REPL 斜杠命令。
- `pickerView` — `repl-commands.js:452` — REPL 斜杠命令。
- `sessionPickerRows` — `repl-commands.js:482` — REPL 斜杠命令。
- `modelPickerRows` — `repl-commands.js:495` — REPL 斜杠命令。
- `initTask` — `repl-commands.js:510` — REPL 斜杠命令。
- `sizeText` — `repl-commands.js:528` — REPL 斜杠命令。
- `changedFilesText` — `repl-commands.js:540` — REPL 斜杠命令。
- `checkpointListText` — `repl-commands.js:559` — REPL 斜杠命令。
- `pickCheckpoint` — `repl-commands.js:575` — REPL 斜杠命令。
- `rewindResultText` — `repl-commands.js:583` — REPL 斜杠命令。
- `mcpText` — `repl-commands.js:595` — REPL 斜杠命令。
- `compactedText` — `repl-commands.js:613` — REPL 斜杠命令。
- `helpText` — `repl-commands.js:621` — REPL 斜杠命令。
- `unknownText` — `repl-commands.js:641` — REPL 斜杠命令。
- `badArgText` — `repl-commands.js:648` — REPL 斜杠命令。
- `makeInbox` — `repl-commands.js:662` — REPL 斜杠命令。
- `sanitizeHistory` — `repl-commands.js:718` — REPL 斜杠命令。
- `workspaceIndex` — `agent.js:235` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `sizeOf` — `agent.js:260` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `missingDeliverables` — `agent.js:269` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `unseenVisualClaims` — `agent.js:315` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `unfinishedMilestones` — `agent.js:338` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `entryChars` — `agent.js:368` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `historyChars` — `agent.js:376` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `dropRendererParams` — `agent.js:397` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `hasRenderer` — `agent.js:408` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `trimHistory` — `agent.js:422` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `envToday` — `agent.js:462` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `safeWorkspaceDir` — `agent.js:469` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `worktreeLine` — `agent.js:478` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `activeChannel` — `agent.js:487` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `stopNotice` — `agent.js:501` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `findCycle` — `agent.js:520` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `deadLoop` — `agent.js:539` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `pausedMediaBlock` — `agent.js:550` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `createAgentRuntime` — `agent.js:561` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `langBlock` — `agent.js:813` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `modePrompt` — `agent.js:818` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `splitParallelRuns` — `agent.js:2487` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `mapPool` — `agent.js:2504` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `tailText` — `agent.js:2534` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `toolHeadline` — `agent.js:2544` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `resultOutcome` — `agent.js:2618` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `previewInput` — `agent.js:2637` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `collectSources` — `agent.js:2653` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `makeFilesEmitter` — `agent.js:2728` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `makeOwnership` — `agent.js:2777` — Agent 核心运行时。Web、IM、专家委派、定时任务共用同一套步循环、自动续跑、子代理递归、成果
- `isHidden` — `sweep.js:63` — 清中间物：只删本轮名单且路径必须在工作区内。
- `siteish` — `sweep.js:69` — 清中间物：只删本轮名单且路径必须在工作区内。
- `partOfSet` — `sweep.js:74` — 清中间物：只删本轮名单且路径必须在工作区内。
- `dirSize` — `sweep.js:87` — 清中间物：只删本轮名单且路径必须在工作区内。
- `scan` — `sweep.js:112` — 清中间物：只删本轮名单且路径必须在工作区内。
- `taskOf` — `sweep.js:155` — 清中间物：只删本轮名单且路径必须在工作区内。
- `taskOfDir` — `sweep.js:157` — 清中间物：只删本轮名单且路径必须在工作区内。
- `deliverablesOf` — `sweep.js:163` — 清中间物：只删本轮名单且路径必须在工作区内。
- `seqDirInfo` — `sweep.js:180` — 清中间物：只删本轮名单且路径必须在工作区内。
- `usageOf` — `sweep.js:219` — 清中间物：只删本轮名单且路径必须在工作区内。
- `plan` — `sweep.js:251` — 清中间物：只删本轮名单且路径必须在工作区内。
- `apply` — `sweep.js:356` — 清中间物：只删本轮名单且路径必须在工作区内。
- `describeCron` — `scheduler.js:37` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `setActiveScheduler` — `scheduler.js:71` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `activeScheduler` — `scheduler.js:75` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `loadStore` — `scheduler.js:79` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `saveStore` — `scheduler.js:91` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `parseField` — `scheduler.js:102` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `parseAt` — `scheduler.js:151` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `describeWhen` — `scheduler.js:186` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `parseCron` — `scheduler.js:198` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `cronMatches` — `scheduler.js:215` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `createScheduler` — `scheduler.js:236` — 定时任务：cron / at 解析、错过补跑、结果推 IM、运行记录、与 agent 运行时对接。
- `looksSecret` — `memory.js:49` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `looksStaleClaim` — `memory.js:70` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `noteUsed` — `memory.js:89` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `flushHits` — `memory.js:97` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `keepScore` — `memory.js:125` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `normalize` — `memory.js:136` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `load` — `memory.js:143` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `save` — `memory.js:149` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `setEmbedder` — `memory.js:158` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `vecLoad` — `memory.js:160` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `vecSave` — `memory.js:164` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `ensureVectors` — `memory.js:174` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `bigrams` — `memory.js:214` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `keywordScore` — `memory.js:220` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `cosine` — `memory.js:228` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `scopeOf` — `memory.js:235` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `add` — `memory.js:246` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `mostSimilar` — `memory.js:307` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `dedupeForPrompt` — `memory.js:337` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `judgeableGrams` — `memory.js:372` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `vectorStatus` — `memory.js:381` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `remove` — `memory.js:400` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `forget` — `memory.js:414` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `renameScope` — `memory.js:428` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `list` — `memory.js:438` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `manual` — `memory.js:445` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `saveManual` — `memory.js:453` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `promptBlock` — `memory.js:464` — 双层长期记忆：手写区 memory.md 与条目区 memories.json；向量召回、密钥拦截、
- `pickBootLog` — `electron-main.js:20` — Electron 主进程：起本地服务、开窗口、系统托盘、桌面宠物、睡眠阻断、隐藏窗口渲染。
- `bootLog` — `electron-main.js:37` — Electron 主进程：起本地服务、开窗口、系统托盘、桌面宠物、睡眠阻断、隐藏窗口渲染。
- `fatal` — `electron-main.js:73` — Electron 主进程：起本地服务、开窗口、系统托盘、桌面宠物、睡眠阻断、隐藏窗口渲染。
- `waitForServer` — `electron-main.js:170` — Electron 主进程：起本地服务、开窗口、系统托盘、桌面宠物、睡眠阻断、隐藏窗口渲染。
- `attachContextMenu` — `electron-main.js:357` — Electron 主进程：起本地服务、开窗口、系统托盘、桌面宠物、睡眠阻断、隐藏窗口渲染。
- `bootHint` — `electron-main.js:384` — Electron 主进程：起本地服务、开窗口、系统托盘、桌面宠物、睡眠阻断、隐藏窗口渲染。
- `showBootFailure` — `electron-main.js:410` — Electron 主进程：起本地服务、开窗口、系统托盘、桌面宠物、睡眠阻断、隐藏窗口渲染。
- `registerShortcuts` — `electron-main.js:445` — Electron 主进程：起本地服务、开窗口、系统托盘、桌面宠物、睡眠阻断、隐藏窗口渲染。
- `ws` — `tools.js:38` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `withWorkspace` — `tools.js:42` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `enterWorkspace` — `tools.js:56` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `getWorkspaceDir` — `tools.js:60` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `orgPolicy` — `tools.js:72` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `withPolicy` — `tools.js:75` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `hostAllowed` — `tools.js:84` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `orgBlocksShell` — `tools.js:102` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `shellBlocked` — `tools.js:106` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `netBlocked` — `tools.js:113` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `getDefaultWorkspaceDir` — `tools.js:122` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `setWorkspaceDir` — `tools.js:126` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `tmpDir` — `tools.js:132` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `ensureDirs` — `tools.js:136` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `safePath` — `tools.js:142` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `safePathIn` — `tools.js:158` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `stampOnce` — `tools.js:654` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `safeOutName` — `tools.js:664` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `fetchRetry` — `tools.js:679` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `mediaKey` — `tools.js:702` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `downloadToWorkspace` — `tools.js:706` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `readBigFile` — `tools.js:725` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `shrinkForVision` — `tools.js:771` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `readImageInput` — `tools.js:802` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `imageDataUri` — `tools.js:818` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `lookAtImage` — `tools.js:830` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `savedAt` — `tools.js:938` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `postWantClean` — `tools.js:957` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `refImageUris` — `tools.js:981` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `generateImage` — `tools.js:1002` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `generateVideo` — `tools.js:1067` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `htmlToImage` — `tools.js:1269` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `textToSpeech` — `tools.js:1295` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `srtTime` — `tools.js:1353` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `transcribeAudio` — `tools.js:1366` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `precheckSyntax` — `tools.js:1456` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `pruneOldOutLogs` — `tools.js:1496` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `makeOutSink` — `tools.js:1507` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `runNode` — `tools.js:1563` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `shellPath` — `tools.js:1609` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `pickShell` — `tools.js:1617` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `missingBinHint` — `tools.js:1659` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `runShell` — `tools.js:1678` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `cleanLibRel` — `tools.js:1727` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `setLibraryDir` — `tools.js:1731` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `getLibraryDir` — `tools.js:1735` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `withLibraryDir` — `tools.js:1739` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `libRoot` — `tools.js:1743` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `libResolve` — `tools.js:1751` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `libraryList` — `tools.js:1768` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `libraryRead` — `tools.js:1819` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `libraryImport` — `tools.js:1852` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `pdfHowTo` — `tools.js:1898` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `docToText` — `tools.js:1916` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `slidesToText` — `tools.js:1934` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `sheetsToText` — `tools.js:1948` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `archiveToText` — `tools.js:1973` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `readDocument` — `tools.js:1985` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `listFiles` — `tools.js:2019` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `dirInsteadOfFile` — `tools.js:2064` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `keepBackup` — `tools.js:2081` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `readBefore` — `tools.js:2103` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `looksText` — `tools.js:2113` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `textOf` — `tools.js:2119` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `diffText` — `tools.js:2126` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `noteChange` — `tools.js:2139` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `countAll` — `tools.js:2145` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `looseLineMatch` — `tools.js:2164` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `shiftIndent` — `tools.js:2183` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `missHint` — `tools.js:2209` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `planEdit` — `tools.js:2246` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `readSource` — `tools.js:2289` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `editFile` — `tools.js:2296` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `selfCheck` — `tools.js:2325` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `undefinedCssVars` — `tools.js:2449` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `orphanSvgStyleScopes` — `tools.js:2481` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `auditHtml` — `tools.js:2495` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `closeHiddenWindow` — `tools.js:2576` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `isRuntimeNoise` — `tools.js:2591` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `readConsoleEvent` — `tools.js:2604` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `cleanConsoleText` — `tools.js:2615` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `checkPage` — `tools.js:2620` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `searchFiles` — `tools.js:2717` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `browserHeaders` — `tools.js:2826` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `looksEmptyPage` — `tools.js:2842` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `looksBinary` — `tools.js:2857` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `saveDownload` — `tools.js:2873` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `decodeBody` — `tools.js:2903` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `fetchUrl` — `tools.js:2920` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `pageTitle` — `tools.js:2998` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `mainRegion` — `tools.js:3013` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `htmlToText` — `tools.js:3024` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `tagsToText` — `tools.js:3033` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `renderPage` — `tools.js:3055` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `stripTags` — `tools.js:3090` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `jinaSearch` — `tools.js:3095` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `tavilySearch` — `tools.js:3105` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `braveSearch` — `tools.js:3117` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `searchProviderKey` — `tools.js:3130` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `webSearch` — `tools.js:3138` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `nearestTool` — `tools.js:3232` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `badToolArgs` — `tools.js:3266` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `withGenCache` — `tools.js:3294` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `unitsFor` — `tools.js:3348` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `mediaProviderOf` — `tools.js:3362` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `asrModelOf` — `tools.js:3366` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `quotaGate` — `tools.js:3381` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `viaMedia` — `tools.js:3401` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasSafeName` — `tools.js:3437` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasCurrentPath` — `tools.js:3442` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasCurrentName` — `tools.js:3443` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasSetCurrentName` — `tools.js:3446` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasStatePath` — `tools.js:3450` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasEmptyState` — `tools.js:3454` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasNormalizeState` — `tools.js:3472` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasBackup` — `tools.js:3505` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasReadState` — `tools.js:3518` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasWriteState` — `tools.js:3531` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasList` — `tools.js:3541` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `canvasManage` — `tools.js:3555` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `executeTool` — `tools.js:3602` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `workspaceKeyOf` — `tools.js:4040` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `workspaceKey` — `tools.js:4043` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `filesScope` — `tools.js:4048` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `noteUserInput` — `tools.js:4092` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `moveUserInput` — `tools.js:4100` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `isUserInput` — `tools.js:4110` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `outputFiles` — `tools.js:4115` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `fileDigest` — `tools.js:4166` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `markDuplicates` — `tools.js:4178` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `retryableStatus` — `tools.js:678` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `runsText` — `tools.js:1914` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `clampWait` — `tools.js:2851` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `userInputKey` — `tools.js:4090` — 内置工具定义与执行器：读写文件、跑命令、抓网页、生图生视频配音、画图、记忆、资料库、画布管理等；含写
- `fingerprint` — `media-health.js:44` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `statusOf` — `media-health.js:55` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `looksNetwork` — `media-health.js:61` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `looksBroke` — `media-health.js:72` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `looksNoModel` — `media-health.js:76` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `now` — `media-health.js:80` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `gate` — `media-health.js:86` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `record` — `media-health.js:107` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `reset` — `media-health.js:138` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `list` — `media-health.js:144` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `candidateDirs` — `toolward.js:84` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `canRun` — `toolward.js:103` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `expandHome` — `toolward.js:111` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `findBin` — `toolward.js:121` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `run` — `toolward.js:142` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `modeOf` — `toolward.js:164` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `setting` — `toolward.js:186` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `probeBin` — `toolward.js:203` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `status` — `toolward.js:222` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `line` — `toolward.js:243` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `scanTargets` — `toolward.js:255` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `scanDir` — `toolward.js:281` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `scanText` — `toolward.js:289` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `scanConnectors` — `toolward.js:314` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `redactServers` — `toolward.js:352` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `safeUrl` — `toolward.js:370` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `toReport` — `toolward.js:380` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `whyOf` — `toolward.js:438` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `merge` — `toolward.js:451` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `mkTemp` — `toolward.js:474` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `rmrf` — `toolward.js:478` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `relTo` — `toolward.js:482` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `clip` — `toolward.js:492` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `short` — `toolward.js:497` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `_reset` — `toolward.js:502` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `typeOf` — `relay-files.js:45` — 中转站生成文件暂存。
- `ensure` — `relay-files.js:49` — 中转站生成文件暂存。
- `readMeta` — `relay-files.js:59` — 中转站生成文件暂存。
- `save` — `relay-files.js:69` — 中转站生成文件暂存。
- `get` — `relay-files.js:97` — 中转站生成文件暂存。
- `remove` — `relay-files.js:109` — 中转站生成文件暂存。
- `sweep` — `relay-files.js:120` — 中转站生成文件暂存。
- `list` — `relay-files.js:146` — 中转站生成文件暂存。
- `binOf` — `relay-files.js:54` — 中转站生成文件暂存。
- `metaOf` — `relay-files.js:55` — 中转站生成文件暂存。
- `okId` — `relay-files.js:57` — 中转站生成文件暂存。
- `roleOf` — `rbac.js:67` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `rankOf` — `rbac.js:71` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `can` — `rbac.js:74` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `outranks` — `rbac.js:78` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `assignProblem` — `rbac.js:90` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `upper` — `rbac.js:102` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `assignableBy` — `rbac.js:108` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `same` — `rbac.js:100` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `parseConfigShow` — `lark-cli.js:13` — 飞书全家桶 CLI 绑定。
- `verifyUrlOf` — `lark-cli.js:32` — 飞书全家桶 CLI 绑定。
- `explainLarkError` — `lark-cli.js:39` — 飞书全家桶 CLI 绑定。
- `usableSecret` — `lark-cli.js:57` — 飞书全家桶 CLI 绑定。
- `appConsoleUrl` — `lark-cli.js:69` — 飞书全家桶 CLI 绑定。
- `randomUin` — `im-ilink.js:64` — 微信 iLink 扫码登录通道。
- `splitText` — `im-ilink.js:68` — 微信 iLink 扫码登录通道。
- `hasCdn` — `im-ilink.js:87` — 微信 iLink 扫码登录通道。
- `describeItems` — `im-ilink.js:99` — 微信 iLink 扫码登录通道。
- `dedupKey` — `im-ilink.js:135` — 微信 iLink 扫码登录通道。
- `fetchQrcode` — `im-ilink.js:144` — 微信 iLink 扫码登录通道。
- `pollQrStatus` — `im-ilink.js:157` — 微信 iLink 扫码登录通道。
- `createIlinkConnection` — `im-ilink.js:193` — 微信 iLink 扫码登录通道。
- `loadSkills` — `skills.js:27` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `safePluginSkills` — `skills.js:50` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `safeName` — `skills.js:61` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `parseFrontmatter` — `skills.js:67` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `findSkillDir` — `skills.js:92` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `getSkillFull` — `skills.js:104` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `assertNotPluginSkill` — `skills.js:117` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `samePlace` — `skills.js:124` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `saveSkill` — `skills.js:130` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `deleteSkill` — `skills.js:172` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `copySkillFolder` — `skills.js:189` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `dirSize` — `skills.js:219` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `discoverSkillDirs` — `skills.js:235` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `gate` — `skills.js:272` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `writeProvenance` — `skills.js:299` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `inspectSkillDir` — `skills.js:324` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `installedFromDir` — `skills.js:338` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `installFromGitHub` — `skills.js:353` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `adaptLibraryAsSkill` — `skills.js:428` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `cloneRepo` — `skills.js:448` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `defaultSkillUrl` — `skills.js:671` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `listDefaultSkills` — `skills.js:677` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `defaultInstallOpts` — `skills.js:703` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `installDefaultSkill` — `skills.js:714` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `ensureDefaultSkills` — `skills.js:726` — 技能加载器：扫描 skills/<名>/skill.md，热更新，装默认技能，安装前体检。
- `insideRoot` — `checkpoints.js:36` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `relOf` — `checkpoints.js:44` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `putObject` — `checkpoints.js:52` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `readObject` — `checkpoints.js:67` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `readLedger` — `checkpoints.js:72` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `appendLedger` — `checkpoints.js:83` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `record` — `checkpoints.js:95` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `fileHash` — `checkpoints.js:124` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `list` — `checkpoints.js:129` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `rewind` — `checkpoints.js:146` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `gc` — `checkpoints.js:184` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `splitLines` — `checkpoints.js:216` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `diffLines` — `checkpoints.js:225` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `unifiedDiff` — `checkpoints.js:262` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `summarize` — `checkpoints.js:313` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `sha` — `checkpoints.js:31` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `objPath` — `checkpoints.js:32` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `ledgerPath` — `checkpoints.js:33` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `today` — `log.js:35` — 日志。
- `fileOf` — `log.js:39` — 日志。
- `ensureDir` — `log.js:55` — 日志。
- `pruneOld` — `log.js:60` — 日志。
- `log` — `log.js:82` — 日志。
- `tail` — `log.js:115` — 日志。
- `days` — `log.js:135` — 日志。
- `setLevel` — `log.js:143` — 日志。
- `debug` — `log.js:104` — 日志。
- `info` — `log.js:105` — 日志。
- `warn` — `log.js:106` — 日志。
- `error` — `log.js:107` — 日志。
- `lastMatchIndex` — `task-verdict.js:137` — 假绿裁定：没抛异常不等于干成了。
- `judgeRun` — `task-verdict.js:153` — 假绿裁定：没抛异常不等于干成了。
- `explainRunError` — `task-verdict.js:248` — 假绿裁定：没抛异常不等于干成了。
- `verdictMessage` — `task-verdict.js:258` — 假绿裁定：没抛异常不等于干成了。
- `bootProblem` — `boot-check.js:35` — 启动自检。
- `findMissing` — `boot-check.js:73` — 启动自检。
- `enforce` — `boot-check.js:88` — 启动自检。
- `isBinaryName` — `skill-guard.js:327` — 技能静态安检：三十余条规则分三档，强装留档。
- `visible` — `skill-guard.js:330` — 技能静态安检：三十余条规则分三档，强装留档。
- `locate` — `skill-guard.js:336` — 技能静态安检：三十余条规则分三档，强装留档。
- `negated` — `skill-guard.js:353` — 技能静态安检：三十余条规则分三档，强装留档。
- `frontmatterEnd` — `skill-guard.js:365` — 技能静态安检：三十余条规则分三档，强装留档。
- `scanText` — `skill-guard.js:374` — 技能静态安检：三十余条规则分三档，强装留档。
- `taint` — `skill-guard.js:423` — 技能静态安检：三十余条规则分三档，强装留档。
- `scanDir` — `skill-guard.js:449` — 技能静态安检：三十余条规则分三档，强装留档。
- `scanOne` — `skill-guard.js:541` — 技能静态安检：三十余条规则分三档，强装留档。
- `explain` — `skill-guard.js:557` — 技能静态安检：三十余条规则分三档，强装留档。
- `mtimeOf` — `config-merge.js:17` — 配置深合并与热加载。
- `snapshot` — `config-merge.js:26` — 配置深合并与热加载。
- `changedPaths` — `config-merge.js:44` — 配置深合并与热加载。
- `applyAt` — `config-merge.js:65` — 配置深合并与热加载。
- `mergeOnto` — `config-merge.js:83` — 配置深合并与热加载。
- `isPlain` — `config-merge.js:34` — 配置深合并与热加载。
- `fillDefaults` — `server.js:139` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `llmForSession` — `server.js:230` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `recordModelHealth` — `server.js:249` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `healthSummary` — `server.js:258` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `saveExperts` — `server.js:289` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `persistRunning` — `server.js:305` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `sweepInterruptedRuns` — `server.js:308` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `assignSessionDir` — `server.js:332` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `sessFile` — `server.js:365` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `sessStat` — `server.js:370` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `sessChangedOnDisk` — `server.js:378` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `getSession` — `server.js:384` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `saveSession` — `server.js:407` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `loadSessIndex` — `server.js:440` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `saveSessIndex` — `server.js:454` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `listSessionsOnDisk` — `server.js:470` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `sessionRow` — `server.js:500` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `ownSession` — `server.js:525` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `autosaveSession` — `server.js:534` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `decideForGoal` — `server.js:574` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `addUsage` — `server.js:598` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `goalThink` — `server.js:614` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `petSay` — `server.js:640` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `recordingEmit` — `server.js:662` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `sessionAllowed` — `server.js:954` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `guardSession` — `server.js:961` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `guardRun` — `server.js:976` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `assetKindOf` — `server.js:1033` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `assetRoleOf` — `server.js:1038` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `assetBase` — `server.js:1046` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `assetRefsIn` — `server.js:1049` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `composeFilterSet` — `server.js:1186` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `composeBins` — `server.js:1200` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `composeProbe` — `server.js:1216` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `composeBuildPlan` — `server.js:1242` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `composeRun` — `server.js:1272` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `composeDropPartial` — `server.js:1290` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `composeExecute` — `server.js:1300` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `composeWriteBack` — `server.js:1364` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `composeView` — `server.js:1382` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `dramaName` — `server.js:1486` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `dramaSummary` — `server.js:1491` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `readDramaJson` — `server.js:1500` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `dramaTargetIds` — `server.js:1674` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `dramaSnapshotDiff` — `server.js:1681` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `shotHistoryArgs` — `server.js:1698` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `maskMedia` — `server.js:1943` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `ownPrefs` — `server.js:2077` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `savePersonalPrefs` — `server.js:2081` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `isLocalModel` — `server.js:2555` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `hasKey` — `server.js:2558` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `auditKeyChanges` — `server.js:2573` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `keyHint` — `server.js:2603` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `probeModel` — `server.js:2611` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `normalizeMcpServer` — `server.js:3318` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `projectMeta` — `server.js:3410` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `activeProject` — `server.js:3424` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `projectContextOf` — `server.js:3433` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `warnOnce` — `server.js:3473` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `clampMemo` — `server.js:3492` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `ensureProjects` — `server.js:3506` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `applyProjectSpaces` — `server.js:3521` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `saveConfig` — `server.js:3542` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `mergeDiskEdits` — `server.js:3551` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `readNotes` — `server.js:3695` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `writeNotes` — `server.js:3699` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `libPath` — `server.js:3713` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `libRel` — `server.js:3721` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `libFolders` — `server.js:3727` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `libCount` — `server.js:3740` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `sessionOutputRow` — `server.js:3893` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `listTaskOutputs` — `server.js:3931` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `statLookup` — `server.js:3967` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `searchRead` — `server.js:4080` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `searchExcerpt` — `server.js:4092` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `libWalk` — `server.js:4109` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `openWithSystem` — `server.js:4343` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `larkRun` — `server.js:4356` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `larkJson` — `server.js:4366` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `larkConfig` — `server.js:4369` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `adoptLarkCreds` — `server.js:4378` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `appUserDataDir` — `server.js:4568` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `dirSize` — `server.js:4577` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `cacheTmpDirs` — `server.js:4591` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `cacheStats` — `server.js:4596` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `userSkillEntries` — `server.js:4623` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `backupAllowed` — `server.js:4635` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `listBackups` — `server.js:4641` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `makeBackup` — `server.js:4655` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `backupFile` — `server.js:4678` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `inspectBackup` — `server.js:4699` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `memoryImportSources` — `server.js:4850` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `imAssign` — `server.js:4956` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `startPluginMcp` — `server.js:5093` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `publicExpert` — `server.js:5138` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `petPhotoPath` — `server.js:5234` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `clearPetPhoto` — `server.js:5241` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `listEmptyTaskDirs` — `server.js:5431` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `hostFileOf` — `server.js:5527` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `relOf` — `server.js:5578` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `mediaMime` — `server.js:5581` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `rememberRoot` — `server.js:5609` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `knownRoots` — `server.js:5619` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `rootFromKey` — `server.js:5631` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `tenantRootOf` — `server.js:5649` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `rootedPath` — `server.js:5658` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `saveDirOf` — `server.js:5786` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `decodeSaveBody` — `server.js:5792` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `saveErrorText` — `server.js:5801` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `electronDialog` — `server.js:5812` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `lanAddress` — `server.js:5885` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `previewState` — `server.js:5894` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `loadSessSearchIdx` — `server.js:6448` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `saveSessSearchIdx` — `server.js:6456` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `sessSearchIndex` — `server.js:6466` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `setSessEmbedder` — `server.js:6494` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `ensureSessVectors` — `server.js:6496` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `queryVector` — `server.js:6526` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `evalSummaryBrief` — `server.js:6593` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `evalHistory` — `server.js:6622` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `schedViewer` — `server.js:6811` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `assistModelOf` — `server.js:6882` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `accountedRuntime` — `server.js:6889` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `main` — `server.js:6940` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `tellShell` — `server.js:7139` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `portHeldByUs` — `server.js:7157` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `listenWithFallback` — `server.js:7184` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `bootFailed` — `server.js:7229` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `goalThinkFor` — `server.js:634` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `enterprise` — `server.js:817` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `isPlatformOwner` — `server.js:887` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `canRemoteControl` — `server.js:894` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `cliOffReason` — `server.js:896` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `traceOwnerOnly` — `server.js:2473` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `opsOwnerOnly` — `server.js:2503` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `approvalScope` — `server.js:2838` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `memScope` — `server.js:4220` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `actorOf` — `server.js:4971` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `skillErr` — `server.js:4978` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `cardAvatar` — `server.js:5137` — Express HTTP 服务入口：会话、设置、评测、技能、记忆、画布、安全、备份等全部 API；被
- `ttlMsFor` — `account.js:44` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `readStore` — `account.js:59` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `writeStoreAtomic` — `account.js:69` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `loadUsers` — `account.js:73` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `openRegister` — `account.js:83` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `creditsEnabled` — `account.js:92` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `saveUsers` — `account.js:95` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `loadUsage` — `account.js:108` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `saveUsage` — `account.js:112` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `localDay` — `account.js:115` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `hashPassword` — `account.js:121` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `passwordProblem` — `account.js:156` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `isRun` — `account.js:177` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `assertPassword` — `account.js:189` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `genPassword` — `account.js:199` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `publicUser` — `account.js:221` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `monthKey` — `account.js:257` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `monthlyQuotaOf` — `account.js:260` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `monthlyLeft` — `account.js:266` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `balanceOf` — `account.js:274` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `normalizeAvatar` — `account.js:283` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `hasUsers` — `account.js:302` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `twoFactorOn` — `account.js:313` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `hashRecovery` — `account.js:319` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `makeRecoveryCodes` — `account.js:322` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `startEnroll` — `account.js:333` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `enableTOTP` — `account.js:348` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `consumeTwoFactor` — `account.js:378` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `disableTOTP` — `account.js:402` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `regenRecovery` — `account.js:423` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `twoFactorStatus` — `account.js:440` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `defaultUser` — `account.js:451` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `register` — `account.js:456` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `renameUser` — `account.js:502` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `verify` — `account.js:525` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `issueToken` — `account.js:539` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `revokeTokens` — `account.js:567` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `prunePairs` — `account.js:607` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `newPairCode` — `account.js:612` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `dropPairCode` — `account.js:623` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `normalizePairCode` — `account.js:627` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `claimPair` — `account.js:631` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `pairStatus` — `account.js:648` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `pairOrigin` — `account.js:660` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `deviceLabel` — `account.js:677` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `deviceId` — `account.js:693` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `listDevices` — `account.js:696` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `revokeDevice` — `account.js:716` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `touchDevice` — `account.js:729` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `tokenFromReq` — `account.js:744` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `tokenKind` — `account.js:749` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `userFromReq` — `account.js:755` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `isHttps` — `account.js:767` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `setTokenCookie` — `account.js:770` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `clearTokenCookie` — `account.js:778` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `createLimiter` — `account.js:790` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `isPrivateAddr` — `account.js:822` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `trustedHops` — `account.js:847` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `clientIp` — `account.js:850` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `creditsFor` — `account.js:867` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `chargeRun` — `account.js:880` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `topup` — `account.js:960` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `fixLegacyCache` — `account.js:979` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `usageSummary` — `account.js:999` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `groupUsage` — `account.js:1079` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `firstAdminIsOwner` — `account.js:1102` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `syncOwner` — `account.js:1108` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `isAdmin` — `account.js:1115` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `canAdmin` — `account.js:1119` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `platformOwner` — `account.js:1123` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `rankFor` — `account.js:1135` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `assertCanManage` — `account.js:1150` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `assertNotLastOwner` — `account.js:1167` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `assertManageable` — `account.js:1174` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `listMembers` — `account.js:1182` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `billingUser` — `account.js:1201` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `setMember` — `account.js:1210` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `resetPassword` — `account.js:1262` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `resetPasswordLocally` — `account.js:1293` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `removeMember` — `account.js:1310` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `createMember` — `account.js:1324` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `transferOwner` — `account.js:1354` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `setOwnerLocally` — `account.js:1386` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `migrateOwners` — `account.js:1410` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `pendingMembers` — `account.js:1441` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `remoteAllowed` — `account.js:1460` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `authGuard` — `account.js:1469` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `adminOnly` — `account.js:1513` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `adminGuard` — `account.js:1518` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `migrateLegacySettings` — `account.js:1533` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `createRouter` — `account.js:1545` — 账号、会话令牌、用量账本、积分、入职离职、邀请码、二次验证接线、管理员守卫。
- `detect` — `term-image.js:48` — 终端画产出图。
- `encode` — `term-image.js:95` — 终端画产出图。
- `pickDrawable` — `term-image.js:120` — 终端画产出图。
- `openerFor` — `term-image.js:133` — 终端画产出图。
- `resolveTarget` — `term-image.js:157` — 终端画产出图。
- `hash` — `vkeys.js:41` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `load` — `vkeys.js:43` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `save` — `vkeys.js:49` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `genRaw` — `vkeys.js:51` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `maskOf` — `vkeys.js:59` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `create` — `vkeys.js:67` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `numOrZero` — `vkeys.js:97` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `cleanList` — `vkeys.js:101` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `cleanCaps` — `vkeys.js:116` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `capAllowed` — `vkeys.js:124` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `publicOf` — `vkeys.js:130` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `list` — `vkeys.js:135` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `verify` — `vkeys.js:149` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `modelAllowed` — `vkeys.js:173` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `ipAllowed` — `vkeys.js:182` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `inCidr` — `vkeys.js:192` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `v4` — `vkeys.js:201` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `localDay` — `vkeys.js:213` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `update` — `vkeys.js:219` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `revoke` — `vkeys.js:242` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `remove` — `vkeys.js:253` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `touch` — `vkeys.js:264` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `ofUser` — `vkeys.js:276` — 虚拟 API Key 签发、校验、IP/模型/能力限制。
- `parseJsonLoose` — `goal.js:30` — Goal 目标模式：拆验收标准、机器实测、Jev 判断模型、最多三轮补跑。
- `createGoalEngine` — `goal.js:41` — Goal 目标模式：拆验收标准、机器实测、Jev 判断模型、最多三轮补跑。
- `base32Encode` — `totp.js:24` — TOTP 二次验证（RFC 4226/6238）。
- `base32Decode` — `totp.js:38` — TOTP 二次验证（RFC 4226/6238）。
- `hotp` — `totp.js:56` — TOTP 二次验证（RFC 4226/6238）。
- `generateSecret` — `totp.js:73` — TOTP 二次验证（RFC 4226/6238）。
- `code` — `totp.js:78` — TOTP 二次验证（RFC 4226/6238）。
- `verify` — `totp.js:95` — TOTP 二次验证（RFC 4226/6238）。
- `otpauthURL` — `totp.js:123` — TOTP 二次验证（RFC 4226/6238）。
- `safeCut` — `md-tty.js:38` — 终端 Markdown 渲染。
- `inline` — `md-tty.js:78` — 终端 Markdown 渲染。
- `undecided` — `md-tty.js:102` — 终端 Markdown 渲染。
- `blockOf` — `md-tty.js:116` — 终端 Markdown 渲染。
- `createRenderer` — `md-tty.js:136` — 终端 Markdown 渲染。
- `configured` — `mailer.js:31` — SMTP 发信与收件人白名单。
- `fromAddr` — `mailer.js:37` — SMTP 发信与收件人白名单。
- `parseAddrs` — `mailer.js:48` — SMTP 发信与收件人白名单。
- `allowRules` — `mailer.js:72` — SMTP 发信与收件人白名单。
- `addrAllowed` — `mailer.js:84` — SMTP 发信与收件人白名单。
- `checkRecipients` — `mailer.js:103` — SMTP 发信与收件人白名单。
- `checkAttachments` — `mailer.js:114` — SMTP 发信与收件人白名单。
- `fmtBytes` — `mailer.js:134` — SMTP 发信与收件人白名单。
- `scrub` — `mailer.js:146` — SMTP 发信与收件人白名单。
- `transportOf` — `mailer.js:153` — SMTP 发信与收件人白名单。
- `verify` — `mailer.js:171` — SMTP 发信与收件人白名单。
- `send` — `mailer.js:189` — SMTP 发信与收件人白名单。
- `normalizeDeptTemplates` — `org.js:103` — 多租户组织、席位、套餐。
- `money` — `org.js:130` — 多租户组织、席位、套餐。
- `normalizeBudget` — `org.js:140` — 多租户组织、席位、套餐。
- `emptyDb` — `org.js:151` — 多租户组织、席位、套餐。
- `load` — `org.js:154` — 多租户组织、席位、套餐。
- `save` — `org.js:164` — 多租户组织、席位、套餐。
- `newId` — `org.js:171` — 多租户组织、席位、套餐。
- `ensureDefault` — `org.js:176` — 多租户组织、席位、套餐。
- `settingsOf` — `org.js:194` — 多租户组织、席位、套餐。
- `listOrgs` — `org.js:198` — 多租户组织、席位、套餐。
- `getOrg` — `org.js:205` — 多租户组织、席位、套餐。
- `multiTenant` — `org.js:210` — 多租户组织、席位、套餐。
- `orgIdOf` — `org.js:215` — 多租户组织、席位、套餐。
- `rootDirOf` — `org.js:224` — 多租户组织、席位、套餐。
- `createOrg` — `org.js:230` — 多租户组织、席位、套餐。
- `updateOrg` — `org.js:255` — 多租户组织、席位、套餐。
- `planInfo` — `org.js:319` — 多租户组织、席位、套餐。
- `listDepts` — `org.js:334` — 多租户组织、席位、套餐。
- `addDept` — `org.js:337` — 多租户组织、席位、套餐。
- `removeDept` — `org.js:350` — 多租户组织、席位、套餐。
- `createInvite` — `org.js:365` — 多租户组织、席位、套餐。
- `listInvites` — `org.js:388` — 多租户组织、席位、套餐。
- `peekInvite` — `org.js:395` — 多租户组织、席位、套餐。
- `consumeInvite` — `org.js:405` — 多租户组织、席位、套餐。
- `revokeInvite` — `org.js:414` — 多租户组织、席位、套餐。
- `pushAudit` — `org.js:425` — 多租户组织、席位、套餐。
- `audit` — `org.js:436` — 多租户组织、席位、套餐。
- `listAudit` — `org.js:450` — 多租户组织、席位、套餐。
- `feishuFetch` — `feishu-doc.js:22` — 飞书 convert API 写云文档。
- `getToken` — `feishu-doc.js:38` — 飞书 convert API 写云文档。
- `inlineEls` — `feishu-doc.js:56` — 飞书 convert API 写云文档。
- `mdToBlocks` — `feishu-doc.js:71` — 飞书 convert API 写云文档。
- `splitMarkdownImages` — `feishu-doc.js:105` — 飞书 convert API 写云文档。
- `convertViaApi` — `feishu-doc.js:127` — 飞书 convert API 写云文档。
- `appendDescendants` — `feishu-doc.js:145` — 飞书 convert API 写云文档。
- `appendImage` — `feishu-doc.js:175` — 飞书 convert API 写云文档。
- `sleepAbortable` — `feishu-doc.js:225` — 飞书 convert API 写云文档。
- `createFeishuDoc` — `feishu-doc.js:240` — 飞书 convert API 写云文档。
- `isPermissionError` — `feishu-doc.js:218` — 飞书 convert API 写云文档。
- `bump` — `metrics.js:46` — 运维指标与 Prometheus 导出。
- `observe` — `metrics.js:55` — 运维指标与 Prometheus 导出。
- `pct` — `metrics.js:69` — 运维指标与 Prometheus 导出。
- `diskFreePct` — `metrics.js:77` — 运维指标与 Prometheus 导出。
- `channelStreaks` — `metrics.js:85` — 运维指标与 Prometheus 导出。
- `auditBlocked` — `metrics.js:98` — 运维指标与 Prometheus 导出。
- `snapshot` — `metrics.js:118` — 运维指标与 Prometheus 导出。
- `shardOf` — `metrics.js:154` — 运维指标与 Prometheus 导出。
- `fileOf` — `metrics.js:155` — 运维指标与 Prometheus 导出。
- `shards` — `metrics.js:156` — 运维指标与 Prometheus 导出。
- `write` — `metrics.js:161` — 运维指标与 Prometheus 导出。
- `read` — `metrics.js:169` — 运维指标与 Prometheus 导出。
- `loadState` — `metrics.js:216` — 运维指标与 Prometheus 导出。
- `saveState` — `metrics.js:219` — 运维指标与 Prometheus 导出。
- `evaluate` — `metrics.js:229` — 运维指标与 Prometheus 导出。
- `start` — `metrics.js:263` — 运维指标与 Prometheus 导出。
- `stop` — `metrics.js:284` — 运维指标与 Prometheus 导出。
- `gaugeFn` — `metrics.js:116` — 运维指标与 Prometheus 导出。
- `isIntranet` — `intranet.js:26` — 内网探测与绑定。
- `isStr` — `systemone.js:79` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `txt` — `systemone.js:80` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `noul` — `systemone.js:83` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `choice` — `systemone.js:84` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `score` — `systemone.js:85` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `normalizeQuestions` — `systemone.js:101` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `stateOf` — `systemone.js:163` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `buildBody` — `systemone.js:179` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `sureOfNoul` — `systemone.js:184` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `probsOf` — `systemone.js:190` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `pct` — `systemone.js:197` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `readAnswers` — `systemone.js:212` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `lineOf` — `systemone.js:267` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `gate` — `systemone.js:292` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `errorOf` — `systemone.js:307` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `urlFromBase` — `systemone.js:333` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `zodSpot` — `systemone.js:342` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `costOf` — `systemone.js:358` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `costText` — `systemone.js:366` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `unitTableFor` — `pricing.js:180` — 积分与单价表。
- `normalizeUnitRow` — `pricing.js:201` — 积分与单价表。
- `unitPriceOf` — `pricing.js:214` — 积分与单价表。
- `costOfUnits` — `pricing.js:238` — 积分与单价表。
- `candidates` — `pricing.js:274` — 积分与单价表。
- `tableFor` — `pricing.js:290` — 积分与单价表。
- `normalizeRow` — `pricing.js:308` — 积分与单价表。
- `priceOf` — `pricing.js:325` — 积分与单价表。
- `costOf` — `pricing.js:353` — 积分与单价表。
- `discountOf` — `pricing.js:381` — 积分与单价表。
- `r6` — `pricing.js:386` — 积分与单价表。
- `yuanText` — `pricing.js:400` — 积分与单价表。
- `catalog` — `pricing.js:418` — 积分与单价表。
- `$` — `pricing.js:61` — 积分与单价表。
- `modeOf` — `modes.js:51` — 执行模式唯一真源：Ask / Plan / Craft / Goal，避免界面与 CLI 漂移。
- `normalizeMode` — `modes.js:53` — 执行模式唯一真源：Ask / Plan / Craft / Goal，避免界面与 CLI 漂移。
- `isMode` — `modes.js:54` — 执行模式唯一真源：Ask / Plan / Craft / Goal，避免界面与 CLI 漂移。
- `agentMode` — `modes.js:56` — 执行模式唯一真源：Ask / Plan / Craft / Goal，避免界面与 CLI 漂移。
- `isGoalMode` — `modes.js:58` — 执行模式唯一真源：Ask / Plan / Craft / Goal，避免界面与 CLI 漂移。
- `modeLabel` — `modes.js:59` — 执行模式唯一真源：Ask / Plan / Craft / Goal，避免界面与 CLI 漂移。
- `modeHint` — `modes.js:61` — 执行模式唯一真源：Ask / Plan / Craft / Goal，避免界面与 CLI 漂移。
- `stampToday` — `migrate.js:32` — 升级迁移。
- `readLedger` — `migrate.js:38` — 升级迁移。
- `backupCanvases` — `migrate.js:48` — 升级迁移。
- `tidyLooseFiles` — `migrate.js:84` — 升级迁移。
- `runMigrations` — `migrate.js:173` — 升级迁移。
- `line` — `callout.js:13` — 正文提示条：网页画图标，终端/IM 换文字。
- `strip` — `callout.js:18` — 正文提示条：网页画图标，终端/IM 换文字。
- `photoDataUrl` — `pet.js:51` — 桌面宠物窗口与状态机。
- `spriteBundle` — `pet.js:73` — 桌面宠物窗口与状态机。
- `posFile` — `pet.js:86` — 桌面宠物窗口与状态机。
- `loadPos` — `pet.js:89` — 桌面宠物窗口与状态机。
- `savePos` — `pet.js:92` — 桌面宠物窗口与状态机。
- `defaultPos` — `pet.js:101` — 桌面宠物窗口与状态机。
- `sanePos` — `pet.js:106` — 桌面宠物窗口与状态机。
- `toggleMain` — `pet.js:115` — 桌面宠物窗口与状态机。
- `bindIpc` — `pet.js:124` — 桌面宠物窗口与状态机。
- `create` — `pet.js:186` — 桌面宠物窗口与状态机。
- `push` — `pet.js:231` — 桌面宠物窗口与状态机。
- `setState` — `pet.js:254` — 桌面宠物窗口与状态机。
- `alertAsk` — `pet.js:270` — 桌面宠物窗口与状态机。
- `notifyFinish` — `pet.js:303` — 桌面宠物窗口与状态机。
- `stopWalk` — `pet.js:333` — 桌面宠物窗口与状态机。
- `startWalk` — `pet.js:338` — 桌面宠物窗口与状态机。
- `armWander` — `pet.js:361` — 桌面宠物窗口与状态机。
- `clearAsk` — `pet.js:371` — 桌面宠物窗口与状态机。
- `show` — `pet.js:379` — 桌面宠物窗口与状态机。
- `hide` — `pet.js:380` — 桌面宠物窗口与状态机。
- `isVisible` — `pet.js:386` — 桌面宠物窗口与状态机。
- `applyConfig` — `pet.js:389` — 桌面宠物窗口与状态机。
- `destroy` — `pet.js:413` — 桌面宠物窗口与状态机。
- `pushWecom` — `notify.js:6` — 群机器人推送。
- `pushDingtalk` — `notify.js:18` — 群机器人推送。
- `pushBots` — `notify.js:38` — 群机器人推送。
- `currentVersion` — `updater.js:16` — 检查 GitHub Releases 更新。
- `installKind` — `updater.js:28` — 检查 GitHub Releases 更新。
- `parseVer` — `updater.js:40` — 检查 GitHub Releases 更新。
- `cmpVer` — `updater.js:45` — 检查 GitHub Releases 更新。
- `howToUpdate` — `updater.js:55` — 检查 GitHub Releases 更新。
- `checkUpdate` — `updater.js:69` — 检查 GitHub Releases 更新。
- `resetCache` — `updater.js:99` — 检查 GitHub Releases 更新。
- `diagramCfg` — `diagram.js:22` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `plantumlEncode` — `diagram.js:32` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `renderECharts` — `diagram.js:43` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `renderDot` — `diagram.js:67` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `renderPlantuml` — `diagram.js:74` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `krokiRender` — `diagram.js:107` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `svgToPngAnyhow` — `diagram.js:128` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `looksLikeSvg` — `diagram.js:177` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `fixLabelBrackets` — `diagram.js:203` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `fixSubgraph` — `diagram.js:232` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `fixTimelineColon` — `diagram.js:247` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `fixGitBranch` — `diagram.js:261` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `repairMermaid` — `diagram.js:272` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `mermaidError` — `diagram.js:286` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `renderDiagram` — `diagram.js:304` — 四类专业图离线渲染：mermaid / Graphviz / ECharts / PlantUML。
- `localTraceFile` — `trace.js:47` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `isFillerSaid` — `trace.js:69` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `labelFromInput` — `trace.js:82` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `archiveName` — `trace.js:107` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `localFiles` — `trace.js:112` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `rotateIfNeeded` — `trace.js:124` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `localRecord` — `trace.js:140` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `readEventsOf` — `trace.js:162` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `localReadEvents` — `trace.js:183` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `localTraceList` — `trace.js:197` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `localTraceClear` — `trace.js:274` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `capText` — `trace.js:302` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `messagesOf` — `trace.js:314` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `cleanHost` — `trace.js:351` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `readCfg` — `trace.js:368` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `post` — `trace.js:402` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `createTracer` — `trace.js:430` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `getTracer` — `trace.js:700` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `nowIso` — `trace.js:298` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `uid` — `trace.js:299` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `worthRetry` — `relay.js:57` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `pickChannels` — `relay.js:73` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `num` — `relay.js:106` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `weightedShuffle` — `relay.js:109` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `prepBody` — `relay.js:130` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `usageOf` — `relay.js:139` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `forward` — `relay.js:158` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `createRouter` — `relay.js:232` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `assistantText` — `session-search.js:27` — 任务历史检索：正文、产出文件名、意思相近。
- `filesOf` — `session-search.js:40` — 任务历史检索：正文、产出文件名、意思相近。
- `digestOf` — `session-search.js:57` — 任务历史检索：正文、产出文件名、意思相近。
- `embedTextOf` — `session-search.js:84` — 任务历史检索：正文、产出文件名、意思相近。
- `normalize` — `session-search.js:90` — 任务历史检索：正文、产出文件名、意思相近。
- `bigrams` — `session-search.js:94` — 任务历史检索：正文、产出文件名、意思相近。
- `keywordScore` — `session-search.js:100` — 任务历史检索：正文、产出文件名、意思相近。
- `cosine` — `session-search.js:108` — 任务历史检索：正文、产出文件名、意思相近。
- `snippet` — `session-search.js:120` — 任务历史检索：正文、产出文件名、意思相近。
- `rank` — `session-search.js:145` — 任务历史检索：正文、产出文件名、意思相近。
- `searchNote` — `session-search.js:187` — 任务历史检索：正文、产出文件名、意思相近。
- `cacheName` — `thumb.js:45` — 缩略图池。
- `makeThumb` — `thumb.js:56` — 缩略图池。
- `thumbFile` — `thumb.js:82` — 缩略图池。
- `hasNativeImage` — `thumb.js:118` — 缩略图池。
- `retire` — `thumb.js:125` — 缩略图池。
- `spawn` — `thumb.js:149` — 缩略图池。
- `pump` — `thumb.js:174` — 缩略图池。
- `enqueueShrink` — `thumb.js:189` — 缩略图池。
- `thumbFileAsync` — `thumb.js:208` — 缩略图池。
- `closeThumbPool` — `thumb.js:236` — 缩略图池。
- `thumbPoolStats` — `thumb.js:251` — 缩略图池。
- `cols` — `text-width.js:14` — 终端东西方字符宽度。
- `padCols` — `text-width.js:17` — 终端东西方字符宽度。
- `dir` — `cli-live.js:47` — 终端与网页直播桥。
- `ensureDir` — `cli-live.js:50` — 终端与网页直播桥。
- `fileOf` — `cli-live.js:53` — 终端与网页直播桥。
- `pidAlive` — `cli-live.js:61` — 终端与网页直播桥。
- `readMeta` — `cli-live.js:67` — 终端与网页直播桥。
- `isLive` — `cli-live.js:75` — 终端与网页直播桥。
- `list` — `cli-live.js:87` — 终端与网页直播桥。
- `rowOf` — `cli-live.js:114` — 终端与网页直播桥。
- `get` — `cli-live.js:144` — 终端与网页直播桥。
- `drop` — `cli-live.js:151` — 终端与网页直播桥。
- `read` — `cli-live.js:164` — 终端与网页直播桥。
- `readLines` — `cli-live.js:205` — 终端与网页直播桥。
- `interject` — `cli-live.js:229` — 终端与网页直播桥。
- `pending` — `cli-live.js:250` — 终端与网页直播桥。
- `answer` — `cli-live.js:271` — 终端与网页直播桥。
- `announce` — `cli-live.js:295` — 终端与网页直播桥。
- `enabled` — `cli-live.js:45` — 终端与网页直播桥。
- `containedIn` — `plugins.js:50` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `validateManifest` — `plugins.js:75` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `discoverSkills` — `plugins.js:117` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `expandVars` — `plugins.js:167` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `discoverMcpServers` — `plugins.js:175` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `loadPlugin` — `plugins.js:262` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `loadPlugins` — `plugins.js:285` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `pluginSkills` — `plugins.js:296` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `pluginMcpServers` — `plugins.js:316` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `safePluginDirName` — `plugins.js:330` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `readSources` — `plugins.js:337` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `writeSources` — `plugins.js:345` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `pluginSource` — `plugins.js:349` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `installPluginFromGitHub` — `plugins.js:355` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `copyTree` — `plugins.js:414` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `removePlugin` — `plugins.js:424` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `updatePlugin` — `plugins.js:436` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `isPlainObject` — `plugins.js:47` — Agent Plugins 1.0.0：一个包同时带技能和 MCP。
- `wrap` — `cli-ask.js:25` — 终端回答 agent 提问。
- `render` — `cli-ask.js:52` — 终端回答 agent 提问。
- `hint` — `cli-ask.js:76` — 终端回答 agent 提问。
- `normalize` — `cli-ask.js:83` — 终端回答 agent 提问。
- `parse` — `cli-ask.js:110` — 终端回答 agent 提问。
- `retryText` — `cli-ask.js:138` — 终端回答 agent 提问。
- `run` — `cli-ask.js:162` — 终端回答 agent 提问。
- `waitText` — `cli-ask.js:181` — 终端回答 agent 提问。
- `readZip` — `preview.js:35` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `decodeEntities` — `preview.js:89` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `parseXml` — `preview.js:100` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `findAll` — `preview.js:149` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `textOf` — `preview.js:154` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `docxRuns` — `preview.js:164` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `docxParagraph` — `preview.js:187` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `docxToDoc` — `preview.js:208` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `pptxToSlides` — `preview.js:268` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `xlsxToSheets` — `preview.js:324` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `zipListing` — `preview.js:358` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `previewData` — `preview.js:375` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `kids` — `preview.js:145` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `child` — `preview.js:147` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `clip` — `preview.js:159` — docx/xlsx/pptx/zip 结构化预览，零新依赖拆 OOXML。
- `whyFailed` — `mcp.js:29` — MCP 客户端：stdio / Streamable HTTP，工具注入 agent，生命周期管理。
- `editDistance` — `cli-args.js:79` — CLI 参数表。
- `nearestFlag` — `cli-args.js:93` — CLI 参数表。
- `nearestSub` — `cli-args.js:104` — CLI 参数表。
- `looksLikeProse` — `cli-args.js:120` — CLI 参数表。
- `problem` — `cli-args.js:124` — CLI 参数表。
- `usableValue` — `cli-args.js:129` — CLI 参数表。
- `parse` — `cli-args.js:141` — CLI 参数表。
- `helpText` — `cli-args.js:265` — CLI 参数表。
- `completionScript` — `cli-args.js:302` — CLI 参数表。
- `problemText` — `cli-args.js:399` — CLI 参数表。
- `sh` — `cli-args.js:393` — CLI 参数表。
- `z` — `cli-args.js:396` — CLI 参数表。
- `endpointHost` — `cdp.js:25` — Chrome DevTools Protocol 真浏览器控制。
- `getJson` — `cdp.js:32` — Chrome DevTools Protocol 真浏览器控制。
- `idleMs` — `cdp.js:55` — Chrome DevTools Protocol 真浏览器控制。
- `clearIdle` — `cdp.js:61` — Chrome DevTools Protocol 真浏览器控制。
- `touchIdle` — `cdp.js:63` — Chrome DevTools Protocol 真浏览器控制。
- `killTree` — `cdp.js:71` — Chrome DevTools Protocol 真浏览器控制。
- `close` — `cdp.js:76` — Chrome DevTools Protocol 真浏览器控制。
- `hookExit` — `cdp.js:88` — Chrome DevTools Protocol 真浏览器控制。
- `findChrome` — `cdp.js:117` — Chrome DevTools Protocol 真浏览器控制。
- `probe` — `cdp.js:125` — Chrome DevTools Protocol 真浏览器控制。
- `readPortFile` — `cdp.js:132` — Chrome DevTools Protocol 真浏览器控制。
- `profileDir` — `cdp.js:135` — Chrome DevTools Protocol 真浏览器控制。
- `wantHeadless` — `cdp.js:146` — Chrome DevTools Protocol 真浏览器控制。
- `launch` — `cdp.js:152` — Chrome DevTools Protocol 真浏览器控制。
- `ensure` — `cdp.js:194` — Chrome DevTools Protocol 真浏览器控制。
- `frame` — `cdp.js:204` — Chrome DevTools Protocol 真浏览器控制。
- `connect` — `cdp.js:214` — Chrome DevTools Protocol 真浏览器控制。
- `withTab` — `cdp.js:261` — Chrome DevTools Protocol 真浏览器控制。
- `waitReady` — `cdp.js:275` — Chrome DevTools Protocol 真浏览器控制。
- `run` — `cdp.js:286` — Chrome DevTools Protocol 真浏览器控制。
- `isLocal` — `cdp.js:23` — Chrome DevTools Protocol 真浏览器控制。
- `sleep` — `cdp.js:46` — Chrome DevTools Protocol 真浏览器控制。
- `validateExperts` — `experts-lib.js:9` — 专家与专家团加载。
- `mergeBuiltinExperts` — `experts-lib.js:51` — 专家与专家团加载。
- `isPackaged` — `paths.js:24` — 数据目录解析：OPENWORKBUDDY_HOME / 打包后用户目录 / 源码旁 data。
- `dataPath` — `paths.js:42` — 数据目录解析：OPENWORKBUDDY_HOME / 打包后用户目录 / 源码旁 data。
- `appPath` — `paths.js:47` — 数据目录解析：OPENWORKBUDDY_HOME / 打包后用户目录 / 源码旁 data。
- `preferData` — `paths.js:52` — 数据目录解析：OPENWORKBUDDY_HOME / 打包后用户目录 / 源码旁 data。
- `copyTree` — `paths.js:76` — 数据目录解析：OPENWORKBUDDY_HOME / 打包后用户目录 / 源码旁 data。
- `copyIfMissing` — `paths.js:91` — 数据目录解析：OPENWORKBUDDY_HOME / 打包后用户目录 / 源码旁 data。
- `seedDataDir` — `paths.js:102` — 数据目录解析：OPENWORKBUDDY_HOME / 打包后用户目录 / 源码旁 data。
- `resolvePort` — `paths.js:124` — 数据目录解析：OPENWORKBUDDY_HOME / 打包后用户目录 / 源码旁 data。
- `renderHtmlToPng` — `htmlshot.js:18` — HTML 离屏截图串行队列。
- `doRender` — `htmlshot.js:25` — HTML 离屏截图串行队列。
- `endpointOf` — `gen-cache.js:65` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `fileFingerprint` — `gen-cache.js:80` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `key` — `gen-cache.js:100` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `load` — `gen-cache.js:135` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `save` — `gen-cache.js:145` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `prune` — `gen-cache.js:157` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `get` — `gen-cache.js:170` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `put` — `gen-cache.js:197` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `stats` — `gen-cache.js:220` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `clear` — `gen-cache.js:227` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `base` — `drama-pipeline.js:48` — 短剧进度与产出路径盘点。
- `isPlaceholder` — `drama-pipeline.js:49` — 短剧进度与产出路径盘点。
- `payloadOf` — `drama-pipeline.js:50` — 短剧进度与产出路径盘点。
- `kindOf` — `drama-pipeline.js:51` — 短剧进度与产出路径盘点。
- `outputOf` — `drama-pipeline.js:54` — 短剧进度与产出路径盘点。
- `outputPaths` — `drama-pipeline.js:72` — 短剧进度与产出路径盘点。
- `medianMs` — `drama-pipeline.js:86` — 短剧进度与产出路径盘点。
- `collectRuns` — `drama-pipeline.js:92` — 短剧进度与产出路径盘点。
- `dramaProgress` — `drama-pipeline.js:106` — 短剧进度与产出路径盘点。
- `insideRoot` — `shot-history.js:46` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `probeFile` — `shot-history.js:56` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `putFile` — `shot-history.js:69` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `readLedger` — `shot-history.js:85` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `appendLedger` — `shot-history.js:96` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `findTarget` — `shot-history.js:103` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `stable` — `shot-history.js:121` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `fingerprint` — `shot-history.js:128` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `snapshot` — `shot-history.js:140` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `blobState` — `shot-history.js:174` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `list` — `shot-history.js:186` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `changed` — `shot-history.js:215` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `restore` — `shot-history.js:229` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `usage` — `shot-history.js:290` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `gc` — `shot-history.js:306` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `sha` — `shot-history.js:37` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `storeDir` — `shot-history.js:38` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `objPath` — `shot-history.js:41` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `boardKey` — `shot-history.js:42` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `ledgerPath` — `shot-history.js:43` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `distance` — `config-lint.js:23` — 配置体检：Key 形态、地址、互斥字段。
- `nearest` — `config-lint.js:39` — 配置体检：Key 形态、地址、互斥字段。
- `lint` — `config-lint.js:79` — 配置体检：Key 形态、地址、互斥字段。
- `lines` — `config-lint.js:123` — 配置体检：Key 形态、地址、互斥字段。
- `KIND` — `config-lint.js:20` — 配置体检：Key 形态、地址、互斥字段。
- `suspendedFromTick` — `awake.js:37` — 睡眠检测与 keep-awake，合盖顺延时限。
- `startTicker` — `awake.js:42` — 睡眠检测与 keep-awake，合盖顺延时限。
- `watch` — `awake.js:63` — 睡眠检测与 keep-awake，合盖顺延时限。
- `totalSuspendedMs` — `awake.js:72` — 睡眠检测与 keep-awake，合盖顺延时限。
- `acquireAssertion` — `awake.js:81` — 睡眠检测与 keep-awake，合盖顺延时限。
- `releaseAssertion` — `awake.js:102` — 睡眠检测与 keep-awake，合盖顺延时限。
- `hold` — `awake.js:117` — 睡眠检测与 keep-awake，合盖顺延时限。
- `repairToolPairs` — `llm.js:31` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `toAnthropicMessages` — `llm.js:70` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `anthropicBase` — `llm.js:116` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `channelEnvName` — `llm.js:148` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `hostOf` — `llm.js:153` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `resolveKey` — `llm.js:159` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `cleanKey` — `llm.js:198` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `headerKey` — `llm.js:214` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `anthropicChat` — `llm.js:218` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `openaiUsage` — `llm.js:307` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `toOpenAIMessages` — `llm.js:317` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `rescueLeakedToolCalls` — `llm.js:354` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `sliceFirstObject` — `llm.js:382` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `createLeakGuard` — `llm.js:402` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `openaiChat` — `llm.js:423` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `keepBadArgs` — `llm.js:591` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `parseToolArgs` — `llm.js:610` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `parseOpenAIChoice` — `llm.js:629` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `chatWithRetry` — `llm.js:653` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `createLLM` — `llm.js:691` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `embedCandidates` — `llm.js:742` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `markEmbedChannelDead` — `llm.js:799` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `embedChannelDead` — `llm.js:800` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `createEmbedder` — `llm.js:807` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `chanKey` — `llm.js:798` — 多模型适配层：OpenAI 兼容与 Anthropic 官方协议、流式输出、工具调用还原、嵌入向量、
- `norm` — `thinking.js:38` — 思考档位到各家参数的映射。
- `planFor` — `thinking.js:60` — 思考档位到各家参数的映射。
- `planForEngine` — `thinking.js:131` — 思考档位到各家参数的映射。
- `isOn` — `thinking.js:44` — 思考档位到各家参数的映射。
- `dropVectorTwins` — `im.js:56` — IM 远程指挥总控：飞书、企微、钉钉、Webhook 入站出站、会话持久化、成果回传。
- `unwrapFeishuInbound` — `im.js:66` — IM 远程指挥总控：飞书、企微、钉钉、Webhook 入站出站、会话持久化、成果回传。
- `feishuDedupeKeys` — `im.js:74` — IM 远程指挥总控：飞书、企微、钉钉、Webhook 入站出站、会话持久化、成果回传。
- `createImRouter` — `im.js:91` — IM 远程指挥总控：飞书、企微、钉钉、Webhook 入站出站、会话持久化、成果回传。
- `findCmd` — `mcp-catalog.js:148` — 连接器广场目录与探测。
- `onPath` — `mcp-catalog.js:157` — 连接器广场目录与探测。
- `resolve` — `mcp-catalog.js:166` — 连接器广场目录与探测。
- `catalog` — `mcp-catalog.js:180` — 连接器广场目录与探测。
- `std` — `mcp-catalog.js:14` — 连接器广场目录与探测。
- `uvx` — `mcp-catalog.js:17` — 连接器广场目录与探测。
- `http` — `mcp-catalog.js:20` — 连接器广场目录与探测。
- `normalize` — `lanes.js:47` — 办公/工程两条泳道。
- `get` — `lanes.js:52` — 办公/工程两条泳道。
- `laneOf` — `lanes.js:67` — 办公/工程两条泳道。
- `engineSessionFor` — `lanes.js:78` — 办公/工程两条泳道。
- `rememberEngineSession` — `lanes.js:89` — 办公/工程两条泳道。
- `emptyDb` — `quota.js:113` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `load` — `quota.js:119` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `loadAll` — `quota.js:121` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `save` — `quota.js:123` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `localDay` — `quota.js:126` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `localMonth` — `quota.js:131` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `normalizeCap` — `quota.js:137` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `quotaTable` — `quota.js:149` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `withActor` — `quota.js:163` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `currentActor` — `quota.js:167` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `used` — `quota.js:173` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `check` — `quota.js:200` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `billable` — `quota.js:248` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `priceKeyOf` — `quota.js:251` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `gate` — `quota.js:262` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `undo` — `quota.js:293` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `record` — `quota.js:304` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `summary` — `quota.js:366` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `suggested` — `quota.js:397` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `normalizeTable` — `quota.js:404` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `permissionMode` — `security.js:42` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `engineGuard` — `security.js:72` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `getSecurity` — `security.js:124` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `audit` — `security.js:132` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `auditList` — `security.js:146` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `auditClear` — `security.js:149` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `auditExport` — `security.js:155` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `expandPath` — `security.js:161` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `underPrefix` — `security.js:164` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `resolvePathWithPolicy` — `security.js:172` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `splitSegments` — `security.js:216` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `stripEnvAssign` — `security.js:263` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `bareCommand` — `security.js:267` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `pathNeedles` — `security.js:279` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `addSessionAllow` — `security.js:298` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `listSessionAllow` — `security.js:303` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `clearSessionAllow` — `security.js:306` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `ruleFor` — `security.js:318` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `matchesPrefix` — `security.js:327` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `checkWrite` — `security.js:338` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `checkCommand` — `security.js:356` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `checkCode` — `security.js:402` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `checkUrl` — `security.js:430` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `watchApprovals` — `security.js:465` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `emitApproval` — `security.js:472` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `requestApproval` — `security.js:482` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `listApprovals` — `security.js:520` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `resolveApproval` — `security.js:530` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `effectiveScope` — `security.js:547` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `checkFullDisk` — `security.js:555` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `checkAccessibility` — `security.js:567` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `checkAutomation` — `security.js:578` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `openPrefPane` — `security.js:594` — 安全中心：文件/命令/网络三闸、权限档位、审计日志、命令审批、高危模式 DANGER_PATTERN
- `accepts` — `static-compress.js:49` — 静态资源压缩。
- `pickEncoding` — `static-compress.js:55` — 静态资源压缩。
- `compress` — `static-compress.js:60` — 静态资源压缩。
- `createStaticCompress` — `static-compress.js:75` — 静态资源压缩。
- `iconNames` — `icons.js:13` — 图标资源。
- `isIconName` — `icons.js:29` — 图标资源。
- `git` — `worktree.js:37` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `realOf` — `worktree.js:49` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `repoOf` — `worktree.js:66` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `plan` — `worktree.js:83` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `seedFrom` — `worktree.js:102` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `open` — `worktree.js:142` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `sigOf` — `worktree.js:177` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `readMeta` — `worktree.js:181` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `status` — `worktree.js:186` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `list` — `worktree.js:208` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `close` — `worktree.js:236` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `release` — `worktree.js:265` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `commitAll` — `worktree.js:274` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `sweep` — `worktree.js:286` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `markOf` — `worktree.js:302` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `hint` — `worktree.js:308` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `out` — `worktree.js:42` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `keyOf` — `worktree.js:43` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `safeName` — `worktree.js:45` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `metaPath` — `worktree.js:46` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `skillPatterns` — `electron-builder.config.js:32` — 项目支撑文件。
- `findAppDir` — `electron-builder.config.js:55` — 项目支撑文件。
- `afterPack` — `electron-builder.config.js:66` — 项目支撑文件。
- `adhocSign` — `electron-builder.config.js:77` — 项目支撑文件。
- `compress` — `json-compress.js:50` — JSON 压缩。
- `createJsonCompress` — `json-compress.js:66` — JSON 压缩。
- `trim` — `jev.js:25` — Jev 协议拼装。
- `keyOfProvider` — `jev.js:28` — Jev 协议拼装。
- `pickRoute` — `jev.js:40` — Jev 协议拼装。
- `status` — `jev.js:84` — Jev 协议拼装。
- `ask` — `jev.js:96` — Jev 协议拼装。
- `pick` — `jev.js:140` — Jev 协议拼装。
- `selftest` — `jev.js:152` — Jev 协议拼装。
- `paintDiff` — `cli.js:74` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `goalThink` — `cli.js:481` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `printGoalCard` — `cli.js:492` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `listCliSessions` — `cli.js:516` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `newSessionId` — `cli.js:538` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `saveSess` — `cli.js:557` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `makeEmit` — `cli.js:563` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `printSummary` — `cli.js:653` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `contextLine` — `cli.js:689` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `noteChanged` — `cli.js:718` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `drawOutputs` — `cli.js:744` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `hintOutputs` — `cli.js:779` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `remoteDeliver` — `cli.js:835` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `remoteWait` — `cli.js:847` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `raceRemote` — `cli.js:872` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `termReadLine` — `cli.js:904` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `askUserBoth` — `cli.js:955` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `handleApproval` — `cli.js:982` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `makeAskUser` — `cli.js:1024` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `runOnce` — `cli.js:1049` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `runOnceIn` — `cli.js:1077` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `readStdin` — `cli.js:1249` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `bringIn` — `cli.js:1272` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `splitFiles` — `cli.js:1287` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `dim` — `cli.js:69` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `yellow` — `cli.js:70` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `red` — `cli.js:71` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `green` — `cli.js:72` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `bold` — `cli.js:81` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `prog` — `cli.js:83` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `answer` — `cli.js:85` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `emitJson` — `cli.js:87` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `newMdRenderer` — `cli.js:96` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `permNow` — `cli.js:168` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `cfgEngine` — `cli.js:441` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `sessFileOf` — `cli.js:506` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `imgCap` — `cli.js:730` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `somebodyHome` — `cli.js:821` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `newId` — `cli.js:947` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `askPrompt` — `cli.js:1022` — openworkbuddy 命令行入口：单发、REPL、管道、doctor、engines、pair
- `getWS` — `im-qq.js:28` — QQ 机器人通道。
- `splitText` — `im-qq.js:38` — QQ 机器人通道。
- `createQQConnection` — `im-qq.js:58` — QQ 机器人通道。
- `crc32` — `thumb-png.js:29` — PNG 缩略图。
- `readChunks` — `thumb-png.js:45` — PNG 缩略图。
- `chunk` — `thumb-png.js:63` — PNG 缩略图。
- `pngInfo` — `thumb-png.js:73` — PNG 缩略图。
- `shrinkPng` — `thumb-png.js:91` — PNG 缩略图。
- `protoOfKind` — `media-models.js:54` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `guessCap` — `media-models.js:198` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `capOfModel` — `media-models.js:224` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `uniqueId` — `media-models.js:244` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `baseForUse` — `media-models.js:258` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `providerKeyOf` — `media-models.js:271` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `guessKind` — `media-models.js:277` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `videoProtoOf` — `media-models.js:314` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `baseOfKind` — `media-models.js:331` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `normalizeProviders` — `media-models.js:343` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `dedupeProviders` — `media-models.js:375` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `normalize` — `media-models.js:422` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `flatten` — `media-models.js:508` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `resolve` — `media-models.js:533` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `pick` — `media-models.js:554` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `catalogFor` — `media-models.js:571` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `brandInCatalog` — `media-models.js:613` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `arkDated` — `media-models.js:631` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `brandOf` — `media-models.js:642` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `mismatch` — `media-models.js:659` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `kindLabel` — `media-models.js:667` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `rehomeMismatched` — `media-models.js:679` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `MODAL` — `media-models.js:204` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `slug` — `media-models.js:241` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `chanKeyOf` — `chat-models.js:43` — 对话渠道表、选型、健康账本。
- `nameForKind` — `chat-models.js:48` — 对话渠道表、选型、健康账本。
- `wantsChannel` — `chat-models.js:65` — 对话渠道表、选型、健康账本。
- `envKeyFor` — `chat-models.js:87` — 对话渠道表、选型、健康账本。
- `pruneSeededPresets` — `chat-models.js:108` — 对话渠道表、选型、健康账本。
- `templates` — `chat-models.js:131` — 对话渠道表、选型、健康账本。
- `planTemplate` — `chat-models.js:151` — 对话渠道表、选型、健康账本。
- `commitTemplate` — `chat-models.js:165` — 对话渠道表、选型、健康账本。
- `legacyRows` — `chat-models.js:188` — 对话渠道表、选型、健康账本。
- `normalize` — `chat-models.js:207` — 对话渠道表、选型、健康账本。
- `modelsOf` — `chat-models.js:277` — 对话渠道表、选型、健康账本。
- `isLocalBase` — `chat-models.js:34` — 对话渠道表、选型、健康账本。
- `nb` — `chat-models.js:35` — 对话渠道表、选型、健康账本。
- `isSeededPreset` — `chat-models.js:85` — 对话渠道表、选型、健康账本。
- `createImSessionStore` — `im-store.js:17` — IM 会话独立落盘，重启不失忆。
- `makeStore` — `usage-store.js:44` — 分片用量账本。
- `classifyToolError` — `evolve.js:59` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `classifyEvent` — `evolve.js:114` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `readSessions` — `evolve.js:137` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `mineSignals` — `evolve.js:156` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `readFeedback` — `evolve.js:230` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `recordFeedback` — `evolve.js:232` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `feedbackSummary` — `evolve.js:255` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `activeRules` — `evolve.js:295` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `ruleDigest` — `evolve.js:310` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `rulesChars` — `evolve.js:315` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `promptBlock` — `evolve.js:331` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `gateProposal` — `evolve.js:369` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `listProposals` — `evolve.js:411` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `saveProposals` — `evolve.js:412` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `addProposals` — `evolve.js:414` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `decideProposal` — `evolve.js:432` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `retireRule` — `evolve.js:458` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `baselineOf` — `evolve.js:482` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `scoreRules` — `evolve.js:489` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `signalsForPrompt` — `evolve.js:552` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `proposeEdits` — `evolve.js:567` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `recordRun` — `evolve.js:607` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `listRuns` — `evolve.js:614` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `runReview` — `evolve.js:617` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `ensureDir` — `evolve.js:49` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `nowIso` — `evolve.js:50` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `uid` — `evolve.js:51` — 自进化闭环：从会话挖信号、反馈落盘、提案闸门、人审、规则打分下架。
- `render` — `cli-approve.js:36` — 终端/手机审批危险操作。
- `hint` — `cli-approve.js:68` — 终端/手机审批危险操作。
- `parse` — `cli-approve.js:81` — 终端/手机审批危险操作。
- `run` — `cli-approve.js:104` — 终端/手机审批危险操作。
- `waitText` — `cli-approve.js:120` — 终端/手机审批危险操作。
- `card` — `cli-approve.js:126` — 终端/手机审批危险操作。
- `platformAdmin` — `admin.js:31` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `platformOnly` — `admin.js:34` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `platformOwnerOnly` — `admin.js:39` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `guarded` — `admin.js:44` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `setDeployment` — `admin.js:149` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `isSoloDesktop` — `admin.js:153` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `ownsGlobalWorkspace` — `admin.js:158` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `platformGuard` — `admin.js:161` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `redactSecrets` — `admin.js:193` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `redactGuard` — `admin.js:202` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `tenantScope` — `admin.js:222` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `createAdminRouter` — `admin.js:269` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `safeCall` — `admin.js:811` — 企业管理后台 API：组织、席位、用量、安全策略、渠道与 Key、部门模板。
- `tokenize` — `cli-attach.js:37` — CLI 带文件和图片。
- `fromFileUrl` — `cli-attach.js:62` — CLI 带文件和图片。
- `pathLike` — `cli-attach.js:71` — CLI 带文件和图片。
- `expandHome` — `cli-attach.js:76` — CLI 带文件和图片。
- `escPath` — `cli-attach.js:84` — CLI 带文件和图片。
- `parseLine` — `cli-attach.js:94` — CLI 带文件和图片。
- `atToken` — `cli-attach.js:151` — CLI 带文件和图片。
- `isInside` — `cli-attach.js:160` — CLI 带文件和图片。
- `cleanName` — `cli-attach.js:168` — CLI 带文件和图片。
- `freeName` — `cli-attach.js:174` — CLI 带文件和图片。
- `stampName` — `cli-attach.js:184` — CLI 带文件和图片。
- `collect` — `cli-attach.js:194` — CLI 带文件和图片。
- `note` — `cli-attach.js:232` — CLI 带文件和图片。
- `withNote` — `cli-attach.js:236` — CLI 带文件和图片。
- `clipboardPlan` — `cli-attach.js:253` — CLI 带文件和图片。
- `readClipboard` — `cli-attach.js:296` — CLI 带文件和图片。
- `clipboardPutPlan` — `cli-attach.js:354` — CLI 带文件和图片。
- `writeClipboard` — `cli-attach.js:388` — CLI 带文件和图片。
- `mb` — `cli-attach.js:165` — CLI 带文件和图片。
- `anyImage` — `cli-attach.js:241` — CLI 带文件和图片。
- `readJson` — `store.js:27` — JSON 小仓库：原子写 + .bak 兜底 + 坏文件隔离；账本走 strict 模式绝不静默回滚
- `recover` — `store.js:50` — JSON 小仓库：原子写 + .bak 兜底 + 坏文件隔离；账本走 strict 模式绝不静默回滚
- `tighten` — `store.js:81` — JSON 小仓库：原子写 + .bak 兜底 + 坏文件隔离；账本走 strict 模式绝不静默回滚
- `writeJsonAtomic` — `store.js:92` — JSON 小仓库：原子写 + .bak 兜底 + 坏文件隔离；账本走 strict 模式绝不静默回滚
- `withPrefs` — `prefs.js:40` — 按账号存储的引擎、思考档、外观偏好。
- `current` — `prefs.js:44` — 按账号存储的引擎、思考档、外观偏好。
- `keyOf` — `prefs.js:54` — 按账号存储的引擎、思考档、外观偏好。
- `fileOf` — `prefs.js:61` — 按账号存储的引擎、思考档、外观偏好。
- `read` — `prefs.js:74` — 按账号存储的引擎、思考档、外观偏好。
- `write` — `prefs.js:98` — 按账号存储的引擎、思考档、外观偏好。
- `merge` — `prefs.js:109` — 按账号存储的引擎、思考档、外观偏好。
- `isPersonalPatch` — `prefs.js:129` — 按账号存储的引擎、思考档、外观偏好。
- `split` — `prefs.js:162` — 按账号存储的引擎、思考档、外观偏好。
- `agentCfg` — `prefs.js:205` — 按账号存储的引擎、思考档、外观偏好。
- `agentView` — `prefs.js:220` — 按账号存储的引擎、思考档、外观偏好。
- `petCfg` — `prefs.js:224` — 按账号存储的引擎、思考档、外观偏好。
- `shortcutsCfg` — `prefs.js:228` — 按账号存储的引擎、思考档、外观偏好。
- `modelCfg` — `prefs.js:232` — 按账号存储的引擎、思考档、外观偏好。
- `monthKey` — `budget.js:50` — 额度预扣与释放，防止并发超卖。
- `spentOf` — `budget.js:64` — 额度预扣与释放，防止并发超卖。
- `round6` — `budget.js:84` — 额度预扣与释放，防止并发超卖。
- `limitsOf` — `budget.js:94` — 额度预扣与释放，防止并发超卖。
- `estimate` — `budget.js:125` — 额度预扣与释放，防止并发超卖。
- `reserve` — `budget.js:143` — 额度预扣与释放，防止并发超卖。
- `settle` — `budget.js:195` — 额度预扣与释放，防止并发超卖。
- `release` — `budget.js:210` — 额度预扣与释放，防止并发超卖。
- `sweep` — `budget.js:226` — 额度预扣与释放，防止并发超卖。
- `status` — `budget.js:240` — 额度预扣与释放，防止并发超卖。
- `exhausted` — `budget.js:275` — 额度预扣与释放，防止并发超卖。
- `record` — `budget.js:305` — 额度预扣与释放，防止并发超卖。
- `invalidate` — `budget.js:322` — 额度预扣与释放，防止并发超卖。
- `item` — `doctor.js:28` — openworkbuddy doctor 体检。
- `verdictNode` — `doctor.js:35` — openworkbuddy doctor 体检。
- `verdictDeps` — `doctor.js:47` — openworkbuddy doctor 体检。
- `verdictDataDir` — `doctor.js:61` — openworkbuddy doctor 体检。
- `verdictConfig` — `doctor.js:75` — openworkbuddy doctor 体检。
- `verdictConfigLint` — `doctor.js:96` — openworkbuddy doctor 体检。
- `lintConfig` — `doctor.js:105` — openworkbuddy doctor 体检。
- `verdictModels` — `doctor.js:118` — openworkbuddy doctor 体检。
- `verdictPort` — `doctor.js:143` — openworkbuddy doctor 体检。
- `verdictWorkspace` — `doctor.js:169` — openworkbuddy doctor 体检。
- `verdictEngine` — `doctor.js:177` — openworkbuddy doctor 体检。
- `verdictTools` — `doctor.js:194` — openworkbuddy doctor 体检。
- `verdictToolward` — `doctor.js:213` — openworkbuddy doctor 体检。
- `worst` — `doctor.js:227` — openworkbuddy doctor 体检。
- `probeWritable` — `doctor.js:239` — openworkbuddy doctor 体检。
- `fetchText` — `doctor.js:267` — openworkbuddy doctor 体检。
- `probeWho` — `doctor.js:295` — openworkbuddy doctor 体检。
- `probePort` — `doctor.js:307` — openworkbuddy doctor 体检。
- `countModels` — `doctor.js:319` — openworkbuddy doctor 体检。
- `knownTool` — `doctor.js:354` — openworkbuddy doctor 体检。
- `probeTools` — `doctor.js:370` — openworkbuddy doctor 体检。
- `gather` — `doctor.js:385` — openworkbuddy doctor 体检。
- `render` — `doctor.js:438` — openworkbuddy doctor 体检。
- `failCode` — `eval/run.js:68` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `seedMemories` — `eval/run.js:101` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `gitCommit` — `eval/run.js:123` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `artifactExcerpts` — `eval/run.js:130` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `judgeOne` — `eval/run.js:141` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `main` — `eval/run.js:185` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `STAMP` — `eval/run.js:24` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `argOf` — `eval/run.js:50` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `hasFlag` — `eval/run.js:51` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `turnsOf` — `eval/run.js:86` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `stepsFor` — `eval/run.js:90` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `timeoutFor` — `eval/run.js:91` — 黑盒评测运行器：pass@k、AI 评委、基线对比。
- `runNode` — `eval/tasks.js:19` — 评测题库。
- `htmlIntact` — `eval/tasks.js:29` — 评测题库。
- `exists` — `eval/tasks.js:25` — 评测题库。
- `read` — `eval/tasks.js:26` — 评测题库。
- `ck` — `eval/tasks.js:27` — 评测题库。
- `subdirs` — `engines/which.js:33` — 探测本机引擎可执行文件。
- `extraDirs` — `engines/which.js:41` — 探测本机引擎可执行文件。
- `runnable` — `engines/which.js:67` — 探测本机引擎可执行文件。
- `findIn` — `engines/which.js:79` — 探测本机引擎可执行文件。
- `searchDirs` — `engines/which.js:92` — 探测本机引擎可执行文件。
- `augmentedPath` — `engines/which.js:101` — 探测本机引擎可执行文件。
- `askLoginShell` — `engines/which.js:110` — 探测本机引擎可执行文件。
- `forget` — `engines/which.js:141` — 探测本机引擎可执行文件。
- `resolveBin` — `engines/which.js:150` — 探测本机引擎可执行文件。
- `shimScript` — `engines/win.js:42` — Windows 引擎路径。
- `pickNode` — `engines/win.js:64` — Windows 引擎路径。
- `escapeArg` — `engines/win.js:76` — Windows 引擎路径。
- `launchPlan` — `engines/win.js:95` — Windows 引擎路径。
- `killTree` — `engines/win.js:137` — Windows 引擎路径。
- `isWin` — `engines/win.js:29` — Windows 引擎路径。
- `isBatch` — `engines/win.js:31` — Windows 引擎路径。
- `runJsonl` — `engines/jsonl.js:38` — 引擎 JSONL 事件流解析。
- `probeVersion` — `engines/jsonl.js:130` — 引擎 JSONL 事件流解析。
- `probeOption` — `engines/jsonl.js:158` — 引擎 JSONL 事件流解析。
- `probeHelp` — `engines/jsonl.js:180` — 引擎 JSONL 事件流解析。
- `firstVersionLine` — `engines/jsonl.js:196` — 引擎 JSONL 事件流解析。
- `purposeOf` — `engines/claude-code.js:32` — 本机 Claude Code 引擎适配。
- `textOfToolResult` — `engines/claude-code.js:38` — 本机 Claude Code 引擎适配。
- `explain` — `engines/claude-code.js:50` — 本机 Claude Code 引擎适配。
- `probeThinking` — `engines/claude-code.js:73` — 本机 Claude Code 引擎适配。
- `probeAddDir` — `engines/claude-code.js:85` — 本机 Claude Code 引擎适配。
- `pickAddDirs` — `engines/claude-code.js:96` — 本机 Claude Code 引擎适配。
- `detect` — `engines/claude-code.js:115` — 本机 Claude Code 引擎适配。
- `run` — `engines/claude-code.js:132` — 本机 Claude Code 引擎适配。
- `list` — `engines/index.js:34` — 执行引擎选择：内置循环 / Claude Code / Codex。
- `get` — `engines/index.js:38` — 执行引擎选择：内置循环 / Claude Code / Codex。
- `detectAll` — `engines/index.js:55` — 执行引擎选择：内置循环 / Claude Code / Codex。
- `detectAllUncached` — `engines/index.js:70` — 执行引擎选择：内置循环 / Claude Code / Codex。
- `resolve` — `engines/index.js:102` — 执行引擎选择：内置循环 / Claude Code / Codex。
- `testConnect` — `engines/index.js:127` — 执行引擎选择：内置循环 / Claude Code / Codex。
- `ask` — `engines/index.js:175` — 执行引擎选择：内置循环 / Claude Code / Codex。
- `nodeLauncher` — `engines/bridge.js:30` — 把本项目工具借给外部 CLI（MCP）。
- `buildServers` — `engines/bridge.js:46` — 把本项目工具借给外部 CLI（MCP）。
- `writeMcpConfig` — `engines/bridge.js:76` — 把本项目工具借给外部 CLI（MCP）。
- `codexArgs` — `engines/bridge.js:88` — 把本项目工具借给外部 CLI（MCP）。
- `writeShim` — `engines/bridge.js:113` — 把本项目工具借给外部 CLI（MCP）。
- `attach` — `engines/bridge.js:130` — 把本项目工具借给外部 CLI（MCP）。
- `shorten` — `engines/codex.js:32` — 本机 Codex 引擎适配。
- `sourceCodexHome` — `engines/codex.js:47` — 本机 Codex 引擎适配。
- `configuredModels` — `engines/codex.js:51` — 本机 Codex 引擎适配。
- `openWorkBuddyCodexHome` — `engines/codex.js:60` — 本机 Codex 引擎适配。
- `toolOf` — `engines/codex.js:80` — 本机 Codex 引擎适配。
- `explain` — `engines/codex.js:90` — 本机 Codex 引擎适配。
- `detect` — `engines/codex.js:102` — 本机 Codex 引擎适配。
- `run` — `engines/codex.js:118` — 本机 Codex 引擎适配。
- `loadConfig` — `engines/tool-bridge.js:77` — 外部引擎工具桥细节。
- `lentDefs` — `engines/tool-bridge.js:95` — 外部引擎工具桥细节。
- `listTools` — `engines/tool-bridge.js:107` — 外部引擎工具桥细节。
- `callTool` — `engines/tool-bridge.js:115` — 外部引擎工具桥细节。
- `send` — `engines/tool-bridge.js:132` — 外部引擎工具桥细节。
- `exitIfIdle` — `engines/tool-bridge.js:141` — 外部引擎工具桥细节。
- `log` — `engines/tool-bridge.js:149` — 外部引擎工具桥细节。
- `handle` — `engines/tool-bridge.js:154` — 外部引擎工具桥细节。
- `main` — `engines/tool-bridge.js:202` — 外部引擎工具桥细节。
- `readArgs` — `engines/tool-bridge.js:228` — 外部引擎工具桥细节。
- `cliList` — `engines/tool-bridge.js:239` — 外部引擎工具桥细节。
- `cli` — `engines/tool-bridge.js:248` — 外部引擎工具桥细节。
- `toErr` — `engines/tool-bridge.js:45` — 外部引擎工具桥细节。
- `defaultPairs` — `scripts/demo-mask.js:9` — 维护脚本：统计、图标、演示录制、打包检查。
- `maskScript` — `scripts/demo-mask.js:24` — 维护脚本：统计、图标、演示录制、打包检查。
- `parseArgs` — `scripts/record-demo.js:52` — 维护脚本：统计、图标、演示录制、打包检查。
- `seedHome` — `scripts/record-demo.js:76` — 维护脚本：统计、图标、演示录制、打包检查。
- `waitHttp` — `scripts/record-demo.js:97` — 维护脚本：统计、图标、演示录制、打包检查。
- `ensureLoggedIn` — `scripts/record-demo.js:106` — 维护脚本：统计、图标、演示录制、打包检查。
- `ffmpeg` — `scripts/record-demo.js:156` — 维护脚本：统计、图标、演示录制、打包检查。
- `installMask` — `scripts/record-demo.js:163` — 维护脚本：统计、图标、演示录制、打包检查。
- `typeInto` — `scripts/record-demo.js:175` — 维护脚本：统计、图标、演示录制、打包检查。
- `sleep` — `scripts/record-demo.js:72` — 维护脚本：统计、图标、演示录制、打包检查。
- `log` — `scripts/record-demo.js:73` — 维护脚本：统计、图标、演示录制、打包检查。
- `wireReadme` — `scripts/demo-readme.js:16` — 维护脚本：统计、图标、演示录制、打包检查。
- `wireReadmes` — `scripts/demo-readme.js:26` — 维护脚本：统计、图标、演示录制、打包检查。
- `pageScript` — `scripts/shot-ui.js:81` — 维护脚本：统计、图标、演示录制、打包检查。
- `serveStatic` — `scripts/shot-ui.js:112` — 维护脚本：统计、图标、演示录制、打包检查。
- `shoot` — `scripts/shot-ui.js:125` — 维护脚本：统计、图标、演示录制、打包检查。
- `read` — `scripts/stats.js:19` — 维护脚本：统计、图标、演示录制、打包检查。
- `bareRequires` — `scripts/check-package-files.js:53` — 维护脚本：统计、图标、演示录制、打包检查。
- `missingDeps` — `scripts/check-package-files.js:69` — 维护脚本：统计、图标、演示录制、打包检查。
- `localRequires` — `scripts/check-package-files.js:94` — 维护脚本：统计、图标、演示录制、打包检查。
- `walkGraph` — `scripts/check-package-files.js:107` — 维护脚本：统计、图标、演示录制、打包检查。
- `missingFrom` — `scripts/check-package-files.js:141` — 维护脚本：统计、图标、演示录制、打包检查。
- `assertPackComplete` — `scripts/check-package-files.js:146` — 维护脚本：统计、图标、演示录制、打包检查。
- `assertDepsRequirable` — `scripts/check-package-files.js:171` — 维护脚本：统计、图标、演示录制、打包检查。
- `firstLine` — `scripts/check-package-files.js:208` — 维护脚本：统计、图标、演示录制、打包检查。
- `assertSlimmed` — `scripts/check-package-files.js:221` — 维护脚本：统计、图标、演示录制、打包检查。
- `fitDurations` — `scripts/demo-timing.js:13` — 维护脚本：统计、图标、演示录制、打包检查。
- `sleep` — `scripts/shot-card.js:26` — 维护脚本：统计、图标、演示录制、打包检查。
- `shoot` — `scripts/shot-card.js:44` — 维护脚本：统计、图标、演示录制、打包检查。
- `keyScript` — `scripts/shot-card.js:28` — 维护脚本：统计、图标、演示录制、打包检查。
- `newSched` — `test/lifecycle.js:57` — 入职离职：一次关权限不删数据，出交接回执。
- `call` — `test/lifecycle.js:406` — 入职离职：一次关权限不删数据，出交接回执。
- `ok` — `test/lifecycle.js:44` — 入职离职：一次关权限不删数据，出交接回执。
- `eq` — `test/lifecycle.js:48` — 入职离职：一次关权限不删数据，出交接回执。
- `throws` — `test/lifecycle.js:50` — 入职离职：一次关权限不删数据，出交接回执。
- `V` — `test/lifecycle.js:62` — 入职离职：一次关权限不删数据，出交接回执。
- `U` — `test/lifecycle.js:202` — 入职离职：一次关权限不删数据，出交接回执。
- `ok` — `test/repl-commands.js:27` — REPL 斜杠命令。
- `eq` — `test/repl-commands.js:31` — REPL 斜杠命令。
- `tag` — `test/repl-commands.js:46` — REPL 斜杠命令。
- `fakeInbox` — `test/repl-commands.js:56` — REPL 斜杠命令。
- `main` — `test/repl-commands.js:75` — REPL 斜杠命令。
- `ok` — `test/sweep.js:36` — 清中间物：只删本轮名单且路径必须在工作区内。
- `tree` — `test/sweep.js:42` — 清中间物：只删本轮名单且路径必须在工作区内。
- `fixture` — `test/sweep.js:52` — 清中间物：只删本轮名单且路径必须在工作区内。
- `group` — `test/sweep.js:54` — 清中间物：只删本轮名单且路径必须在工作区内。
- `pathsOf` — `test/sweep.js:55` — 清中间物：只删本轮名单且路径必须在工作区内。
- `pfx` — `test/sweep.js:63` — 清中间物：只删本轮名单且路径必须在工作区内。
- `frames` — `test/sweep.js:65` — 清中间物：只删本轮名单且路径必须在工作区内。
- `clean` — `test/office-tools.js:41` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ok` — `test/office-tools.js:45` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `eq` — `test/office-tools.js:49` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `has` — `test/office-tools.js:53` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `makeZip` — `test/office-tools.js:73` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `makeFixtures` — `test/office-tools.js:102` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `run` — `test/office-tools.js:145` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `eq` — `test/media-health.js:22` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `ok` — `test/media-health.js:23` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `err` — `test/media-health.js:26` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `fresh` — `test/media-health.js:27` — 媒体渠道熔断：连挂硬错后暂停，提示词里写明。
- `ok` — `test/toolward.js:32` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `tryInstall` — `test/toolward.js:290` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `mkClean` — `test/toolward.js:295` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `F` — `test/toolward.js:48` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `REP` — `test/toolward.js:56` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `mkGuard` — `test/toolward.js:115` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `plan` — `test/toolward.js:209` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `calls` — `test/toolward.js:210` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `clearCalls` — `test/toolward.js:211` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `installedDir` — `test/toolward.js:289` — 可选外挂第二把尺子：本机装了 toolward 则合入结论且只严不松。
- `call` — `test/rbac.js:60` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `login` — `test/rbac.js:84` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `ok` — `test/rbac.js:38` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `eq` — `test/rbac.js:42` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `why` — `test/rbac.js:44` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `denied` — `test/rbac.js:46` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `allowed` — `test/rbac.js:47` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `raw` — `test/rbac.js:90` — RBAC 唯一真源：角色分档、只能管比自己低的一档、超管只能转让。
- `ok` — `test/checkpoints.js:18` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `read` — `test/checkpoints.js:22` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `rec` — `test/checkpoints.js:52` — 文件改前留底：内容寻址对象库、整步回退、改前 diff、垃圾回收。
- `ignored` — `test/deploy.js:50` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `done` — `test/deploy.js:625` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ok` — `test/deploy.js:33` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `read` — `test/deploy.js:37` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `bare` — `test/deploy.js:39` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `esc` — `test/frontend.js:162` — 真 Chromium 里跑前端源码切片断言。
- `fileIcon` — `test/frontend.js:164` — 真 Chromium 里跑前端源码切片断言。
- `fmtSize` — `test/frontend.js:165` — 真 Chromium 里跑前端源码切片断言。
- `revealFile` — `test/frontend.js:167` — 真 Chromium 里跑前端源码切片断言。
- `previewFile` — `test/frontend.js:168` — 真 Chromium 里跑前端源码切片断言。
- `startPreview` — `test/frontend.js:169` — 真 Chromium 里跑前端源码切片断言。
- `toast` — `test/frontend.js:171` — 真 Chromium 里跑前端源码切片断言。
- `onActivate` — `test/frontend.js:173` — 真 Chromium 里跑前端源码切片断言。
- `esc` — `test/frontend.js:190` — 真 Chromium 里跑前端源码切片断言。
- `canOpenOnHost` — `test/frontend.js:196` — 真 Chromium 里跑前端源码切片断言。
- `revealFile` — `test/frontend.js:197` — 真 Chromium 里跑前端源码切片断言。
- `renderFiles` — `test/frontend.js:198` — 真 Chromium 里跑前端源码切片断言。
- `toast` — `test/frontend.js:199` — 真 Chromium 里跑前端源码切片断言。
- `openSweep` — `test/frontend.js:200` — 真 Chromium 里跑前端源码切片断言。
- `assertFailedCardsCollapsed` — `test/frontend.js:6107` — 真 Chromium 里跑前端源码切片断言。
- `snapshotFiles` — `test/frontend.js:6762` — 真 Chromium 里跑前端源码切片断言。
- `previewFile` — `test/frontend.js:6765` — 真 Chromium 里跑前端源码切片断言。
- `toast` — `test/frontend.js:8877` — 真 Chromium 里跑前端源码切片断言。
- `canvasSetSelection` — `test/frontend.js:9076` — 真 Chromium 里跑前端源码切片断言。
- `canvasRenderInspector` — `test/frontend.js:9077` — 真 Chromium 里跑前端源码切片断言。
- `canvasPersist` — `test/frontend.js:9078` — 真 Chromium 里跑前端源码切片断言。
- `canvasToast` — `test/frontend.js:9079` — 真 Chromium 里跑前端源码切片断言。
- `canvasUndo` — `test/frontend.js:9080` — 真 Chromium 里跑前端源码切片断言。
- `canvasRedo` — `test/frontend.js:9081` — 真 Chromium 里跑前端源码切片断言。
- `mkWin` — `test/frontend.js:9152` — 真 Chromium 里跑前端源码切片断言。
- `srcLine` — `test/frontend.js:99` — 真 Chromium 里跑前端源码切片断言。
- `srcBlock` — `test/frontend.js:104` — 真 Chromium 里跑前端源码切片断言。
- `INDEX_CSS` — `test/frontend.js:122` — 真 Chromium 里跑前端源码切片断言。
- `INDEX_FP_FILTER` — `test/frontend.js:129` — 真 Chromium 里跑前端源码切片断言。
- `revealBtn` — `test/frontend.js:166` — 真 Chromium 里跑前端源码切片断言。
- `FB_WRAP` — `test/frontend.js:1117` — 真 Chromium 里跑前端源码切片断言。
- `pickLine` — `test/frontend.js:1246` — 真 Chromium 里跑前端源码切片断言。
- `CTX_BAR_HTML` — `test/frontend.js:1496` — 真 Chromium 里跑前端源码切片断言。
- `APP03_KS` — `test/frontend.js:1568` — 真 Chromium 里跑前端源码切片断言。
- `AUTH_CARD` — `test/frontend.js:1900` — 真 Chromium 里跑前端源码切片断言。
- `AUTH_WRAP` — `test/frontend.js:1928` — 真 Chromium 里跑前端源码切片断言。
- `esc` — `test/frontend.js:3677` — 真 Chromium 里跑前端源码切片断言。
- `saveSettings` — `test/frontend.js:3684` — 真 Chromium 里跑前端源码切片断言。
- `refreshImStatus` — `test/frontend.js:3685` — 真 Chromium 里跑前端源码切片断言。
- `renderSettings` — `test/frontend.js:3686` — 真 Chromium 里跑前端源码切片断言。
- `renderLarkQr` — `test/frontend.js:3687` — 真 Chromium 里跑前端源码切片断言。
- `OPENWS_SITES` — `test/frontend.js:4277` — 真 Chromium 里跑前端源码切片断言。
- `AVA_SRC` — `test/frontend.js:4602` — 真 Chromium 里跑前端源码切片断言。
- `SPRITE_SVG` — `test/frontend.js:4613` — 真 Chromium 里跑前端源码切片断言。
- `LOOK_SRC` — `test/frontend.js:4741` — 真 Chromium 里跑前端源码切片断言。
- `HL_SRC` — `test/frontend.js:4941` — 真 Chromium 里跑前端源码切片断言。
- `MENU_SRC` — `test/frontend.js:5064` — 真 Chromium 里跑前端源码切片断言。
- `esc` — `test/frontend.js:6547` — 真 Chromium 里跑前端源码切片断言。
- `toast` — `test/frontend.js:6549` — 真 Chromium 里跑前端源码切片断言。
- `hubMatch` — `test/frontend.js:6551` — 真 Chromium 里跑前端源码切片断言。
- `amPlatformOwner` — `test/frontend.js:6553` — 真 Chromium 里跑前端源码切片断言。
- `renderHubBody` — `test/frontend.js:6554` — 真 Chromium 里跑前端源码切片断言。
- `MASK_CHECKS` — `test/frontend.js:6929` — 真 Chromium 里跑前端源码切片断言。
- `SKEG_SRC` — `test/frontend.js:6959` — 真 Chromium 里跑前端源码切片断言。
- `SKEG_DOCS` — `test/frontend.js:6970` — 真 Chromium 里跑前端源码切片断言。
- `LANE_STATE_SRC` — `test/frontend.js:7074` — 真 Chromium 里跑前端源码切片断言。
- `LANE_SRC` — `test/frontend.js:7080` — 真 Chromium 里跑前端源码切片断言。
- `HIST_MIN_SRC` — `test/frontend.js:7087` — 真 Chromium 里跑前端源码切片断言。
- `laneHtml` — `test/frontend.js:7095` — 真 Chromium 里跑前端源码切片断言。
- `ASIDE_SRC` — `test/frontend.js:7112` — 真 Chromium 里跑前端源码切片断言。
- `HIST_RSZ_SRC` — `test/frontend.js:7119` — 真 Chromium 里跑前端源码切片断言。
- `PROJ_SRC` — `test/frontend.js:7611` — 真 Chromium 里跑前端源码切片断言。
- `APP03_MERGE` — `test/frontend.js:7698` — 真 Chromium 里跑前端源码切片断言。
- `TH_PIC` — `test/frontend.js:7796` — 真 Chromium 里跑前端源码切片断言。
- `mkNode` — `test/frontend.js:9071` — 真 Chromium 里跑前端源码切片断言。
- `ok` — `test/skill-guard.js:41` — 技能静态安检：三十余条规则分三档，强装留档。
- `both` — `test/skill-guard.js:53` — 技能静态安检：三十余条规则分三档，强装留档。
- `mkSrc` — `test/skill-guard.js:184` — 技能静态安检：三十余条规则分三档，强装留档。
- `tryInstall` — `test/skill-guard.js:195` — 技能静态安检：三十余条规则分三档，强装留档。
- `S` — `test/skill-guard.js:48` — 技能静态安检：三十余条规则分三档，强装留档。
- `rules` — `test/skill-guard.js:49` — 技能静态安检：三十余条规则分三档，强装留档。
- `hit` — `test/skill-guard.js:50` — 技能静态安检：三十余条规则分三档，强装留档。
- `installedDir` — `test/skill-guard.js:193` — 技能静态安检：三十余条规则分三档，强装留档。
- `ok` — `test/term-image.js:27` — 终端画产出图。
- `eq` — `test/term-image.js:31` — 终端画产出图。
- `ok` — `test/totp.js:21` — TOTP 二次验证（RFC 4226/6238）。
- `eq` — `test/totp.js:25` — TOTP 二次验证（RFC 4226/6238）。
- `ok` — `test/md-tty.js:25` — 终端 Markdown 渲染。
- `eq` — `test/md-tty.js:29` — 终端 Markdown 渲染。
- `plain` — `test/md-tty.js:32` — 终端 Markdown 渲染。
- `colored` — `test/md-tty.js:37` — 终端 Markdown 渲染。
- `streamed` — `test/md-tty.js:42` — 终端 Markdown 渲染。
- `ok` — `test/ops.js:42` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `eq` — `test/ops.js:46` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `readLog` — `test/ops.js:49` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `wipeLog` — `test/ops.js:53` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ok` — `test/systemone.js:29` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `eq` — `test/systemone.js:33` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `rest` — `test/systemone.js:265` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `src` — `test/systemone.js:34` — Jev 判断模型：是非/单选/打分与确定度门槛。
- `ok` — `test/auth-2fa.js:26` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `why` — `test/auth-2fa.js:31` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `rewindStep` — `test/auth-2fa.js:35` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `hit` — `test/auth-2fa.js:165` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ck` — `test/auth-2fa.js:189` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `fakeLangfuse` — `test/trace.js:53` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `main` — `test/trace.js:82` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `ok` — `test/trace.js:43` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `eq` — `test/trace.js:47` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `wait` — `test/trace.js:80` — 本地 Trace 账本 + 可选 Langfuse 上报。
- `ok` — `test/relay.js:60` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `eq` — `test/relay.js:68` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `threw` — `test/relay.js:73` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `bill` — `test/relay.js:79` — OpenAI 兼容中转站：虚拟 Key、按型号计价、加权选渠。
- `ok` — `test/session-search.js:30` — 任务历史检索：正文、产出文件名、意思相近。
- `eq` — `test/session-search.js:34` — 任务历史检索：正文、产出文件名、意思相近。
- `sess` — `test/session-search.js:37` — 任务历史检索：正文、产出文件名、意思相近。
- `one` — `test/session-search.js:55` — 任务历史检索：正文、产出文件名、意思相近。
- `titles` — `test/session-search.js:56` — 任务历史检索：正文、产出文件名、意思相近。
- `ok` — `test/cli-live.js:27` — 终端与网页直播桥。
- `eq` — `test/cli-live.js:31` — 终端与网页直播桥。
- `metaOf` — `test/cli-live.js:33` — 终端与网页直播桥。
- `writeMeta` — `test/cli-live.js:34` — 终端与网页直播桥。
- `rowOf` — `test/cli-live.js:39` — 终端与网页直播桥。
- `call` — `test/tenant.js:138` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `login` — `test/tenant.js:162` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ok` — `test/tenant.js:41` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `eq` — `test/tenant.js:45` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `approvalScope` — `test/tenant.js:67` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `memScope` — `test/tenant.js:83` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `isPlatformOwner` — `test/tenant.js:107` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `fakeIO` — `test/cli-ask.js:21` — 终端回答 agent 提问。
- `run` — `test/cli-ask.js:37` — 终端回答 agent 提问。
- `ok` — `test/cli-args.js:24` — CLI 参数表。
- `eq` — `test/cli-args.js:28` — CLI 参数表。
- `say` — `test/cli-args.js:31` — CLI 参数表。
- `clean` — `test/cli-args.js:32` — CLI 参数表。
- `fakeChrome` — `test/cdp.js:31` — Chrome DevTools Protocol 真浏览器控制。
- `ok` — `test/cdp.js:21` — Chrome DevTools Protocol 真浏览器控制。
- `eq` — `test/cdp.js:22` — Chrome DevTools Protocol 真浏览器控制。
- `listen` — `test/cdp.js:24` — Chrome DevTools Protocol 真浏览器控制。
- `wsFrame` — `test/cdp.js:25` — Chrome DevTools Protocol 真浏览器控制。
- `ok` — `test/eval.js:36` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `src` — `test/eval.js:41` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `fresh` — `test/eval.js:42` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ok` — `test/gen-cache.js:34` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `eq` — `test/gen-cache.js:38` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `e2e` — `test/gen-cache.js:188` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `finish` — `test/gen-cache.js:376` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `resolveFile` — `test/gen-cache.js:56` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `K` — `test/gen-cache.js:67` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `json` — `test/gen-cache.js:185` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `bin` — `test/gen-cache.js:186` — 生成结果内容寻址缓存，同一格重跑不二次扣费。
- `ok` — `test/shot-history.js:20` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `freshBoard` — `test/shot-history.js:34` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `abs` — `test/shot-history.js:27` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `write` — `test/shot-history.js:28` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `read` — `test/shot-history.js:29` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `board` — `test/shot-history.js:30` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `saveBoard` — `test/shot-history.js:31` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `shotOf` — `test/shot-history.js:32` — 分镜版本对象库：按 sha256 留首帧/视频/配音，整镜恢复。
- `ok` — `test/lanes.js:24` — 办公/工程两条泳道。
- `eq` — `test/lanes.js:28` — 办公/工程两条泳道。
- `backendId` — `test/lanes.js:71` — 办公/工程两条泳道。
- `reapStaleTempHomes` — `test/e2e.js:47` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `bootRealServer` — `test/e2e.js:82` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `makeFakeLLM` — `test/e2e.js:127` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testAgentPipeline` — `test/e2e.js:216` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testOfficeLibs` — `test/e2e.js:243` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPreviewExtract` — `test/e2e.js:267` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testNodeSyntaxPrecheck` — `test/e2e.js:348` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testShellGlobCompat` — `test/e2e.js:373` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testMissingBinHintWired` — `test/e2e.js:416` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testSessionFileLayout` — `test/e2e.js:445` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCssTokenGate` — `test/e2e.js:501` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testMotionGate` — `test/e2e.js:660` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testSurfaceLayerGate` — `test/e2e.js:739` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testVerdictGate` — `test/e2e.js:838` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDocLinkGate` — `test/e2e.js:884` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testImageWatermarkGate` — `test/e2e.js:993` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testVideoWatermarkGate` — `test/e2e.js:1064` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testMediaImageInputGate` — `test/e2e.js:1138` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testVideoProtocols` — `test/e2e.js:1333` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testMediaKeyHygiene` — `test/e2e.js:1539` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCliMode` — `test/e2e.js:1629` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDeliverableGate` — `test/e2e.js:1846` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testContextBudget` — `test/e2e.js:1884` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCtxMeterWiring` — `test/e2e.js:1936` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testToolPairRepair` — `test/e2e.js:1964` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testFetchRetry` — `test/e2e.js:2045` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCheckPageConsole` — `test/e2e.js:2172` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testLookAtImage` — `test/e2e.js:2203` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testAnthropicEndpointAgreement` — `test/e2e.js:2324` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `mkPlugin` — `test/e2e.js:2386` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPluginManifest` — `test/e2e.js:2400` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPluginComponentIsolation` — `test/e2e.js:2450` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPluginMcpRuntime` — `test/e2e.js:2511` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPluginSkillsIntegration` — `test/e2e.js:2570` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `startFakeMcpHttp` — `test/e2e.js:2621` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testMcpStreamableHttp` — `test/e2e.js:2661` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testMcpManagerLifecycle` — `test/e2e.js:2682` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `runChild` — `test/e2e.js:2739` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testFrontendSvgFigures` — `test/e2e.js:2765` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDockerDeploy` — `test/e2e.js:2791` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testNodeSuite` — `test/e2e.js:2809` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testAdminConsoleUI` — `test/e2e.js:2819` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDesktopAppIdentity` — `test/e2e.js:2838` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDefaultSkillsManifest` — `test/e2e.js:2941` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCron` — `test/e2e.js:2993` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testSchedulerRuntime` — `test/e2e.js:3029` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPermissionModes` — `test/e2e.js:3091` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testEventLedgerParity` — `test/e2e.js:3218` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testEvolveLoop` — `test/e2e.js:3233` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testEvolveRecency` — `test/e2e.js:3388` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testEvolvePromptBudget` — `test/e2e.js:3491` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testAgentPromptDrift` — `test/e2e.js:3622` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testTaskDirLifecycle` — `test/e2e.js:3717` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCodingTools` — `test/e2e.js:3791` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDeliverableQuality` — `test/e2e.js:3951` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDiagramRepair` — `test/e2e.js:4147` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testEvolveCaliberAndSpread` — `test/e2e.js:4247` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testMemoryLayer` — `test/e2e.js:4364` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testMemoryNearDup` — `test/e2e.js:4547` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCommandGate` — `test/e2e.js:4598` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testAccountStore` — `test/e2e.js:4665` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCreditsGate` — `test/e2e.js:4735` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCachedLedger` — `test/e2e.js:4807` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testRenameLogin` — `test/e2e.js:4866` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testAvatarRules` — `test/e2e.js:4917` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testJsonStore` — `test/e2e.js:4937` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testImSessionStore` — `test/e2e.js:4975` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testForcedWrapUp` — `test/e2e.js:5073` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCollectSources` — `test/e2e.js:5151` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testLlmStreamFailures` — `test/e2e.js:5184` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testLeakedToolCallRescue` — `test/e2e.js:5224` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testFetchUrlShapes` — `test/e2e.js:5260` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testParallelToolBatch` — `test/e2e.js:5345` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPathSafety` — `test/e2e.js:5415` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDesktopPet` — `test/e2e.js:5428` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testConnectorToggleAndTools` — `test/e2e.js:5564` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testScheduleRunTrace` — `test/e2e.js:5777` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testMcpFailureReason` — `test/e2e.js:5974` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPetSprites` — `test/e2e.js:6005` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `questionVsWorkProblems` — `test/e2e.js:6117` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `capturePrompts` — `test/e2e.js:6151` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPromptQuestionVsWork` — `test/e2e.js:6178` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPromptNoAskContradiction` — `test/e2e.js:6229` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `enginePathProblems` — `test/e2e.js:6269` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `e2ePng` — `test/e2e.js:6298` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testFilePathRouting` — `test/e2e.js:6360` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testLibraryOutputsTruth` — `test/e2e.js:6518` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testLibraryTurnAnchor` — `test/e2e.js:6731` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testSweepApi` — `test/e2e.js:6863` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testSessionSearchLive` — `test/e2e.js:6987` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDecideLive` — `test/e2e.js:7101` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testConfigExternalEdit` — `test/e2e.js:7195` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testKeyGuard` — `test/e2e.js:7275` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPortCollision` — `test/e2e.js:7461` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testLocalEngineConnect` — `test/e2e.js:7558` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testAskUser` — `test/e2e.js:7642` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testOutputFilesRecency` — `test/e2e.js:7718` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCheckpoints` — `test/e2e.js:7829` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testBackupRoundTrip` — `test/e2e.js:7889` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testThumbPng` — `test/e2e.js:8064` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testThumbPool` — `test/e2e.js:8263` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `main` — `test/e2e.js:8503` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCanvasCreativeLineage` — `test/e2e.js:8692` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCanvasMissingAssets` — `test/e2e.js:8737` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCanvasThumb` — `test/e2e.js:8873` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testGoalOnLocalEngine` — `test/e2e.js:8948` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDetectCache` — `test/e2e.js:9000` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testStreamRender` — `test/e2e.js:9033` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testEmbedFailoverResilience` — `test/e2e.js:9062` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testThinkingSwitch` — `test/e2e.js:9207` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testOnboardingWizardApi` — `test/e2e.js:9349` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testThinkingSettingsApi` — `test/e2e.js:9516` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testFilesEmitter` — `test/e2e.js:9596` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testHeavyTools` — `test/e2e.js:9815` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testOutputOwnership` — `test/e2e.js:10003` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testEngineToolBridge` — `test/e2e.js:10121` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testEngineSecurityGuard` — `test/e2e.js:10289` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testReadmeFrontGate` — `test/e2e.js:10420` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `keySourcesCheck` — `test/e2e.js:10528` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `packagingCheck` — `test/e2e.js:10589` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `packageAssetDrift` — `test/e2e.js:10630` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `noticeDrift` — `test/e2e.js:10671` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testNoticeCoverage` — `test/e2e.js:10681` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testIntranet` — `test/e2e.js:10722` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `styleDirectionDrift` — `test/e2e.js:10931` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testStyleDirection` — `test/e2e.js:10993` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `shortDramaDrift` — `test/e2e.js:11050` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testShortDrama` — `test/e2e.js:11117` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `releasePipelineDrift` — `test/e2e.js:11195` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testReleasePipeline` — `test/e2e.js:11305` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPackageAssetDrift` — `test/e2e.js:11367` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `nestedRoutes` — `test/e2e.js:11405` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testNoNestedRoutes` — `test/e2e.js:11431` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testPackagingAndDemoGate` — `test/e2e.js:11457` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testAdminModelsPage` — `test/e2e.js:11496` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testKeySourcesGate` — `test/e2e.js:11599` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testI18n` — `test/e2e.js:11625` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testConnectorsAndExperts` — `test/e2e.js:11769` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testOutputArrivalStatic` — `test/e2e.js:11937` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDemoReadmeWire` — `test/e2e.js:11979` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDemoTiming` — `test/e2e.js:12043` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testLookPrefsStatic` — `test/e2e.js:12083` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testUiNoRawMarkdown` — `test/e2e.js:12168` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testEngineStoppedSurfacing` — `test/e2e.js:12234` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testEngineContextParity` — `test/e2e.js:12318` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testFeedbackAndUsage` — `test/e2e.js:12451` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testImInboundMedia` — `test/e2e.js:12657` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testImCredentialGuard` — `test/e2e.js:12814` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testUpdaterVersions` — `test/e2e.js:12904` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCompletionGate` — `test/e2e.js:13040` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testStepLines` — `test/e2e.js:13154` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testStepLinesWired` — `test/e2e.js:13231` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testLarkCliParse` — `test/e2e.js:13276` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testLarkSecretNotClobbered` — `test/e2e.js:13347` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testOutNameKeepsExt` — `test/e2e.js:13460` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testSkillRenameKeepsAssets` — `test/e2e.js:13494` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testWindowsLaunch` — `test/e2e.js:13572` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testSessionCacheReload` — `test/e2e.js:13717` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testRunOwnership` — `test/e2e.js:13842` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testSessionIndex` — `test/e2e.js:13953` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testCanvasDataLoss` — `test/e2e.js:14152` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testUpgradeMigration` — `test/e2e.js:14270` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `_testUpgradeMigrationBody` — `test/e2e.js:14280` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDramaAssets` — `test/e2e.js:14420` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDramaPipeline` — `test/e2e.js:14523` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDramaCompose` — `test/e2e.js:14806` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDramaCast` — `test/e2e.js:15347` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDramaShotRefs` — `test/e2e.js:15580` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDramaVoice` — `test/e2e.js:15731` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDramaBoardExpand` — `test/e2e.js:15906` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `testDramaBoardWriteback` — `test/e2e.js:16127` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `goodManifest` — `test/e2e.js:2397` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `skillBody` — `test/e2e.js:2398` — 最大端到端套件，覆盖 agent、办公库、安全、进化、画布等。
- `ok` — `test/quota.js:38` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `eq` — `test/quota.js:42` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `reset` — `test/quota.js:46` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `table` — `test/quota.js:54` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `actor` — `test/quota.js:57` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `burn` — `test/quota.js:61` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `ok_silent` — `test/quota.js:260` — 付费能力额度闸：搜索/生图/生视频/配音/转写按次限额与预扣。
- `hits` — `test/icons.js:45` — 图标资源。
- `regexCanStart` — `test/icons.js:53` — 图标资源。
- `stripComments` — `test/icons.js:66` — 图标资源。
- `stripHtmlComments` — `test/icons.js:113` — 图标资源。
- `stripEmojiRegions` — `test/icons.js:124` — 图标资源。
- `scan` — `test/icons.js:132` — 图标资源。
- `ok` — `test/icons.js:27` — 图标资源。
- `eq` — `test/icons.js:31` — 图标资源。
- `allHits` — `test/icons.js:162` — 图标资源。
- `accepts` — `test/icons.js:303` — 图标资源。
- `call` — `test/admin-ui.js:137` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `openAdmin` — `test/admin-ui.js:162` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ok` — `test/admin-ui.js:77` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `GOTO` — `test/admin-ui.js:192` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ok` — `test/worktree.js:24` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `mkRepo` — `test/worktree.js:37` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `eq` — `test/worktree.js:28` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `store` — `test/worktree.js:32` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `git` — `test/worktree.js:33` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `out` — `test/worktree.js:34` — 两条任务撞同一 git 仓库时自动开 owb/<会话> 分身，绝不自动合回。
- `req` — `test/remote.js:65` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ok` — `test/remote.js:38` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `eq` — `test/remote.js:42` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ck` — `test/remote.js:92` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `rest` — `test/media-models.js:231` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `asrChecks` — `test/media-models.js:528` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `done` — `test/media-models.js:649` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `ok` — `test/media-models.js:29` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `eq` — `test/media-models.js:33` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `legacy` — `test/media-models.js:42` — 生图/生视频/配音/转写/看图多模型与五家视频协议识别。
- `ok` — `test/chat-models.js:32` — 对话渠道表、选型、健康账本。
- `eq` — `test/chat-models.js:36` — 对话渠道表、选型、健康账本。
- `legacy` — `test/chat-models.js:46` — 对话渠道表、选型、健康账本。
- `stripComments` — `test/repo-hygiene.js:77` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `stripTemplates` — `test/repo-hygiene.js:93` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `assertStripSane` — `test/repo-hygiene.js:101` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ignoredSkills` — `test/repo-hygiene.js:112` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `skillNameLiterals` — `test/repo-hygiene.js:132` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `shippedSkills` — `test/repo-hygiene.js:259` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `namingHits` — `test/repo-hygiene.js:340` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `scanTargets` — `test/repo-hygiene.js:44` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `DECLARED_DEPS` — `test/repo-hygiene.js:50` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `pkgOf` — `test/repo-hygiene.js:59` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ok` — `test/repo-hygiene.js:66` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `HAS_GIT` — `test/repo-hygiene.js:182` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `unshipped` — `test/repo-hygiene.js:272` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `namingAllowed` — `test/repo-hygiene.js:336` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `fakeIO` — `test/cli-approve.js:21` — 终端/手机审批危险操作。
- `run` — `test/cli-approve.js:36` — 终端/手机审批危险操作。
- `ok` — `test/cli-attach.js:28` — CLI 带文件和图片。
- `eq` — `test/cli-attach.js:32` — CLI 带文件和图片。
- `fakeFs` — `test/cli-attach.js:129` — CLI 带文件和图片。
- `runner` — `test/cli-attach.js:267` — CLI 带文件和图片。
- `P` — `test/cli-attach.js:46` — CLI 带文件和图片。
- `call` — `test/prefs.js:298` — 按账号存储的引擎、思考档、外观偏好。
- `slice` — `test/prefs.js:468` — 按账号存储的引擎、思考档、外观偏好。
- `runSeedCopy` — `test/prefs.js:492` — 按账号存储的引擎、思考档、外观偏好。
- `runSourcePins` — `test/prefs.js:569` — 按账号存储的引擎、思考档、外观偏好。
- `runConfigGates` — `test/prefs.js:763` — 按账号存储的引擎、思考档、外观偏好。
- `ok` — `test/prefs.js:47` — 按账号存储的引擎、思考档、外观偏好。
- `eq` — `test/prefs.js:51` — 按账号存储的引擎、思考档、外观偏好。
- `makeConfig` — `test/prefs.js:55` — 按账号存储的引擎、思考档、外观偏好。
- `flat` — `test/prefs.js:156` — 按账号存储的引擎、思考档、外观偏好。
- `ok` — `test/agent-loop.js:23` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `eq` — `test/agent-loop.js:27` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `key` — `test/agent-loop.js:34` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `hist` — `test/agent-loop.js:35` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `call` — `test/agent-loop.js:37` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `step` — `test/agent-loop.js:39` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `cycle` — `test/agent-loop.js:40` — 自动化测试套件。失败退出码非 0，由 test/all.js 汇总。
- `ok` — `test/doctor.js:25` — openworkbuddy doctor 体检。
- `eq` — `test/doctor.js:29` — openworkbuddy doctor 体检。
- `kt` — `test/doctor.js:147` — openworkbuddy doctor 体检。
- `renderSecurityPane` — `public/js/app-06.js:1` — 前端评测、自进化、备份。
- `renderTwoFactorBox` — `public/js/app-06.js:370` — 前端评测、自进化、备份。
- `renderShortcutsPane` — `public/js/app-06.js:548` — 前端评测、自进化、备份。
- `renderEvolvePane` — `public/js/app-06.js:637` — 前端评测、自进化、备份。
- `renderLookPane` — `public/js/app-06.js:776` — 前端评测、自进化、备份。
- `renderAboutPane` — `public/js/app-06.js:807` — 前端评测、自进化、备份。
- `modelReady` — `public/js/app-02.js:6` — 前端任务直播、过程卡、模式切换。
- `renderModelMenu` — `public/js/app-02.js:12` — 前端任务直播、过程卡、模式切换。
- `renderGoalCard` — `public/js/app-02.js:60` — 前端任务直播、过程卡、模式切换。
- `setWorkspaceDir` — `public/js/app-02.js:100` — 前端任务直播、过程卡、模式切换。
- `renderWsMenu` — `public/js/app-02.js:107` — 前端任务直播、过程卡、模式切换。
- `modeInfo` — `public/js/app-02.js:146` — 前端任务直播、过程卡、模式切换。
- `setMode` — `public/js/app-02.js:147` — 前端任务直播、过程卡、模式切换。
- `loadExecModes` — `public/js/app-02.js:158` — 前端任务直播、过程卡、模式切换。
- `attachKind` — `public/js/app-02.js:194` — 前端任务直播、过程卡、模式切换。
- `markerName` — `public/js/app-02.js:203` — 前端任务直播、过程卡、模式切换。
- `attachmentOrder` — `public/js/app-02.js:204` — 前端任务直播、过程卡、模式切换。
- `removeAttachmentMarker` — `public/js/app-02.js:213` — 前端任务直播、过程卡、模式切换。
- `insertAttachmentMarker` — `public/js/app-02.js:223` — 前端任务直播、过程卡、模式切换。
- `addAttachChip` — `public/js/app-02.js:227` — 前端任务直播、过程卡、模式切换。
- `bytesToB64` — `public/js/app-02.js:267` — 前端任务直播、过程卡、模式切换。
- `uploadBytes` — `public/js/app-02.js:273` — 前端任务直播、过程卡、模式切换。
- `uploadFiles` — `public/js/app-02.js:294` — 前端任务直播、过程卡、模式切换。
- `stampName` — `public/js/app-02.js:314` — 前端任务直播、过程卡、模式切换。
- `uploadText` — `public/js/app-02.js:329` — 前端任务直播、过程卡、模式切换。
- `composeOutgoing` — `public/js/app-02.js:348` — 前端任务直播、过程卡、模式切换。
- `insertAtCursor` — `public/js/app-02.js:395` — 前端任务直播、过程卡、模式切换。
- `pastedName` — `public/js/app-02.js:409` — 前端任务直播、过程卡、模式切换。
- `laneOfSession` — `public/js/app-02.js:452` — 前端任务直播、过程卡、模式切换。
- `renderLaneTabs` — `public/js/app-02.js:456` — 前端任务直播、过程卡、模式切换。
- `refreshLanes` — `public/js/app-02.js:473` — 前端任务直播、过程卡、模式切换。
- `cliLiveKey` — `public/js/app-02.js:507` — 前端任务直播、过程卡、模式切换。
- `pollCliLive` — `public/js/app-02.js:508` — 前端任务直播、过程卡、模式切换。
- `openCliLive` — `public/js/app-02.js:528` — 前端任务直播、过程卡、模式切换。
- `stopCliWatch` — `public/js/app-02.js:558` — 前端任务直播、过程卡、模式切换。
- `finishCliWatch` — `public/js/app-02.js:564` — 前端任务直播、过程卡、模式切换。
- `pollCliAsk` — `public/js/app-02.js:592` — 前端任务直播、过程卡、模式切换。
- `interjectCli` — `public/js/app-02.js:634` — 前端任务直播、过程卡、模式切换。
- `projectSessions` — `public/js/app-02.js:655` — 前端任务直播、过程卡、模式切换。
- `histMatch` — `public/js/app-02.js:681` — 前端任务直播、过程卡、模式切换。
- `histSearchSoon` — `public/js/app-02.js:687` — 前端任务直播、过程卡、模式切换。
- `histHitRow` — `public/js/app-02.js:710` — 前端任务直播、过程卡、模式切换。
- `renderHistory` — `public/js/app-02.js:725` — 前端任务直播、过程卡、模式切换。
- `jumpToTurn` — `public/js/app-02.js:822` — 前端任务直播、过程卡、模式切换。
- `openSession` — `public/js/app-02.js:839` — 前端任务直播、过程卡、模式切换。
- `refreshProjects` — `public/js/app-02.js:931` — 前端任务直播、过程卡、模式切换。
- `renderProjects` — `public/js/app-02.js:941` — 前端任务直播、过程卡、模式切换。
- `syncPlaceholder` — `public/js/app-02.js:1011` — 前端任务直播、过程卡、模式切换。
- `hasDraft` — `public/js/app-02.js:1018` — 前端任务直播、过程卡、模式切换。
- `syncSendBtn` — `public/js/app-02.js:1023` — 前端任务直播、过程卡、模式切换。
- `updateSendUI` — `public/js/app-02.js:1040` — 前端任务直播、过程卡、模式切换。
- `renderQueueBar` — `public/js/app-02.js:1047` — 前端任务直播、过程卡、模式切换。
- `bindComposer` — `public/js/app-02.js:1076` — 前端任务直播、过程卡、模式切换。
- `queueText` — `public/js/app-02.js:1087` — 前端任务直播、过程卡、模式切换。
- `drainQueue` — `public/js/app-02.js:1092` — 前端任务直播、过程卡、模式切换。
- `interjectText` — `public/js/app-02.js:1100` — 前端任务直播、过程卡、模式切换。
- `interject` — `public/js/app-02.js:1116` — 前端任务直播、过程卡、模式切换。
- `stopTask` — `public/js/app-02.js:1123` — 前端任务直播、过程卡、模式切换。
- `send` — `public/js/app-02.js:1132` — 前端任务直播、过程卡、模式切换。
- `doSend` — `public/js/app-02.js:1156` — 前端任务直播、过程卡、模式切换。
- `makeRecCounter` — `public/js/app-02.js:1173` — 前端任务直播、过程卡、模式切换。
- `pumpStream` — `public/js/app-02.js:1187` — 前端任务直播、过程卡、模式切换。
- `keepAttached` — `public/js/app-02.js:1211` — 前端任务直播、过程卡、模式切换。
- `probeRunning` — `public/js/app-02.js:1238` — 前端任务直播、过程卡、模式切换。
- `endRun` — `public/js/app-02.js:1250` — 前端任务直播、过程卡、模式切换。
- `notifyRunDone` — `public/js/app-02.js:1260` — 前端任务直播、过程卡、模式切换。
- `bumpDoneWhileAway` — `public/js/app-02.js:1288` — 前端任务直播、过程卡、模式切换。
- `reattachRunning` — `public/js/app-02.js:1305` — 前端任务直播、过程卡、模式切换。
- `runTurn` — `public/js/app-02.js:1337` — 前端任务直播、过程卡、模式切换。
- `openHub` — `public/js/app-02.js:1387` — 前端任务直播、过程卡、模式切换。
- `openModal` — `public/js/app-02.js:1395` — 前端任务直播、过程卡、模式切换。
- `canonAccel` — `public/js/app-02.js:1435` — 前端任务直播、过程卡、模式切换。
- `accelFromEvent` — `public/js/app-02.js:1445` — 前端任务直播、过程卡、模式切换。
- `accelDisplay` — `public/js/app-02.js:1455` — 前端任务直播、过程卡、模式切换。
- `toast` — `public/js/app-02.js:1470` — 前端任务直播、过程卡、模式切换。
- `toggleAppFullscreen` — `public/js/app-02.js:1489` — 前端任务直播、过程卡、模式切换。
- `navTask` — `public/js/app-02.js:1496` — 前端任务直播、过程卡、模式切换。
- `openChatSearch` — `public/js/app-02.js:1552` — 前端任务直播、过程卡、模式切换。
- `closeChatSearch` — `public/js/app-02.js:1572` — 前端任务直播、过程卡、模式切换。
- `runChatSearch` — `public/js/app-02.js:1580` — 前端任务直播、过程卡、模式切换。
- `csNav` — `public/js/app-02.js:1593` — 前端任务直播、过程卡、模式切换。
- `pollApprovals` — `public/js/app-02.js:1605` — 前端任务直播、过程卡、模式切换。
- `syncPermLabel` — `public/js/app-02.js:1667` — 前端任务直播、过程卡、模式切换。
- `loadPermModes` — `public/js/app-02.js:1673` — 前端任务直播、过程卡、模式切换。
- `setPermMode` — `public/js/app-02.js:1695` — 前端任务直播、过程卡、模式切换。
- `shrinkImage` — `public/js/app-02.js:1718` — 前端任务直播、过程卡、模式切换。
- `avatarEditorHtml` — `public/js/app-02.js:1748` — 前端任务直播、过程卡、模式切换。
- `avatarTab` — `public/js/app-02.js:1779` — 前端任务直播、过程卡、模式切换。
- `bindAvatarEditor` — `public/js/app-02.js:1787` — 前端任务直播、过程卡、模式切换。
- `renderProfile` — `public/js/app-02.js:1865` — 前端任务直播、过程卡、模式切换。
- `renderUserChip` — `public/js/app-02.js:1927` — 前端任务直播、过程卡、模式切换。
- `lookRead` — `public/js/app-02.js:1943` — 前端任务直播、过程卡、模式切换。
- `lookWrite` — `public/js/app-02.js:1944` — 前端任务直播、过程卡、模式切换。
- `getTheme` — `public/js/app-02.js:1947` — 前端任务直播、过程卡、模式切换。
- `applyTheme` — `public/js/app-02.js:1948` — 前端任务直播、过程卡、模式切换。
- `setTheme` — `public/js/app-02.js:1953` — 前端任务直播、过程卡、模式切换。
- `lookGet` — `public/js/app-02.js:1963` — 前端任务直播、过程卡、模式切换。
- `applyLook` — `public/js/app-02.js:1964` — 前端任务直播、过程卡、模式切换。
- `setLook` — `public/js/app-02.js:1971` — 前端任务直播、过程卡、模式切换。
- `closeUserMenu` — `public/js/app-02.js:1976` — 前端任务直播、过程卡、模式切换。
- `toggleUserMenu` — `public/js/app-02.js:1977` — 前端任务直播、过程卡、模式切换。
- `openUserMenu` — `public/js/app-02.js:1978` — 前端任务直播、过程卡、模式切换。
- `openMdFileLink` — `public/js/app-02.js:2049` — 前端任务直播、过程卡、模式切换。
- `flashBtn` — `public/js/app-02.js:2101` — 前端任务直播、过程卡、模式切换。
- `postJson` — `public/js/app-02.js:2118` — 前端任务直播、过程卡、模式切换。
- `browserDownload` — `public/js/app-02.js:2147` — 前端任务直播、过程卡、模式切换。
- `saveInlineFile` — `public/js/app-02.js:2161` — 前端任务直播、过程卡、模式切换。
- `closeFigZoom` — `public/js/app-02.js:2180` — 前端任务直播、过程卡、模式切换。
- `openFigZoom` — `public/js/app-02.js:2185` — 前端任务直播、过程卡、模式切换。
- `checkUpdate` — `public/js/app-02.js:2249` — 前端任务直播、过程卡、模式切换。
- `plusInsert` — `public/js/app-02.js:2287` — 前端任务直播、过程卡、模式切换。
- `plusClose` — `public/js/app-02.js:2296` — 前端任务直播、过程卡、模式切换。
- `plusItem` — `public/js/app-02.js:2302` — 前端任务直播、过程卡、模式切换。
- `renderPlusRoot` — `public/js/app-02.js:2317` — 前端任务直播、过程卡、模式切换。
- `plusFit` — `public/js/app-02.js:2330` — 前端任务直播、过程卡、模式切换。
- `openPlusSub` — `public/js/app-02.js:2336` — 前端任务直播、过程卡、模式切换。
- `dragHasPayload` — `public/js/app-02.js:369` — 前端任务直播、过程卡、模式切换。
- `closeModal` — `public/js/app-02.js:1371` — 前端任务直播、过程卡、模式切换。
- `plusEmpty` — `public/js/app-02.js:2312` — 前端任务直播、过程卡、模式切换。
- `plusHead` — `public/js/app-02.js:2313` — 前端任务直播、过程卡、模式切换。
- `plusFoot` — `public/js/app-02.js:2314` — 前端任务直播、过程卡、模式切换。
- `updateEvalView` — `public/js/app-04.js:1` — 前端专家技能连接器广场。
- `openEvalDetail` — `public/js/app-04.js:62` — 前端专家技能连接器广场。
- `libTaskOf` — `public/js/app-04.js:154` — 前端专家技能连接器广场。
- `libOwnerTask` — `public/js/app-04.js:179` — 前端专家技能连接器广场。
- `libTurnOf` — `public/js/app-04.js:193` — 前端专家技能连接器广场。
- `libIcon` — `public/js/app-04.js:198` — 前端专家技能连接器广场。
- `libSize` — `public/js/app-04.js:203` — 前端专家技能连接器广场。
- `libKindOk` — `public/js/app-04.js:216` — 前端专家技能连接器广场。
- `libKindOf` — `public/js/app-04.js:221` — 前端专家技能连接器广场。
- `libUrl` — `public/js/app-04.js:232` — 前端专家技能连接器广场。
- `libTimeBucket` — `public/js/app-04.js:246` — 前端专家技能连接器广场。
- `libGroupFiles` — `public/js/app-04.js:256` — 前端专家技能连接器广场。
- `libWhen` — `public/js/app-04.js:272` — 前端专家技能连接器广场。
- `libMark` — `public/js/app-04.js:284` — 前端专家技能连接器广场。
- `libRowHtml` — `public/js/app-04.js:302` — 前端专家技能连接器广场。
- `libPerRow` — `public/js/app-04.js:331` — 前端专家技能连接器广场。
- `showLibPrev` — `public/js/app-04.js:342` — 前端专家技能连接器广场。
- `renderLibPage` — `public/js/app-04.js:347` — 前端专家技能连接器广场。
- `libTasksHtml` — `public/js/app-04.js:595` — 前端专家技能连接器广场。
- `libSearchHtml` — `public/js/app-04.js:636` — 前端专家技能连接器广场。
- `renderLibPreview` — `public/js/app-04.js:669` — 前端专家技能连接器广场。
- `renderHubPage` — `public/js/app-04.js:867` — 前端专家技能连接器广场。
- `renderHubBody` — `public/js/app-04.js:908` — 前端专家技能连接器广场。
- `renderHubExperts` — `public/js/app-04.js:920` — 前端专家技能连接器广场。
- `renderHubEditor` — `public/js/app-04.js:1027` — 前端专家技能连接器广场。
- `skillExamples` — `public/js/app-04.js:1190` — 前端专家技能连接器广场。
- `renderHubSkills` — `public/js/app-04.js:1219` — 前端专家技能连接器广场。
- `renderHubSkillEditor` — `public/js/app-04.js:1377` — 前端专家技能连接器广场。
- `showScanGate` — `public/js/app-04.js:1434` — 前端专家技能连接器广场。
- `renderHubPlugins` — `public/js/app-04.js:1469` — 前端专家技能连接器广场。
- `fmtBytes` — `public/js/app-04.js:865` — 前端专家技能连接器广场。
- `hubMatch` — `public/js/app-04.js:917` — 前端专家技能连接器广场。
- `renderCtxMeter` — `public/js/app-01.js:52` — 前端会话、消息、文件面板。
- `resetCtxMeter` — `public/js/app-01.js:86` — 前端会话、消息、文件面板。
- `procNote` — `public/js/app-01.js:96` — 前端会话、消息、文件面板。
- `setBusySendMode` — `public/js/app-01.js:130` — 前端会话、消息、文件面板。
- `saveSessions` — `public/js/app-01.js:147` — 前端会话、消息、文件面板。
- `esc` — `public/js/app-01.js:164` — 前端会话、消息、文件面板。
- `prettyUrl` — `public/js/app-01.js:179` — 前端会话、消息、文件面板。
- `autoLinkUrls` — `public/js/app-01.js:203` — 前端会话、消息、文件面板。
- `getList` — `public/js/app-01.js:227` — 前端会话、消息、文件面板。
- `amPlatformOwner` — `public/js/app-01.js:237` — 前端会话、消息、文件面板。
- `canOpenOnHost` — `public/js/app-01.js:247` — 前端会话、消息、文件面板。
- `dirOf` — `public/js/app-01.js:249` — 前端会话、消息、文件面板。
- `fpath` — `public/js/app-01.js:255` — 前端会话、消息、文件面板。
- `withRoot` — `public/js/app-01.js:267` — 前端会话、消息、文件面板。
- `rootOf` — `public/js/app-01.js:272` — 前端会话、消息、文件面板。
- `joinRel` — `public/js/app-01.js:277` — 前端会话、消息、文件面板。
- `mdImg` — `public/js/app-01.js:298` — 前端会话、消息、文件面板。
- `escInline` — `public/js/app-01.js:316` — 前端会话、消息、文件面板。
- `avatarBits` — `public/js/app-01.js:323` — 前端会话、消息、文件面板。
- `paintAvatar` — `public/js/app-01.js:335` — 前端会话、消息、文件面板。
- `displayName` — `public/js/app-01.js:343` — 前端会话、消息、文件面板。
- `applyAssistantIdentity` — `public/js/app-01.js:345` — 前端会话、消息、文件面板。
- `syncScrollGuides` — `public/js/app-01.js:363` — 前端会话、消息、文件面板。
- `scrollBottom` — `public/js/app-01.js:373` — 前端会话、消息、文件面板。
- `wireProcWarn` — `public/js/app-01.js:393` — 前端会话、消息、文件面板。
- `repairBareCode` — `public/js/app-01.js:408` — 前端会话、消息、文件面板。
- `mdFileLink` — `public/js/app-01.js:460` — 前端会话、消息、文件面板。
- `renderMd` — `public/js/app-01.js:478` — 前端会话、消息、文件面板。
- `balancedHtml` — `public/js/app-01.js:594` — 前端会话、消息、文件面板。
- `paintStream` — `public/js/app-01.js:603` — 前端会话、消息、文件面板。
- `sealStream` — `public/js/app-01.js:637` — 前端会话、消息、文件面板。
- `stripSceneTag` — `public/js/app-01.js:653` — 前端会话、消息、文件面板。
- `quoteTextOf` — `public/js/app-01.js:668` — 前端会话、消息、文件面板。
- `quoteReply` — `public/js/app-01.js:681` — 前端会话、消息、文件面板。
- `createTurnUI` — `public/js/app-01.js:697` — 前端会话、消息、文件面板。
- `liveActivity` — `public/js/app-01.js:1495` — 前端会话、消息、文件面板。
- `renderComposerTags` — `public/js/app-01.js:1582` — 前端会话、消息、文件面板。
- `setSceneTag` — `public/js/app-01.js:1593` — 前端会话、消息、文件面板。
- `setUseTag` — `public/js/app-01.js:1595` — 前端会话、消息、文件面板。
- `useDirective` — `public/js/app-01.js:1604` — 前端会话、消息、文件面板。
- `buildEmpty` — `public/js/app-01.js:1610` — 前端会话、消息、文件面板。
- `detectMention` — `public/js/app-01.js:1645` — 前端会话、消息、文件面板。
- `refreshFilesCache` — `public/js/app-01.js:1655` — 前端会话、消息、文件面板。
- `renderMentionMenu` — `public/js/app-01.js:1663` — 前端会话、消息、文件面板。
- `applyMention` — `public/js/app-01.js:1691` — 前端会话、消息、文件面板。
- `hlTokens` — `public/js/app-01.js:1707` — 前端会话、消息、文件面板。
- `syncInputHl` — `public/js/app-01.js:1716` — 前端会话、消息、文件面板。
- `openSweep` — `public/js/app-01.js:1767` — 前端会话、消息、文件面板。
- `renderSweepPanel` — `public/js/app-01.js:1782` — 前端会话、消息、文件面板。
- `makeSweepCard` — `public/js/app-01.js:1890` — 前端会话、消息、文件面板。
- `paintDiff` — `public/js/app-01.js:1956` — 前端会话、消息、文件面板。
- `rewindButton` — `public/js/app-01.js:1969` — 前端会话、消息、文件面板。
- `makeAskCard` — `public/js/app-01.js:2000` — 前端会话、消息、文件面板。
- `fileIcon` — `public/js/app-01.js:2138` — 前端会话、消息、文件面板。
- `fmtSize` — `public/js/app-01.js:2153` — 前端会话、消息、文件面板。
- `isResultFile` — `public/js/app-01.js:2170` — 前端会话、消息、文件面板。
- `revealFile` — `public/js/app-01.js:2178` — 前端会话、消息、文件面板。
- `copyHostFile` — `public/js/app-01.js:2190` — 前端会话、消息、文件面板。
- `openOnHost` — `public/js/app-01.js:2206` — 前端会话、消息、文件面板。
- `openWorkspaceOnHost` — `public/js/app-01.js:2213` — 前端会话、消息、文件面板。
- `downloadFile` — `public/js/app-01.js:2219` — 前端会话、消息、文件面板。
- `matchFiles` — `public/js/app-01.js:2231` — 前端会话、消息、文件面板。
- `renderFileFilter` — `public/js/app-01.js:2242` — 前端会话、消息、文件面板。
- `renderFiles` — `public/js/app-01.js:2270` — 前端会话、消息、文件面板。
- `refreshImStatus` — `public/js/app-01.js:2422` — 前端会话、消息、文件面板。
- `previewKind` — `public/js/app-01.js:2491` — 前端会话、消息、文件面板。
- `looksBinary` — `public/js/app-01.js:2508` — 前端会话、消息、文件面板。
- `fetchTextHead` — `public/js/app-01.js:2519` — 前端会话、消息、文件面板。
- `bindPvFallback` — `public/js/app-01.js:2543` — 前端会话、消息、文件面板。
- `fetchTextRange` — `public/js/app-01.js:2576` — 前端会话、消息、文件面板。
- `bindPvMore` — `public/js/app-01.js:2587` — 前端会话、消息、文件面板。
- `docHtml` — `public/js/app-01.js:2626` — 前端会话、消息、文件面板。
- `sheetHtml` — `public/js/app-01.js:2642` — 前端会话、消息、文件面板。
- `slidesHtml` — `public/js/app-01.js:2653` — 前端会话、消息、文件面板。
- `archiveHtml` — `public/js/app-01.js:2663` — 前端会话、消息、文件面板。
- `parseCsv` — `public/js/app-01.js:2670` — 前端会话、消息、文件面板。
- `csvHtml` — `public/js/app-01.js:2686` — 前端会话、消息、文件面板。
- `isCodeFile` — `public/js/app-01.js:2713` — 前端会话、消息、文件面板。
- `codeLang` — `public/js/app-01.js:2720` — 前端会话、消息、文件面板。
- `scanStr` — `public/js/app-01.js:2741` — 前端会话、消息、文件面板。
- `scanRe` — `public/js/app-01.js:2767` — 前端会话、消息、文件面板。
- `reAllowed` — `public/js/app-01.js:2786` — 前端会话、消息、文件面板。
- `longestLine` — `public/js/app-01.js:2797` — 前端会话、消息、文件面板。
- `codeMinified` — `public/js/app-01.js:2809` — 前端会话、消息、文件面板。
- `unminify` — `public/js/app-01.js:2822` — 前端会话、消息、文件面板。
- `hiCode` — `public/js/app-01.js:2930` — 前端会话、消息、文件面板。
- `codeHtml` — `public/js/app-01.js:2953` — 前端会话、消息、文件面板。
- `bindPvCode` — `public/js/app-01.js:2973` — 前端会话、消息、文件面板。
- `fitPreviewFrame` — `public/js/app-01.js:3005` — 前端会话、消息、文件面板。
- `previewFile` — `public/js/app-01.js:3090` — 前端会话、消息、文件面板。
- `copyPreviewImage` — `public/js/app-01.js:3197` — 前端会话、消息、文件面板。
- `copyImageFromUrl` — `public/js/app-01.js:3210` — 前端会话、消息、文件面板。
- `renderDeployBar` — `public/js/app-01.js:3266` — 前端会话、消息、文件面板。
- `startPreview` — `public/js/app-01.js:3316` — 前端会话、消息、文件面板。
- `snapshotFiles` — `public/js/app-01.js:3327` — 前端会话、消息、文件面板。
- `changedFiles` — `public/js/app-01.js:3340` — 前端会话、消息、文件面板。
- `renderSources` — `public/js/app-01.js:3354` — 前端会话、消息、文件面板。
- `hostOf` — `public/js/app-01.js:3381` — 前端会话、消息、文件面板。
- `reapScope` — `public/js/app-01.js:3403` — 前端会话、消息、文件面板。
- `reapDeletedOutputs` — `public/js/app-01.js:3444` — 前端会话、消息、文件面板。
- `renderTurnOutputs` — `public/js/app-01.js:3466` — 前端会话、消息、文件面板。
- `hideCardedRows` — `public/js/app-01.js:3553` — 前端会话、消息、文件面板。
- `clipOutList` — `public/js/app-01.js:3573` — 前端会话、消息、文件面板。
- `isDeliverable` — `public/js/app-01.js:3585` — 前端会话、消息、文件面板。
- `pathDepth` — `public/js/app-01.js:3590` — 前端会话、消息、文件面板。
- `extOf` — `public/js/app-01.js:3591` — 前端会话、消息、文件面板。
- `pairedCard` — `public/js/app-01.js:3594` — 前端会话、消息、文件面板。
- `attachAltFmt` — `public/js/app-01.js:3606` — 前端会话、消息、文件面板。
- `mergeFmtPairs` — `public/js/app-01.js:3631` — 前端会话、消息、文件面板。
- `makeOutCard` — `public/js/app-01.js:3647` — 前端会话、消息、文件面板。
- `markDupBasenames` — `public/js/app-01.js:3714` — 前端会话、消息、文件面板。
- `cssEsc` — `public/js/app-01.js:3731` — 前端会话、消息、文件面板。
- `fileLinkTargets` — `public/js/app-01.js:3737` — 前端会话、消息、文件面板。
- `linkifyOutputs` — `public/js/app-01.js:3752` — 前端会话、消息、文件面板。
- `makeFileLink` — `public/js/app-01.js:3814` — 前端会话、消息、文件面板。
- `filesInAsk` — `public/js/app-01.js:3847` — 前端会话、消息、文件面板。
- `askPreviewPlan` — `public/js/app-01.js:3869` — 前端会话、消息、文件面板。
- `finishPreviewPlan` — `public/js/app-01.js:3882` — 前端会话、消息、文件面板。
- `pickFinishDeliverable` — `public/js/app-01.js:3895` — 前端会话、消息、文件面板。
- `outputArrivalPlan` — `public/js/app-01.js:3912` — 前端会话、消息、文件面板。
- `applyOutputArrival` — `public/js/app-01.js:3919` — 前端会话、消息、文件面板。
- `bumpFilesBadge` — `public/js/app-01.js:3925` — 前端会话、消息、文件面板。
- `clearFilesBadge` — `public/js/app-01.js:3932` — 前端会话、消息、文件面板。
- `toggleSidebar` — `public/js/app-01.js:3947` — 前端会话、消息、文件面板。
- `setupPicker` — `public/js/app-01.js:3966` — 前端会话、消息、文件面板。
- `closeAllMenus` — `public/js/app-01.js:3972` — 前端会话、消息、文件面板。
- `refreshSettingsCache` — `public/js/app-01.js:3979` — 前端会话、消息、文件面板。
- `syncNavByRole` — `public/js/app-01.js:4002` — 前端会话、消息、文件面板。
- `currentSessModel` — `public/js/app-01.js:4018` — 前端会话、消息、文件面板。
- `defaultPendingModel` — `public/js/app-01.js:4024` — 前端会话、消息、文件面板。
- `activeEngine` — `public/js/app-01.js:4041` — 前端会话、消息、文件面板。
- `updateModelLabel` — `public/js/app-01.js:4048` — 前端会话、消息、文件面板。
- `setSessionModel` — `public/js/app-01.js:4069` — 前端会话、消息、文件面板。
- `healthBadge` — `public/js/app-01.js:4096` — 前端会话、消息、文件面板。
- `toolIcon` — `public/js/app-01.js:39` — 前端会话、消息、文件面板。
- `shortTool` — `public/js/app-01.js:40` — 前端会话、消息、文件面板。
- `curBusy` — `public/js/app-01.js:114` — 前端会话、消息、文件面板。
- `cliBusy` — `public/js/app-01.js:116` — 前端会话、消息、文件面板。
- `qOf` — `public/js/app-01.js:117` — 前端会话、消息、文件面板。
- `busySendMode` — `public/js/app-01.js:126` — 前端会话、消息、文件面板。
- `onlyResults` — `public/js/app-01.js:2156` — 前端会话、消息、文件面板。
- `revealBtn` — `public/js/app-01.js:2225` — 前端会话、消息、文件面板。
- `pvFallback` — `public/js/app-01.js:2535` — 前端会话、消息、文件面板。
- `pvTrunc` — `public/js/app-01.js:2562` — 前端会话、消息、文件面板。
- `runsHtml` — `public/js/app-01.js:2614` — 前端会话、消息、文件面板。
- `cellsHtml` — `public/js/app-01.js:2622` — 前端会话、消息、文件面板。
- `gridHtml` — `public/js/app-01.js:2623` — 前端会话、消息、文件面板。
- `ic` — `public/js/app-00-ui.js:8` — 前端 UI 基础：主题、字号、组件。
- `setMsg` — `public/js/app-00-ui.js:16` — 前端 UI 基础：主题、字号、组件。
- `isIconName` — `public/js/app-00-ui.js:28` — 前端 UI 基础：主题、字号、组件。
- `ava` — `public/js/app-00-ui.js:32` — 前端 UI 基础：主题、字号、组件。
- `avaCell` — `public/js/app-00-ui.js:71` — 前端 UI 基础：主题、字号、组件。
- `avaPicks` — `public/js/app-00-ui.js:80` — 前端 UI 基础：主题、字号、组件。
- `markActivatable` — `public/js/app-00-ui.js:153` — 前端 UI 基础：主题、字号、组件。
- `onActivate` — `public/js/app-00-ui.js:166` — 前端 UI 基础：主题、字号、组件。
- `rszRoom` — `public/js/app-00-ui.js:217` — 前端 UI 基础：主题、字号、组件。
- `setPanelW` — `public/js/app-00-ui.js:221` — 前端 UI 基础：主题、字号、组件。
- `resetPanelW` — `public/js/app-00-ui.js:229` — 前端 UI 基础：主题、字号、组件。
- `applyStoredW` — `public/js/app-00-ui.js:234` — 前端 UI 基础：主题、字号、组件。
- `initResizers` — `public/js/app-00-ui.js:245` — 前端 UI 基础：主题、字号、组件。
- `histMax` — `public/js/app-00-ui.js:293` — 前端 UI 基础：主题、字号、组件。
- `setHistH` — `public/js/app-00-ui.js:299` — 前端 UI 基础：主题、字号、组件。
- `resetHistH` — `public/js/app-00-ui.js:306` — 前端 UI 基础：主题、字号、组件。
- `applyStoredHistH` — `public/js/app-00-ui.js:312` — 前端 UI 基础：主题、字号、组件。
- `initHistResizer` — `public/js/app-00-ui.js:321` — 前端 UI 基础：主题、字号、组件。
- `fadeOnOverflow` — `public/js/app-00-ui.js:370` — 前端 UI 基础：主题、字号、组件。
- `initSideFades` — `public/js/app-00-ui.js:381` — 前端 UI 基础：主题、字号、组件。
- `rszW` — `public/js/app-00-ui.js:215` — 前端 UI 基础：主题、字号、组件。
- `cacheTxt` — `public/js/app-03.js:9` — 前端设置与安全。
- `renderAccount` — `public/js/app-03.js:14` — 前端设置与安全。
- `showAuth` — `public/js/app-03.js:157` — 前端设置与安全。
- `applyAuthMode` — `public/js/app-03.js:175` — 前端设置与安全。
- `forgotHintHtml` — `public/js/app-03.js:272` — 前端设置与安全。
- `submitAuth` — `public/js/app-03.js:290` — 前端设置与安全。
- `submitPair` — `public/js/app-03.js:344` — 前端设置与安全。
- `showAuthBlocked` — `public/js/app-03.js:392` — 前端设置与安全。
- `showTwoFactorGate` — `public/js/app-03.js:427` — 前端设置与安全。
- `initAuth` — `public/js/app-03.js:445` — 前端设置与安全。
- `mergeServerSessions` — `public/js/app-03.js:488` — 前端设置与安全。
- `keyLink` — `public/js/app-03.js:547` — 前端设置与安全。
- `modelKeySource` — `public/js/app-03.js:553` — 前端设置与安全。
- `onbSkipFlag` — `public/js/app-03.js:605` — 前端设置与安全。
- `maybeOnboard` — `public/js/app-03.js:614` — 前端设置与安全。
- `openOnboarding` — `public/js/app-03.js:627` — 前端设置与安全。
- `closeOnboarding` — `public/js/app-03.js:634` — 前端设置与安全。
- `dismissOnboarding` — `public/js/app-03.js:644` — 前端设置与安全。
- `onbReload` — `public/js/app-03.js:658` — 前端设置与安全。
- `onbGo` — `public/js/app-03.js:663` — 前端设置与安全。
- `onbHead` — `public/js/app-03.js:667` — 前端设置与安全。
- `onbFoot` — `public/js/app-03.js:671` — 前端设置与安全。
- `renderOnb` — `public/js/app-03.js:674` — 前端设置与安全。
- `renderOnbBrain` — `public/js/app-03.js:701` — 前端设置与安全。
- `renderOnbSearch` — `public/js/app-03.js:820` — 前端设置与安全。
- `renderOnbMedia` — `public/js/app-03.js:869` — 前端设置与安全。
- `renderOnbIm` — `public/js/app-03.js:933` — 前端设置与安全。
- `renderOnbDone` — `public/js/app-03.js:955` — 前端设置与安全。
- `finishOnb` — `public/js/app-03.js:979` — 前端设置与安全。
- `loadScriptOnce` — `public/js/app-03.js:1015` — 前端设置与安全。
- `loadCanvasDeps` — `public/js/app-03.js:1029` — 前端设置与安全。
- `renderCanvasLazy` — `public/js/app-03.js:1033` — 前端设置与安全。
- `openPageView` — `public/js/app-03.js:1060` — 前端设置与安全。
- `openAssistView` — `public/js/app-03.js:1088` — 前端设置与安全。
- `closeAssistView` — `public/js/app-03.js:1097` — 前端设置与安全。
- `renderAssistPage` — `public/js/app-03.js:1109` — 前端设置与安全。
- `sendAssistLocal` — `public/js/app-03.js:1164` — 前端设置与安全。
- `imBotAva` — `public/js/app-03.js:1172` — 前端设置与安全。
- `doAssistLocal` — `public/js/app-03.js:1176` — 前端设置与安全。
- `updateAssistLive` — `public/js/app-03.js:1203` — 前端设置与安全。
- `renderAssistFeed` — `public/js/app-03.js:1220` — 前端设置与安全。
- `renderProjPage` — `public/js/app-03.js:1254` — 前端设置与安全。
- `openProjEditor` — `public/js/app-03.js:1323` — 前端设置与安全。
- `buildCronFrom` — `public/js/app-03.js:1434` — 前端设置与安全。
- `cronToForm` — `public/js/app-03.js:1449` — 前端设置与安全。
- `renderAutomPage` — `public/js/app-03.js:1462` — 前端设置与安全。
- `renderAutomTplPicker` — `public/js/app-03.js:1580` — 前端设置与安全。
- `renderAutomForm` — `public/js/app-03.js:1594` — 前端设置与安全。
- `renderAutomRuns` — `public/js/app-03.js:1667` — 前端设置与安全。
- `renderFeedbackSummary` — `public/js/app-03.js:1749` — 前端设置与安全。
- `renderEvalPage` — `public/js/app-03.js:1775` — 前端设置与安全。
- `libPrefer` — `public/js/app-03.js:1730` — 前端设置与安全。
- `dramaFileUrl` — `public/js/app-07-drama.js:9` — 短剧业务节点与生成回写。
- `dramaBaseName` — `public/js/app-07-drama.js:12` — 短剧业务节点与生成回写。
- `dramaShotId` — `public/js/app-07-drama.js:13` — 短剧业务节点与生成回写。
- `dramaChars` — `public/js/app-07-drama.js:14` — 短剧业务节点与生成回写。
- `dramaAllShots` — `public/js/app-07-drama.js:17` — 短剧业务节点与生成回写。
- `dramaToast` — `public/js/app-07-drama.js:24` — 短剧业务节点与生成回写。
- `dramaList` — `public/js/app-07-drama.js:29` — 短剧业务节点与生成回写。
- `dramaLoad` — `public/js/app-07-drama.js:34` — 短剧业务节点与生成回写。
- `dramaSave` — `public/js/app-07-drama.js:42` — 短剧业务节点与生成回写。
- `dramaSaveShot` — `public/js/app-07-drama.js:63` — 短剧业务节点与生成回写。
- `dramaRerun` — `public/js/app-07-drama.js:80` — 短剧业务节点与生成回写。
- `dramaHistDiff` — `public/js/app-07-drama.js:149` — 短剧业务节点与生成回写。
- `dramaHistWhen` — `public/js/app-07-drama.js:155` — 短剧业务节点与生成回写。
- `dramaHistClose` — `public/js/app-07-drama.js:161` — 短剧业务节点与生成回写。
- `dramaHistKey` — `public/js/app-07-drama.js:166` — 短剧业务节点与生成回写。
- `dramaHistBlob` — `public/js/app-07-drama.js:167` — 短剧业务节点与生成回写。
- `dramaHistRow` — `public/js/app-07-drama.js:171` — 短剧业务节点与生成回写。
- `dramaHistRender` — `public/js/app-07-drama.js:191` — 短剧业务节点与生成回写。
- `dramaHistory` — `public/js/app-07-drama.js:229` — 短剧业务节点与生成回写。
- `dramaSnapBefore` — `public/js/app-07-drama.js:253` — 短剧业务节点与生成回写。
- `dramaCard` — `public/js/app-07-drama.js:264` — 短剧业务节点与生成回写。
- `dramaScene` — `public/js/app-07-drama.js:281` — 短剧业务节点与生成回写。
- `dramaJointType` — `public/js/app-07-drama.js:289` — 短剧业务节点与生成回写。
- `dramaJointZoom` — `public/js/app-07-drama.js:308` — 短剧业务节点与生成回写。
- `dramaJointPan` — `public/js/app-07-drama.js:315` — 短剧业务节点与生成回写。
- `renderDramaCanvas` — `public/js/app-07-drama.js:338` — 短剧业务节点与生成回写。
- `renderDramaPage` — `public/js/app-07-drama.js:392` — 短剧业务节点与生成回写。
- `dramaFieldCn` — `public/js/app-07-drama.js:148` — 短剧业务节点与生成回写。
- `canvasStorageKey` — `public/js/app-07-canvas.js:11` — 无限画布 JointJS 底座。
- `canvasToast` — `public/js/app-07-canvas.js:62` — 无限画布 JointJS 底座。
- `canvasType` — `public/js/app-07-canvas.js:67` — 无限画布 JointJS 底座。
- `canvasPayload` — `public/js/app-07-canvas.js:83` — 无限画布 JointJS 底座。
- `canvasKind` — `public/js/app-07-canvas.js:84` — 无限画布 JointJS 底座。
- `canvasEndpointId` — `public/js/app-07-canvas.js:85` — 无限画布 JointJS 底座。
- `canvasRelationLabel` — `public/js/app-07-canvas.js:90` — 无限画布 JointJS 底座。
- `canvasDefaultRelation` — `public/js/app-07-canvas.js:91` — 无限画布 JointJS 底座。
- `canvasLinkRelation` — `public/js/app-07-canvas.js:102` — 无限画布 JointJS 底座。
- `canvasRelationOptions` — `public/js/app-07-canvas.js:103` — 无限画布 JointJS 底座。
- `canvasNodeLabel` — `public/js/app-07-canvas.js:108` — 无限画布 JointJS 底座。
- `canvasSafeText` — `public/js/app-07-canvas.js:112` — 无限画布 JointJS 底座。
- `canvasResolvedFileName` — `public/js/app-07-canvas.js:113` — 无限画布 JointJS 底座。
- `canvasFileUrl` — `public/js/app-07-canvas.js:135` — 无限画布 JointJS 底座。
- `canvasMediaPath` — `public/js/app-07-canvas.js:143` — 无限画布 JointJS 底座。
- `canvasMediaMime` — `public/js/app-07-canvas.js:144` — 无限画布 JointJS 底座。
- `canvasMediaAvailable` — `public/js/app-07-canvas.js:168` — 无限画布 JointJS 底座。
- `canvasAudioPreview` — `public/js/app-07-canvas.js:175` — 无限画布 JointJS 底座。
- `canvasOutputFilename` — `public/js/app-07-canvas.js:180` — 无限画布 JointJS 底座。
- `canvasDefaultPayload` — `public/js/app-07-canvas.js:190` — 无限画布 JointJS 底座。
- `canvasZoom` — `public/js/app-07-canvas.js:206` — 无限画布 JointJS 底座。
- `canvasFitToBox` — `public/js/app-07-canvas.js:214` — 无限画布 JointJS 底座。
- `canvasFitAll` — `public/js/app-07-canvas.js:229` — 无限画布 JointJS 底座。
- `canvasCenterSelected` — `public/js/app-07-canvas.js:242` — 无限画布 JointJS 底座。
- `canvasAutoLayout` — `public/js/app-07-canvas.js:252` — 无限画布 JointJS 底座。
- `canvasBindViewport` — `public/js/app-07-canvas.js:274` — 无限画布 JointJS 底座。
- `canvasField` — `public/js/app-07-canvas.js:389` — 无限画布 JointJS 底座。
- `canvasAssetField` — `public/js/app-07-canvas.js:394` — 无限画布 JointJS 底座。
- `canvasTagField` — `public/js/app-07-canvas.js:421` — 无限画布 JointJS 底座。
- `canvasFileKindFromFile` — `public/js/app-07-canvas.js:427` — 无限画布 JointJS 底座。
- `canvasBytesToB64` — `public/js/app-07-canvas.js:434` — 无限画布 JointJS 底座。
- `canvasUploadWorkspaceFile` — `public/js/app-07-canvas.js:440` — 无限画布 JointJS 底座。
- `canvasFilePicker` — `public/js/app-07-canvas.js:451` — 无限画布 JointJS 底座。
- `canvasNodeHtml` — `public/js/app-07-canvas.js:455` — 无限画布 JointJS 底座。
- `canvasSelectedNode` — `public/js/app-07-canvas.js:525` — 无限画布 JointJS 底座。
- `canvasSetSelection` — `public/js/app-07-canvas.js:530` — 无限画布 JointJS 底座。
- `canvasPreviewRight` — `public/js/app-07-canvas.js:539` — 无限画布 JointJS 底座。
- `canvasDeleteSelection` — `public/js/app-07-canvas.js:544` — 无限画布 JointJS 底座。
- `canvasOpenContextMenu` — `public/js/app-07-canvas.js:552` — 无限画布 JointJS 底座。
- `canvasRefreshNode` — `public/js/app-07-canvas.js:577` — 无限画布 JointJS 底座。
- `canvasSnapshot` — `public/js/app-07-canvas.js:585` — 无限画布 JointJS 底座。
- `canvasHistoryKey` — `public/js/app-07-canvas.js:598` — 无限画布 JointJS 底座。
- `canvasHistoryCommit` — `public/js/app-07-canvas.js:599` — 无限画布 JointJS 底座。
- `canvasHistorySchedule` — `public/js/app-07-canvas.js:608` — 无限画布 JointJS 底座。
- `canvasHistoryReset` — `public/js/app-07-canvas.js:613` — 无限画布 JointJS 底座。
- `canvasHistoryFlush` — `public/js/app-07-canvas.js:617` — 无限画布 JointJS 底座。
- `canvasUndo` — `public/js/app-07-canvas.js:622` — 无限画布 JointJS 底座。
- `canvasRedo` — `public/js/app-07-canvas.js:629` — 无限画布 JointJS 底座。
- `canvasLoadCanvasList` — `public/js/app-07-canvas.js:637` — 无限画布 JointJS 底座。
- `canvasLoadWorkspaceProjects` — `public/js/app-07-canvas.js:644` — 无限画布 JointJS 底座。
- `canvasUniqueProjectName` — `public/js/app-07-canvas.js:652` — 无限画布 JointJS 底座。
- `canvasFinishWorkspaceSwitch` — `public/js/app-07-canvas.js:659` — 无限画布 JointJS 底座。
- `canvasSwitchWorkspace` — `public/js/app-07-canvas.js:670` — 无限画布 JointJS 底座。
- `canvasAskNewBoardName` — `public/js/app-07-canvas.js:698` — 无限画布 JointJS 底座。
- `canvasCreateBoard` — `public/js/app-07-canvas.js:720` — 无限画布 JointJS 底座。
- `canvasDeleteBoard` — `public/js/app-07-canvas.js:729` — 无限画布 JointJS 底座。
- `canvasPersist` — `public/js/app-07-canvas.js:738` — 无限画布 JointJS 底座。
- `canvasLoadSaved` — `public/js/app-07-canvas.js:767` — 无限画布 JointJS 底座。
- `canvasDecorateLink` — `public/js/app-07-canvas.js:775` — 无限画布 JointJS 底座。
- `canvasConnect` — `public/js/app-07-canvas.js:782` — 无限画布 JointJS 底座。
- `canvasLoadRemote` — `public/js/app-07-canvas.js:794` — 无限画布 JointJS 底座。
- `canvasReportLost` — `public/js/app-07-canvas.js:820` — 无限画布 JointJS 底座。
- `canvasInferLegacyEdges` — `public/js/app-07-canvas.js:833` — 无限画布 JointJS 底座。
- `canvasApplySnapshot` — `public/js/app-07-canvas.js:845` — 无限画布 JointJS 底座。
- `canvasStartRemoteSync` — `public/js/app-07-canvas.js:865` — 无限画布 JointJS 底座。
- `canvasRunInternal` — `public/js/app-07-canvas.js:875` — 无限画布 JointJS 底座。
- `canvasUpstreamInputs` — `public/js/app-07-canvas.js:893` — 无限画布 JointJS 底座。
- `canvasUpstreamNodes` — `public/js/app-07-canvas.js:904` — 无限画布 JointJS 底座。
- `canvasGenerationContext` — `public/js/app-07-canvas.js:906` — 无限画布 JointJS 底座。
- `canvasUpstreamImage` — `public/js/app-07-canvas.js:924` — 无限画布 JointJS 底座。
- `canvasUpstreamImages` — `public/js/app-07-canvas.js:940` — 无限画布 JointJS 底座。
- `canvasVoiceCandidates` — `public/js/app-07-canvas.js:955` — 无限画布 JointJS 底座。
- `canvasResolveVoice` — `public/js/app-07-canvas.js:964` — 无限画布 JointJS 底座。
- `canvasGenerationInputs` — `public/js/app-07-canvas.js:986` — 无限画布 JointJS 底座。
- `canvasGenerationSummary` — `public/js/app-07-canvas.js:990` — 无限画布 JointJS 底座。
- `canvasGenerationInspector` — `public/js/app-07-canvas.js:998` — 无限画布 JointJS 底座。
- `canvasRecordGeneration` — `public/js/app-07-canvas.js:1006` — 无限画布 JointJS 底座。
- `canvasUpsertResult` — `public/js/app-07-canvas.js:1016` — 无限画布 JointJS 底座。
- `canvasCreateDramaWorkflow` — `public/js/app-07-canvas.js:1029` — 无限画布 JointJS 底座。
- `canvasChatDisplayText` — `public/js/app-07-canvas.js:1051` — 无限画布 JointJS 底座。
- `canvasCompactToolPreview` — `public/js/app-07-canvas.js:1057` — 无限画布 JointJS 底座。
- `canvasChatReferenceIcon` — `public/js/app-07-canvas.js:1064` — 无限画布 JointJS 底座。
- `canvasChatReferenceDefaultUse` — `public/js/app-07-canvas.js:1065` — 无限画布 JointJS 底座。
- `canvasChatReferenceUseLabel` — `public/js/app-07-canvas.js:1066` — 无限画布 JointJS 底座。
- `canvasChatReferenceKind` — `public/js/app-07-canvas.js:1067` — 无限画布 JointJS 底座。
- `canvasChatReferenceMarkup` — `public/js/app-07-canvas.js:1068` — 无限画布 JointJS 底座。
- `canvasChatAppend` — `public/js/app-07-canvas.js:1076` — 无限画布 JointJS 底座。
- `canvasChatCandidates` — `public/js/app-07-canvas.js:1085` — 无限画布 JointJS 底座。
- `canvasRenderChatModeSelect` — `public/js/app-07-canvas.js:1102` — 无限画布 JointJS 底座。
- `canvasRenderChatModelSelect` — `public/js/app-07-canvas.js:1113` — 无限画布 JointJS 底座。
- `canvasMediaModelField` — `public/js/app-07-canvas.js:1120` — 无限画布 JointJS 底座。
- `canvasRenderChatReferences` — `public/js/app-07-canvas.js:1132` — 无限画布 JointJS 底座。
- `canvasRenderChatMentionMenu` — `public/js/app-07-canvas.js:1141` — 无限画布 JointJS 底座。
- `canvasChatAttachFile` — `public/js/app-07-canvas.js:1158` — 无限画布 JointJS 底座。
- `canvasOpenImagePreview` — `public/js/app-07-canvas.js:1169` — 无限画布 JointJS 底座。
- `canvasChatHasDraft` — `public/js/app-07-canvas.js:1187` — 无限画布 JointJS 底座。
- `canvasSyncChatSendButton` — `public/js/app-07-canvas.js:1191` — 无限画布 JointJS 底座。
- `canvasReferenceContext` — `public/js/app-07-canvas.js:1202` — 无限画布 JointJS 底座。
- `canvasChatStop` — `public/js/app-07-canvas.js:1206` — 无限画布 JointJS 底座。
- `canvasChatInterject` — `public/js/app-07-canvas.js:1215` — 无限画布 JointJS 底座。
- `canvasChatSend` — `public/js/app-07-canvas.js:1226` — 无限画布 JointJS 底座。
- `canvasRegisterTask` — `public/js/app-07-canvas.js:1230` — 无限画布 JointJS 底座。
- `canvasTaskSessionId` — `public/js/app-07-canvas.js:1239` — 无限画布 JointJS 底座。
- `canvasRenameTask` — `public/js/app-07-canvas.js:1250` — 无限画布 JointJS 底座。
- `canvasChatRun` — `public/js/app-07-canvas.js:1256` — 无限画布 JointJS 底座。
- `canvasBoardCastOf` — `public/js/app-07-canvas.js:1312` — 无限画布 JointJS 底座。
- `canvasBoardCastSync` — `public/js/app-07-canvas.js:1318` — 无限画布 JointJS 底座。
- `canvasBoardFramePrompt` — `public/js/app-07-canvas.js:1360` — 无限画布 JointJS 底座。
- `canvasBoardContentSync` — `public/js/app-07-canvas.js:1368` — 无限画布 JointJS 底座。
- `canvasBoardWriteback` — `public/js/app-07-canvas.js:1418` — 无限画布 JointJS 底座。
- `canvasGenerate` — `public/js/app-07-canvas.js:1436` — 无限画布 JointJS 底座。
- `canvasBindNode` — `public/js/app-07-canvas.js:1540` — 无限画布 JointJS 底座。
- `canvasBoardPlan` — `public/js/app-07-canvas.js:1603` — 无限画布 JointJS 底座。
- `canvasApplyBoardPlan` — `public/js/app-07-canvas.js:1642` — 无限画布 JointJS 底座。
- `canvasAddNode` — `public/js/app-07-canvas.js:1670` — 无限画布 JointJS 底座。
- `canvasUpdateSelected` — `public/js/app-07-canvas.js:1679` — 无限画布 JointJS 底座。
- `canvasRenderInspector` — `public/js/app-07-canvas.js:1686` — 无限画布 JointJS 底座。
- `canvasLoadBoards` — `public/js/app-07-canvas.js:1762` — 无限画布 JointJS 底座。
- `canvasFileKind` — `public/js/app-07-canvas.js:1764` — 无限画布 JointJS 底座。
- `canvasAssetRows` — `public/js/app-07-canvas.js:1782` — 无限画布 JointJS 底座。
- `canvasAssetUseText` — `public/js/app-07-canvas.js:1790` — 无限画布 JointJS 底座。
- `canvasRenderLibrary` — `public/js/app-07-canvas.js:1798` — 无限画布 JointJS 底座。
- `canvasBytesText` — `public/js/app-07-canvas.js:1836` — 无限画布 JointJS 底座。
- `canvasEtaText` — `public/js/app-07-canvas.js:1857` — 无限画布 JointJS 底座。
- `canvasProgressFocus` — `public/js/app-07-canvas.js:1866` — 无限画布 JointJS 底座。
- `canvasComposeHtml` — `public/js/app-07-canvas.js:1879` — 无限画布 JointJS 底座。
- `canvasProgressHeight` — `public/js/app-07-canvas.js:1957` — 无限画布 JointJS 底座。
- `canvasRenderProgress` — `public/js/app-07-canvas.js:1965` — 无限画布 JointJS 底座。
- `canvasLoadProgress` — `public/js/app-07-canvas.js:2041` — 无限画布 JointJS 底座。
- `canvasRunPending` — `public/js/app-07-canvas.js:2054` — 无限画布 JointJS 底座。
- `canvasComposeOpen` — `public/js/app-07-canvas.js:2127` — 无限画布 JointJS 底座。
- `canvasComposeStart` — `public/js/app-07-canvas.js:2145` — 无限画布 JointJS 底座。
- `canvasComposePoll` — `public/js/app-07-canvas.js:2160` — 无限画布 JointJS 底座。
- `canvasComposeStop` — `public/js/app-07-canvas.js:2179` — 无限画布 JointJS 底座。
- `canvasIsPlaceholderPrompt` — `public/js/app-07-canvas.js:2190` — 无限画布 JointJS 底座。
- `canvasLoadAssets` — `public/js/app-07-canvas.js:2192` — 无限画布 JointJS 底座。
- `canvasVerifyMissing` — `public/js/app-07-canvas.js:2208` — 无限画布 JointJS 底座。
- `canvasLoadLibrary` — `public/js/app-07-canvas.js:2240` — 无限画布 JointJS 底座。
- `canvasEmbeddedImage` — `public/js/app-07-canvas.js:2250` — 无限画布 JointJS 底座。
- `canvasMaterializeResultNodes` — `public/js/app-07-canvas.js:2255` — 无限画布 JointJS 底座。
- `canvasRenderBroken` — `public/js/app-07-canvas.js:2277` — 无限画布 JointJS 底座。
- `canvasRestoreFromLocal` — `public/js/app-07-canvas.js:2299` — 无限画布 JointJS 底座。
- `canvasRestoreOrSeed` — `public/js/app-07-canvas.js:2311` — 无限画布 JointJS 底座。
- `canvasDestroy` — `public/js/app-07-canvas.js:2321` — 无限画布 JointJS 底座。
- `renderCanvasPage` — `public/js/app-07-canvas.js:2323` — 无限画布 JointJS 底座。
- `fmtTs` — `public/js/admin.js:32` — 管理后台前端。
- `fmtDate` — `public/js/admin.js:38` — 管理后台前端。
- `ago` — `public/js/admin.js:45` — 管理后台前端。
- `api` — `public/js/admin.js:70` — 管理后台前端。
- `toast` — `public/js/admin.js:87` — 管理后台前端。
- `act` — `public/js/admin.js:96` — 管理后台前端。
- `modal` — `public/js/admin.js:112` — 管理后台前端。
- `confirmBox` — `public/js/admin.js:179` — 管理后台前端。
- `statCols` — `public/js/admin.js:209` — 管理后台前端。
- `table` — `public/js/admin.js:230` — 管理后台前端。
- `presetRange` — `public/js/admin.js:246` — 管理后台前端。
- `activePreset` — `public/js/admin.js:261` — 管理后台前端。
- `filterBar` — `public/js/admin.js:273` — 管理后台前端。
- `bindFilter` — `public/js/admin.js:291` — 管理后台前端。
- `pager` — `public/js/admin.js:310` — 管理后台前端。
- `bindPager` — `public/js/admin.js:323` — 管理后台前端。
- `downloadCsv` — `public/js/admin.js:329` — 管理后台前端。
- `bars7` — `public/js/admin.js:356` — 管理后台前端。
- `showReceipt` — `public/js/admin.js:1183` — 管理后台前端。
- `showPassword` — `public/js/admin.js:1193` — 管理后台前端。
- `inviteLink` — `public/js/admin.js:1213` — 管理后台前端。
- `copyText` — `public/js/admin.js:1216` — 管理后台前端。
- `fallbackCopy` — `public/js/admin.js:1220` — 管理后台前端。
- `saveBar` — `public/js/admin.js:2041` — 管理后台前端。
- `bindSettings` — `public/js/admin.js:2046` — 管理后台前端。
- `chanIdle` — `public/js/admin.js:2295` — 管理后台前端。
- `navHit` — `public/js/admin.js:2590` — 管理后台前端。
- `renderNav` — `public/js/admin.js:2593` — 管理后台前端。
- `bindNavSearch` — `public/js/admin.js:2618` — 管理后台前端。
- `navItem` — `public/js/admin.js:2648` — 管理后台前端。
- `route` — `public/js/admin.js:2654` — 管理后台前端。
- `gate` — `public/js/admin.js:2684` — 管理后台前端。
- `boot` — `public/js/admin.js:2699` — 管理后台前端。
- `$` — `public/js/admin.js:19` — 管理后台前端。
- `esc` — `public/js/admin.js:20` — 管理后台前端。
- `ic` — `public/js/admin.js:22` — 管理后台前端。
- `num` — `public/js/admin.js:23` — 管理后台前端。
- `big` — `public/js/admin.js:25` — 管理后台前端。
- `pad2` — `public/js/admin.js:31` — 管理后台前端。
- `mb` — `public/js/admin.js:58` — 管理后台前端。
- `jsonOpts` — `public/js/admin.js:82` — 管理后台前端。
- `post` — `public/js/admin.js:83` — 管理后台前端。
- `del` — `public/js/admin.js:84` — 管理后台前端。
- `card` — `public/js/admin.js:184` — 管理后台前端。
- `cardT` — `public/js/admin.js:190` — 管理后台前端。
- `secT` — `public/js/admin.js:191` — 管理后台前端。
- `headRow` — `public/js/admin.js:193` — 管理后台前端。
- `note` — `public/js/admin.js:199` — 管理后台前端。
- `kpi` — `public/js/admin.js:214` — 管理后台前端。
- `badge` — `public/js/admin.js:218` — 管理后台前端。
- `empty` — `public/js/admin.js:219` — 管理后台前端。
- `progress` — `public/js/admin.js:221` — 管理后台前端。
- `dl` — `public/js/admin.js:229` — 管理后台前端。
- `iso` — `public/js/admin.js:244` — 管理后台前端。
- `qs` — `public/js/admin.js:341` — 管理后台前端。
- `field` — `public/js/admin.js:369` — 管理后台前端。
- `sw` — `public/js/admin.js:373` — 管理后台前端。
- `inp` — `public/js/admin.js:375` — 管理后台前端。
- `inpO` — `public/js/admin.js:632` — 管理后台前端。
- `yuan` — `public/js/admin.js:1552` — 管理后台前端。
- `cap` — `public/js/admin.js:1560` — 管理后台前端。
- `renderHubMcp` — `public/js/app-05.js:1` — 前端模型表、媒体、Trace。
- `renderPromptPage` — `public/js/app-05.js:283` — 前端模型表、媒体、Trace。
- `startTaskWith` — `public/js/app-05.js:332` — 前端模型表、媒体、Trace。
- `startTaskUsing` — `public/js/app-05.js:348` — 前端模型表、媒体、Trace。
- `cronToHuman` — `public/js/app-05.js:360` — 前端模型表、媒体、Trace。
- `localMin` — `public/js/app-05.js:380` — 前端模型表、媒体、Trace。
- `whenToHuman` — `public/js/app-05.js:385` — 前端模型表、媒体、Trace。
- `renderSettings` — `public/js/app-05.js:419` — 前端模型表、媒体、Trace。
- `saveSettings` — `public/js/app-05.js:457` — 前端模型表、媒体、Trace。
- `loadMediaCatalog` — `public/js/app-05.js:506` — 前端模型表、媒体、Trace。
- `saveAllModelTables` — `public/js/app-05.js:517` — 前端模型表、媒体、Trace。
- `saveMediaTables` — `public/js/app-05.js:524` — 前端模型表、媒体、Trace。
- `repaintMedia` — `public/js/app-05.js:528` — 前端模型表、媒体、Trace。
- `mediaFormOpen` — `public/js/app-05.js:541` — 前端模型表、媒体、Trace。
- `renderMediaPane` — `public/js/app-05.js:545` — 前端模型表、媒体、Trace。
- `paintMediaPaused` — `public/js/app-05.js:572` — 前端模型表、媒体、Trace。
- `provDupeTags` — `public/js/app-05.js:600` — 前端模型表、媒体、Trace。
- `provLabel` — `public/js/app-05.js:613` — 前端模型表、媒体、Trace。
- `provOptions` — `public/js/app-05.js:619` — 前端模型表、媒体、Trace。
- `paintMedia` — `public/js/app-05.js:626` — 前端模型表、媒体、Trace。
- `capCard` — `public/js/app-05.js:650` — 前端模型表、媒体、Trace。
- `bindMedia` — `public/js/app-05.js:704` — 前端模型表、媒体、Trace。
- `mmRelay` — `public/js/app-05.js:774` — 前端模型表、媒体、Trace。
- `mmBrand` — `public/js/app-05.js:777` — 前端模型表、媒体、Trace。
- `mmMismatch` — `public/js/app-05.js:788` — 前端模型表、媒体、Trace。
- `kindLabel` — `public/js/app-05.js:794` — 前端模型表、媒体、Trace。
- `fillProvSelect` — `public/js/app-05.js:799` — 前端模型表、媒体、Trace。
- `fillModelSelect` — `public/js/app-05.js:813` — 前端模型表、媒体、Trace。
- `injectLive` — `public/js/app-05.js:843` — 前端模型表、媒体、Trace。
- `renderModelsPane` — `public/js/app-05.js:879` — 前端模型表、媒体、Trace。
- `paintModels` — `public/js/app-05.js:896` — 前端模型表、媒体、Trace。
- `chatOverview` — `public/js/app-05.js:1018` — 前端模型表、媒体、Trace。
- `capsRow` — `public/js/app-05.js:1075` — 前端模型表、媒体、Trace。
- `chanIdle` — `public/js/app-05.js:1089` — 前端模型表、媒体、Trace。
- `modelKeyed` — `public/js/app-05.js:1103` — 前端模型表、媒体、Trace。
- `chanTestPill` — `public/js/app-05.js:1119` — 前端模型表、媒体、Trace。
- `chanTestNote` — `public/js/app-05.js:1129` — 前端模型表、媒体、Trace。
- `chanCard` — `public/js/app-05.js:1138` — 前端模型表、媒体、Trace。
- `modelRow` — `public/js/app-05.js:1196` — 前端模型表、媒体、Trace。
- `rowMenu` — `public/js/app-05.js:1217` — 前端模型表、媒体、Trace。
- `kindKeyLink` — `public/js/app-05.js:1229` — 前端模型表、媒体、Trace。
- `bindRowMenus` — `public/js/app-05.js:1237` — 前端模型表、媒体、Trace。
- `bindModels` — `public/js/app-05.js:1256` — 前端模型表、媒体、Trace。
- `fillChanSelect` — `public/js/app-05.js:1515` — 前端模型表、媒体、Trace。
- `fillChatModelSelect` — `public/js/app-05.js:1532` — 前端模型表、媒体、Trace。
- `injectLiveChat` — `public/js/app-05.js:1562` — 前端模型表、媒体、Trace。
- `renderSearchPane` — `public/js/app-05.js:1578` — 前端模型表、媒体、Trace。
- `traceFmtDuration` — `public/js/app-05.js:1631` — 前端模型表、媒体、Trace。
- `traceModels` — `public/js/app-05.js:1637` — 前端模型表、媒体、Trace。
- `traceUsage` — `public/js/app-05.js:1640` — 前端模型表、媒体、Trace。
- `traceObsDuration` — `public/js/app-05.js:1648` — 前端模型表、媒体、Trace。
- `traceObsStats` — `public/js/app-05.js:1653` — 前端模型表、媒体、Trace。
- `traceTimeRange` — `public/js/app-05.js:1662` — 前端模型表、媒体、Trace。
- `traceTarget` — `public/js/app-05.js:1676` — 前端模型表、媒体、Trace。
- `traceTargetText` — `public/js/app-05.js:1690` — 前端模型表、媒体、Trace。
- `traceJson` — `public/js/app-05.js:1699` — 前端模型表、媒体、Trace。
- `traceFold` — `public/js/app-05.js:1702` — 前端模型表、媒体、Trace。
- `traceStepHtml` — `public/js/app-05.js:1707` — 前端模型表、媒体、Trace。
- `renderTraceDetail` — `public/js/app-05.js:1729` — 前端模型表、媒体、Trace。
- `traceHit` — `public/js/app-05.js:1772` — 前端模型表、媒体、Trace。
- `renderTracePage` — `public/js/app-05.js:1777` — 前端模型表、媒体、Trace。
- `loadTracePage` — `public/js/app-05.js:1806` — 前端模型表、媒体、Trace。
- `traceWho` — `public/js/app-05.js:1845` — 前端模型表、媒体、Trace。
- `traceWhere` — `public/js/app-05.js:1849` — 前端模型表、媒体、Trace。
- `traceRowHtml` — `public/js/app-05.js:1871` — 前端模型表、媒体、Trace。
- `paintTraceList` — `public/js/app-05.js:1907` — 前端模型表、媒体、Trace。
- `renderOpsPane` — `public/js/app-05.js:1931` — 前端模型表、媒体、Trace。
- `renderTracePane` — `public/js/app-05.js:2029` — 前端模型表、媒体、Trace。
- `renderAgentPane` — `public/js/app-05.js:2107` — 前端模型表、媒体、Trace。
- `renderThinkingCard` — `public/js/app-05.js:2188` — 前端模型表、媒体、Trace。
- `renderEngineCard` — `public/js/app-05.js:2218` — 前端模型表、媒体、Trace。
- `engineExtraHtml` — `public/js/app-05.js:2279` — 前端模型表、媒体、Trace。
- `bindEngineExtra` — `public/js/app-05.js:2305` — 前端模型表、媒体、Trace。
- `testEngineConnect` — `public/js/app-05.js:2327` — 前端模型表、媒体、Trace。
- `renderPersonaPane` — `public/js/app-05.js:2354` — 前端模型表、媒体、Trace。
- `petCardHtml` — `public/js/app-05.js:2392` — 前端模型表、媒体、Trace。
- `petSpriteHint` — `public/js/app-05.js:2426` — 前端模型表、媒体、Trace。
- `petSpriteOptions` — `public/js/app-05.js:2435` — 前端模型表、媒体、Trace。
- `bindPetCard` — `public/js/app-05.js:2443` — 前端模型表、媒体、Trace。
- `squareThumb` — `public/js/app-05.js:2489` — 前端模型表、媒体、Trace。
- `renderMemoryPane` — `public/js/app-05.js:2511` — 前端模型表、媒体、Trace。
- `renderDataPane` — `public/js/app-05.js:2635` — 前端模型表、媒体、Trace。
- `larkConsoleLink` — `public/js/app-05.js:2765` — 前端模型表、媒体、Trace。
- `larkChip` — `public/js/app-05.js:2772` — 前端模型表、媒体、Trace。
- `renderLarkQr` — `public/js/app-05.js:2780` — 前端模型表、媒体、Trace。
- `setPacked` — `public/js/app-05.js:2861` — 前端模型表、媒体、Trace。
- `parseFeishuCreds` — `public/js/app-05.js:2875` — 前端模型表、媒体、Trace。
- `wxStatus` — `public/js/app-05.js:2884` — 前端模型表、媒体、Trace。
- `wsChip` — `public/js/app-05.js:2888` — 前端模型表、媒体、Trace。
- `renderImPane` — `public/js/app-05.js:2959` — 前端模型表、媒体、Trace。

## 5. 测试函数索引

- `testAgentPipeline` — `test/e2e.js:216` — 用模拟 LLM 跑通规划-工具-交付主路径，确认过程事件与成果文件都在。
- `testOfficeLibs` — `test/e2e.js:243` — 真用 docx/exceljs/pptxgenjs 生成再读回，标题级别表格不能丢。
- `testPreviewExtract` — `test/e2e.js:267` — 预览拆包不喂手搓样本，坏文件必须报错而不是半截垃圾。
- `testNodeSyntaxPrecheck` — `test/e2e.js:348` — 写后 JS 语法预检能拦住明显坏脚本。
- `testShellGlobCompat` — `test/e2e.js:373` — zsh nonomatch：通配符没匹配时命令仍要跑，URL 里的 ? [] 不能被当 glob。
- `testMissingBinHintWired` — `test/e2e.js:416` — 缺 ffmpeg 等命令时回执翻译成人话安装提示。
- `testSessionFileLayout` — `test/e2e.js:445` — 工具回执必须报子目录相对路径，判重只认逐字节哈希。
- `testCssTokenGate` — `test/e2e.js:501` — JS 引用过的 CSS 变量必须在样式表里定义。
- `testMotionGate` — `test/e2e.js:660` — 回归该主题行为，含边界与负对照。
- `testSurfaceLayerGate` — `test/e2e.js:739` — 回归该主题行为，含边界与负对照。
- `testVerdictGate` — `test/e2e.js:838` — 回归该主题行为，含边界与负对照。
- `testDocLinkGate` — `test/e2e.js:884` — 回归该主题行为，含边界与负对照。
- `testImageWatermarkGate` — `test/e2e.js:993` — 回归该主题行为，含边界与负对照。
- `testVideoWatermarkGate` — `test/e2e.js:1064` — 回归该主题行为，含边界与负对照。
- `testMediaImageInputGate` — `test/e2e.js:1138` — 回归该主题行为，含边界与负对照。
- `testVideoProtocols` — `test/e2e.js:1333` — 回归该主题行为，含边界与负对照。
- `testMediaKeyHygiene` — `test/e2e.js:1539` — 回归该主题行为，含边界与负对照。
- `testCliMode` — `test/e2e.js:1629` — 回归该主题行为，含边界与负对照。
- `testDeliverableGate` — `test/e2e.js:1846` — 声称的文件不在磁盘要打回。
- `testContextBudget` — `test/e2e.js:1884` — 超预算裁剪后关键约束仍在或有摘要。
- `testCtxMeterWiring` — `test/e2e.js:1936` — 回归该主题行为，含边界与负对照。
- `testToolPairRepair` — `test/e2e.js:1964` — 半截 tool_use/tool_result 对子在发请求前补齐或丢掉。
- `testFetchRetry` — `test/e2e.js:2045` — 5xx/429 退避重试，4xx 立即失败。
- `testCheckPageConsole` — `test/e2e.js:2172` — Electron 自注入 CSP 警告不算页面错误。
- `testLookAtImage` — `test/e2e.js:2203` — 看图只回文字，图不进对话历史。
- `testAnthropicEndpointAgreement` — `test/e2e.js:2324` — 回归该主题行为，含边界与负对照。
- `testPluginManifest` — `test/e2e.js:2400` — Agent Plugins 清单字段齐全。
- `testPluginComponentIsolation` — `test/e2e.js:2450` — 回归该主题行为，含边界与负对照。
- `testPluginMcpRuntime` — `test/e2e.js:2511` — 回归该主题行为，含边界与负对照。
- `testPluginSkillsIntegration` — `test/e2e.js:2570` — 回归该主题行为，含边界与负对照。
- `testMcpStreamableHttp` — `test/e2e.js:2661` — MCP Streamable HTTP 握手与工具列表。
- `testMcpManagerLifecycle` — `test/e2e.js:2682` — 回归该主题行为，含边界与负对照。
- `testFrontendSvgFigures` — `test/e2e.js:2765` — 回归该主题行为，含边界与负对照。
- `testDockerDeploy` — `test/e2e.js:2791` — 回归该主题行为，含边界与负对照。
- `testNodeSuite` — `test/e2e.js:2809` — 回归该主题行为，含边界与负对照。
- `testAdminConsoleUI` — `test/e2e.js:2819` — 回归该主题行为，含边界与负对照。
- `testDesktopAppIdentity` — `test/e2e.js:2838` — 回归该主题行为，含边界与负对照。
- `testDefaultSkillsManifest` — `test/e2e.js:2941` — 回归该主题行为，含边界与负对照。
- `testCron` — `test/e2e.js:2993` — cron 五字段解析与匹配。
- `testSchedulerRuntime` — `test/e2e.js:3029` — 回归该主题行为，含边界与负对照。
- `testPermissionModes` — `test/e2e.js:3091` — 四档权限写/命令行为符合矩阵。
- `testEventLedgerParity` — `test/e2e.js:3218` — 回归该主题行为，含边界与负对照。
- `testEvolveLoop` — `test/e2e.js:3233` — 信号窗口、👎改判、闸门六种毙法、提案不点头不进提示词、打分只认数字。
- `testEvolveRecency` — `test/e2e.js:3388` — 按轮时间戳开窗，没有时间不许编日期。
- `testEvolvePromptBudget` — `test/e2e.js:3491` — 回归该主题行为，含边界与负对照。
- `testAgentPromptDrift` — `test/e2e.js:3622` — 回归该主题行为，含边界与负对照。
- `testTaskDirLifecycle` — `test/e2e.js:3717` — 回归该主题行为，含边界与负对照。
- `testCodingTools` — `test/e2e.js:3791` — 回归该主题行为，含边界与负对照。
- `testDeliverableQuality` — `test/e2e.js:3951` — 回归该主题行为，含边界与负对照。
- `testDiagramRepair` — `test/e2e.js:4147` — 回归该主题行为，含边界与负对照。
- `testEvolveCaliberAndSpread` — `test/e2e.js:4247` — 回归该主题行为，含边界与负对照。
- `testMemoryLayer` — `test/e2e.js:4364` — 双层记忆隔离、去重、超量淘汰、密钥拒记。
- `testMemoryNearDup` — `test/e2e.js:4547` — 回归该主题行为，含边界与负对照。
- `testCommandGate` — `test/e2e.js:4598` — 黑名单与高危模式全档位生效。
- `testAccountStore` — `test/e2e.js:4665` — 账本 strict、改名登录、积分。
- `testCreditsGate` — `test/e2e.js:4735` — 积分不足拒跑，缓存命中按 1/10。
- `testCachedLedger` — `test/e2e.js:4807` — 回归该主题行为，含边界与负对照。
- `testRenameLogin` — `test/e2e.js:4866` — 回归该主题行为，含边界与负对照。
- `testAvatarRules` — `test/e2e.js:4917` — 回归该主题行为，含边界与负对照。
- `testJsonStore` — `test/e2e.js:4937` — 原子写、坏文件隔离、空文件自愈。
- `testImSessionStore` — `test/e2e.js:4975` — 回归该主题行为，含边界与负对照。
- `testForcedWrapUp` — `test/e2e.js:5073` — 撞上限额外收尾请求不带工具。
- `testCollectSources` — `test/e2e.js:5151` — 回归该主题行为，含边界与负对照。
- `testLlmStreamFailures` — `test/e2e.js:5184` — 回归该主题行为，含边界与负对照。
- `testLeakedToolCallRescue` — `test/e2e.js:5224` — 正文里的工具标记还原为调用。
- `testFetchUrlShapes` — `test/e2e.js:5260` — 回归该主题行为，含边界与负对照。
- `testParallelToolBatch` — `test/e2e.js:5345` — 只读并发、有写串行、call id 配对。
- `testPathSafety` — `test/e2e.js:5415` — 工作区外与 .. 逃逸拒绝。
- `testDesktopPet` — `test/e2e.js:5428` — 回归该主题行为，含边界与负对照。
- `testConnectorToggleAndTools` — `test/e2e.js:5564` — 回归该主题行为，含边界与负对照。
- `testScheduleRunTrace` — `test/e2e.js:5777` — 回归该主题行为，含边界与负对照。
- `testMcpFailureReason` — `test/e2e.js:5974` — 回归该主题行为，含边界与负对照。
- `testPetSprites` — `test/e2e.js:6005` — 回归该主题行为，含边界与负对照。
- `testPromptQuestionVsWork` — `test/e2e.js:6178` — 回归该主题行为，含边界与负对照。
- `testPromptNoAskContradiction` — `test/e2e.js:6229` — 回归该主题行为，含边界与负对照。
- `testFilePathRouting` — `test/e2e.js:6360` — 回归该主题行为，含边界与负对照。
- `testLibraryOutputsTruth` — `test/e2e.js:6518` — 回归该主题行为，含边界与负对照。
- `testLibraryTurnAnchor` — `test/e2e.js:6731` — 回归该主题行为，含边界与负对照。
- `testSweepApi` — `test/e2e.js:6863` — 回归该主题行为，含边界与负对照。
- `testSessionSearchLive` — `test/e2e.js:6987` — 回归该主题行为，含边界与负对照。
- `testDecideLive` — `test/e2e.js:7101` — 回归该主题行为，含边界与负对照。
- `testConfigExternalEdit` — `test/e2e.js:7195` — 回归该主题行为，含边界与负对照。
- `testKeyGuard` — `test/e2e.js:7275` — 回归该主题行为，含边界与负对照。
- `testPortCollision` — `test/e2e.js:7461` — 回归该主题行为，含边界与负对照。
- `testLocalEngineConnect` — `test/e2e.js:7558` — 回归该主题行为，含边界与负对照。
- `testAskUser` — `test/e2e.js:7642` — 提问卡选项含代价说明，超时走默认。
- `testOutputFilesRecency` — `test/e2e.js:7718` — 回归该主题行为，含边界与负对照。
- `testCheckpoints` — `test/e2e.js:7829` — 新建-修改-回退-撤销一整圈，越权路径 refused。
- `testBackupRoundTrip` — `test/e2e.js:7889` — 打包恢复前有 before-restore 安全备份。
- `testThumbPng` — `test/e2e.js:8064` — 回归该主题行为，含边界与负对照。
- `testThumbPool` — `test/e2e.js:8263` — 回归该主题行为，含边界与负对照。
- `testCanvasCreativeLineage` — `test/e2e.js:8692` — 回归该主题行为，含边界与负对照。
- `testCanvasMissingAssets` — `test/e2e.js:8737` — 回归该主题行为，含边界与负对照。
- `testCanvasThumb` — `test/e2e.js:8873` — 回归该主题行为，含边界与负对照。
- `testGoalOnLocalEngine` — `test/e2e.js:8948` — Goal 壳在外部引擎上仍跑验收。
- `testDetectCache` — `test/e2e.js:9000` — 回归该主题行为，含边界与负对照。
- `testStreamRender` — `test/e2e.js:9033` — 回归该主题行为，含边界与负对照。
- `testEmbedFailoverResilience` — `test/e2e.js:9062` — 回归该主题行为，含边界与负对照。
- `testThinkingSwitch` — `test/e2e.js:9207` — 思考档映射各家参数，auto 不发字段。
- `testOnboardingWizardApi` — `test/e2e.js:9349` — 回归该主题行为，含边界与负对照。
- `testThinkingSettingsApi` — `test/e2e.js:9516` — 回归该主题行为，含边界与负对照。
- `testFilesEmitter` — `test/e2e.js:9596` — 回归该主题行为，含边界与负对照。
- `testHeavyTools` — `test/e2e.js:9815` — 回归该主题行为，含边界与负对照。
- `testOutputOwnership` — `test/e2e.js:10003` — 回归该主题行为，含边界与负对照。
- `testEngineToolBridge` — `test/e2e.js:10121` — 回归该主题行为，含边界与负对照。
- `testEngineSecurityGuard` — `test/e2e.js:10289` — 回归该主题行为，含边界与负对照。
- `testReadmeFrontGate` — `test/e2e.js:10420` — 回归该主题行为，含边界与负对照。
- `testNoticeCoverage` — `test/e2e.js:10681` — 回归该主题行为，含边界与负对照。
- `testIntranet` — `test/e2e.js:10722` — 回归该主题行为，含边界与负对照。
- `testStyleDirection` — `test/e2e.js:10993` — 回归该主题行为，含边界与负对照。
- `testShortDrama` — `test/e2e.js:11117` — 回归该主题行为，含边界与负对照。
- `testReleasePipeline` — `test/e2e.js:11305` — 回归该主题行为，含边界与负对照。
- `testPackageAssetDrift` — `test/e2e.js:11367` — 回归该主题行为，含边界与负对照。
- `testNoNestedRoutes` — `test/e2e.js:11431` — 回归该主题行为，含边界与负对照。
- `testPackagingAndDemoGate` — `test/e2e.js:11457` — 回归该主题行为，含边界与负对照。
- `testAdminModelsPage` — `test/e2e.js:11496` — 回归该主题行为，含边界与负对照。
- `testKeySourcesGate` — `test/e2e.js:11599` — 回归该主题行为，含边界与负对照。
- `testI18n` — `test/e2e.js:11625` — 回归该主题行为，含边界与负对照。
- `testConnectorsAndExperts` — `test/e2e.js:11769` — 回归该主题行为，含边界与负对照。
- `testOutputArrivalStatic` — `test/e2e.js:11937` — 回归该主题行为，含边界与负对照。
- `testDemoReadmeWire` — `test/e2e.js:11979` — 回归该主题行为，含边界与负对照。
- `testDemoTiming` — `test/e2e.js:12043` — 回归该主题行为，含边界与负对照。
- `testLookPrefsStatic` — `test/e2e.js:12083` — 回归该主题行为，含边界与负对照。
- `testUiNoRawMarkdown` — `test/e2e.js:12168` — 回归该主题行为，含边界与负对照。
- `testEngineStoppedSurfacing` — `test/e2e.js:12234` — 回归该主题行为，含边界与负对照。
- `testEngineContextParity` — `test/e2e.js:12318` — 回归该主题行为，含边界与负对照。
- `testFeedbackAndUsage` — `test/e2e.js:12451` — 回归该主题行为，含边界与负对照。
- `testImInboundMedia` — `test/e2e.js:12657` — 回归该主题行为，含边界与负对照。
- `testImCredentialGuard` — `test/e2e.js:12814` — 回归该主题行为，含边界与负对照。
- `testUpdaterVersions` — `test/e2e.js:12904` — 回归该主题行为，含边界与负对照。
- `testCompletionGate` — `test/e2e.js:13040` — 回归该主题行为，含边界与负对照。
- `testStepLines` — `test/e2e.js:13154` — 回归该主题行为，含边界与负对照。
- `testStepLinesWired` — `test/e2e.js:13231` — 回归该主题行为，含边界与负对照。
- `testLarkCliParse` — `test/e2e.js:13276` — 回归该主题行为，含边界与负对照。
- `testLarkSecretNotClobbered` — `test/e2e.js:13347` — 回归该主题行为，含边界与负对照。
- `testOutNameKeepsExt` — `test/e2e.js:13460` — 回归该主题行为，含边界与负对照。
- `testSkillRenameKeepsAssets` — `test/e2e.js:13494` — 回归该主题行为，含边界与负对照。
- `testWindowsLaunch` — `test/e2e.js:13572` — 回归该主题行为，含边界与负对照。
- `testSessionCacheReload` — `test/e2e.js:13717` — 回归该主题行为，含边界与负对照。
- `testRunOwnership` — `test/e2e.js:13842` — 回归该主题行为，含边界与负对照。
- `testSessionIndex` — `test/e2e.js:13953` — 回归该主题行为，含边界与负对照。
- `testCanvasDataLoss` — `test/e2e.js:14152` — 回归该主题行为，含边界与负对照。
- `testUpgradeMigration` — `test/e2e.js:14270` — 回归该主题行为，含边界与负对照。
- `testDramaAssets` — `test/e2e.js:14420` — 回归该主题行为，含边界与负对照。
- `testDramaPipeline` — `test/e2e.js:14523` — 短剧进度、占位符、磁盘真实存在判定。
- `testDramaCompose` — `test/e2e.js:14806` — 拼片计划、字幕时间轴、配乐混音参数。
- `testDramaCast` — `test/e2e.js:15347` — 回归该主题行为，含边界与负对照。
- `testDramaShotRefs` — `test/e2e.js:15580` — 回归该主题行为，含边界与负对照。
- `testDramaVoice` — `test/e2e.js:15731` — 回归该主题行为，含边界与负对照。
- `testDramaBoardExpand` — `test/e2e.js:15906` — 回归该主题行为，含边界与负对照。
- `testDramaBoardWriteback` — `test/e2e.js:16127` — 回归该主题行为，含边界与负对照。

## 6. 第三方运行时库与源码边界

业务逻辑以本仓库 JS 为准。`node_modules` 不入库。打包时 `scripts/check-package-files.js` 沿 require 图检查发行物是否缺文件、是否夹带死重量。阅读源码不必展开 mermaid 或 electron 内部。

技能目录中的 XSD/XML 模板来自 Office 标准，用于校验与生成，不应当作「项目自己发明的数据模型」。

## 7. 生成与同步

`docs/stats.json` 由 `npm run stats` 更新，README 徽章读取。改技能或工具数量后应跑 stats。本 SOURCE.md 由源码扫描生成说明性文字，行号随提交变化，函数名相对稳定。

## 8. 结语

OpenWorkBuddy 的源码是一份**把 Agent 做成产品**的完整标本：入口多、工具真、闸门真、测试真、文件真。文件数与行数会继续涨，但分层稳定：入口 → server → agent → tools/llm → 磁盘。抓住这一条，任何新文件都可以判断它该放在哪一层，以及该用哪一类测试钉住。
