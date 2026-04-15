# 自 `a3eb06` Merge 以来的 Upstream CHANGELOG

## 范围与同步结论

- 基线提交：`a3eb0667336259bcf4032dcb3f1daf73e4fe6f66`
  - 2026-04-10
  - `Merge pull request #24 from HyacinthHaru/fix/project-bugs`
- 上游最新：`096362ab492ba220bebddf709ee2d57b9513f045`
  - 2026-04-14
  - `Merge pull request #31 from Coldin04/refactor/homepage`
- 数据来源：GitHub API 比对 `Coldin04/Cyime`
  - 本地 `git fetch` 在当前环境下没有成功返回，因此这里的“上游最新”是通过远端 API 校验得到，不是本地 refs 已经完成同步。
- 当前关系：
  - 上游父仓库：`Coldin04/Cyime`
  - 本地 fork：`HyacinthHaru/CyimeWrite`
  - `HyacinthHaru:main` 相对上游 `main` 处于 `behind`，落后 `34` 个提交。
- 本次 changelog 统计区间：
  - `a3eb06..096362ab`
  - 共 `26` 个提交

## 快速评估

### 1. 项目当前状态

项目处于明显的高频演进期，而且方向比较集中，不像是散乱修补。最近几天的主线变化主要围绕四个主题展开：

- 认证与输入安全加固
- 编辑器与协作落盘逻辑统一
- 文档配额与回收站规则收紧
- 品牌/UI/首页体验重构

这说明项目并不是“停滞仓库”，而是在快速往“可发布、可运营”的状态收敛。

### 2. 架构成熟度

从仓库结构和 README 看，整体架构仍然稳定：

- `packages/server`：Go API 服务
- `packages/web`：SvelteKit SSR 前端
- `packages/realtime`：独立 Yjs/Hocuspocus 实时协作服务

这种拆分对部署和问题隔离是加分项，尤其适合后续把 realtime 与普通 HTTP 生命周期分开治理。

### 3. 质量与风险判断

正向信号：

- 后端已有一批针对 `auth`、`media`、`content`、`workspace` 的 Go 单测文件。
- 近期提交里能看到明显的安全意识，例如 OAuth 账号绑定校验、粘贴图片 URI 过滤、协作开关降级处理、配额边界补强。
- 改动主题虽然多，但大多围绕已有能力做收口，而不是临时发明新接口。

需要注意的风险：

- 最近提交大量触达 `auth / collaboration / workspace / homepage` 这些跨模块路径，回归面不小。
- 2026-04-14 的品牌与首页重构和协作开关降级是典型“容易引发边缘回归”的区域。
- 当前环境下没能完成一次完整本地验证：
  - `go test ./...` 受网络限制，缺少 Go 模块下载能力。
  - 机器上没有可直接调用的 `pnpm`，所以未执行 web 的 `check/build`。

### 4. 建议的回归重点

如果下一步要真正同步并验收上游版本，优先回归以下路径：

- GitHub OAuth 登录、邮箱绑定、已有账号合并
- 编辑器 autosave 与 realtime 同时在线/离线切换
- 协作关闭时的公开访问、所有者操作、共享入口隐藏
- 文档配额统计、回收站恢复、超额提示
- 首页资源、暗黑/亮色预览图、路由守卫加载态

## CHANGELOG

## 安全与账号体系

- 修复 GitHub OAuth 在账号绑定前的邮箱校验问题，降低错误合并账号的风险。
- 编辑器侧阻止不安全的粘贴图片 URI，补上一条存储型 XSS 防线。
- 为认证提供商增加可选显示名，改善多登录源场景下的可读性与配置表达。

相关提交：

- `231f4b0` Fix GitHub OAuth email verification before account linking
- `52f0a31` Merge pull request #25 from Coldin04/codex/fix-oauth-email-merge-vulnerability
- `951decb` fix(web): block unsafe pasted image URIs in editor sanitizer
- `7ad6632` Merge pull request #26 from Coldin04/codex/fix-stored-xss-due-to-base64-image-support
- `addc19e` Add optional provider display name

## 编辑器与实时协作

- 引入 Yjs canonical content 持久化，说明主线已经开始收敛“协作状态”和“最终落盘状态”的一致性。
- 增加全局协作开关，意味着系统开始支持“有协作”和“无协作”两种部署/运行形态。
- 本地化协作 UI，并对 autosave 进行收敛，减少协作态与 HTTP 自动保存并存时的抖动。
- 在关闭协作的情况下，补齐公开访问与 owner 控制逻辑，避免功能开关关闭后把正常只读/公开链路也一并打坏。

相关提交：

- `550e7a2` Persist canonical content with Yjs support
- `19ea2dc` Add global collaboration feature toggle
- `8438f39` Localize collaboration UI and clamp autosave
- `511eeb7` Merge pull request #27 from Coldin04/fix/editor-save-unification
- `54fdb13` Allow public access and owner controls with collab off

## 文档配额与回收站规则

- 文档配额开始把回收站内容计入统计，规则比之前更严格，也更接近用户真实占用。
- 在恢复回收站项目前做配额预检查，避免“恢复动作执行到一半才失败”。
- UI 上补充展示活动文档数与已删除文档数的合计，和新的配额语义保持一致。

相关提交：

- `673dce6` Enforce document quota including trashed items
- `9b092e1` Precheck document quota in RestoreTrashedItems
- `04eb980` Merge pull request #28 from Coldin04/fix/document-quota-include-trash
- `5432dc7` Show combined active and trashed doc count
- `d93f128` Merge pull request #29 from Coldin04/fix/document-quota-include-trash

## 品牌、视觉与首页

- 项目名称从 `CyimeWrite` 正式收敛为 `Cyime`。
- 主视觉从偏 teal 调整到 sky，整体品牌色更统一。
- 首页加入 Logo 组件、语言切换、页脚链接、深浅色预览图，首页已开始承担真正的产品展示职责。
- `RouteAuthGuard` 与首页加载态被持续重构，说明作者在修“首屏体验”和“未登录访问路径”的细节问题。

相关提交：

- `ee3c04d` Rename project from CyimeWrite to Cyime
- `abd9bc0` Switch folder UI color from teal to sky
- `f85a657` Switch primary color to sky and refine UI
- `fbdf733` Use Logo component and add locale selector
- `53bff46` Update homepage layout and footer links
- `9889722` Address PR review feedback: fix typo, paths, permissions section, and init prompt
- `5975841` Merge pull request #30 from Coldin04/refactor/homepage
- `bffb4bc` Add dark/light homepage preview images
- `840d57c` Address review feedback: fix RouteAuthGuard loading states and homepage dark-mode image strategy
- `49f6a78` Refactor RouteAuthGuard: extract showLoadingScreen derived for readability
- `096362a` Merge pull request #31 from Coldin04/refactor/homepage

## 一句话总结

`a3eb06` 之后的上游主线可以概括为：

先把你这边合进去的安全/协作修复接住，然后继续补安全边界、统一编辑器落盘语义、收紧配额规则，最后快速推进一轮品牌与首页重构。

如果准备跟进上游，这一段历史整体值得同步，但建议把它视作一次“小版本升级”，而不是一次无风险的日常快进。
