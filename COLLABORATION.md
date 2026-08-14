# 回森新用户体验健康看板协作说明

> 项目目标：共同构建并部署回森新用户体验健康看板，最终运行在快手内网服务器。
>
> 目标仓库：`https://github.com/MOYi63/fuhua-EHM`

## 1. 当前文件结构

```text
体验监控/
├── dashboard/
│   └── index.html       # 当前看板页面，纯 HTML/CSS/JavaScript
└── COLLABORATION.md     # 双人协作与部署说明
```

当前看板是单页 HTML，不依赖 Node、构建工具或后端服务。页面可以直接用浏览器打开，也可以由 Nginx、Apache 或快手内网静态资源服务托管。

## 2. 两人协作分工

建议固定为两个角色，避免同时修改同一段 CSS：

### 角色 A：页面与交互

负责：

- 页面模块结构与视觉样式
- 核心链路、分项指标卡、状态颜色
- 浏览器兼容性与响应式检查
- 页面交互和展示细节

主要修改文件：

```text
 dashboard/index.html
```

### 角色 B：数据与部署

负责：

- 指标口径确认和模拟数据替换
- 数据字段、接口或静态 JSON 接入
- 内网服务器部署配置
- 发布验证、回滚和运行日志

如果需要修改同一个 HTML 文件，先在群里声明修改区域和预计完成时间，避免两人同时大范围重写文件。

## 3. 分支策略

`main` 只保留可展示、可部署版本。每个需求从 `main` 创建独立分支：

```bash
git clone git@github.com:MOYi63/fuhua-EHM.git
cd fuhua-EHM

git checkout main
git pull --ff-only origin main

git checkout -b feat/dashboard-ui
# 或：git checkout -b feat/dashboard-data
```

分支命名建议：

- `feat/dashboard-ui`：页面结构和视觉调整
- `feat/dashboard-data`：指标、数据接口、模拟数据替换
- `fix/dashboard-layout`：布局或兼容性修复
- `chore/deploy`：部署脚本和服务器配置
- `docs/collaboration`：协作说明和项目文档

## 4. 日常开发流程

### 开始工作前

```bash
git checkout main
git pull --ff-only origin main
git checkout -b feat/你的分支名
```

### 本地预览

纯 HTML 可以直接打开：

```bash
open dashboard/index.html
```

也可以使用静态服务器预览：

```bash
python3 -m http.server 8080 --directory dashboard
```

然后访问：`http://127.0.0.1:8080`

### 提交前检查

```bash
git status
git diff --check
```

浏览器检查重点：

- 页面没有横向滚动条
- 顶部评价和结论对齐
- 核心链路 5 个节点完整显示
- A/B/C/D/E 分项卡片不溢出、不裁切
- 模拟数据提示清晰可见
- Chrome 和快手内网实际浏览器中字体、间距正常

### 提交与推送

```bash
git add dashboard/index.html COLLABORATION.md
git commit -m "feat: update user experience health dashboard"
git push -u origin feat/你的分支名
```

提交信息建议使用以下格式：

```text
feat: 新增功能
fix: 修复问题
docs: 更新文档
chore: 工程配置
```

## 5. 合并规则

1. 不直接向 `main` 推送个人开发代码。
2. 每次改动通过 Pull Request 合并。
3. PR 标题写清楚影响范围，例如：
   - `feat: optimize dashboard metric layout`
   - `feat: replace mock metrics with production data`
4. PR 描述至少包含：
   - 改动内容
   - 截图或录屏
   - 本地验证方式
   - 是否涉及数据口径变化
   - 是否需要部署配置变化
5. 页面改动和数据改动尽量拆成两个 PR，方便回滚和 Review。
6. 合并前必须由另一位同事至少 Review 一次。

## 6. 数据接入约定

当前页面中的指标是模拟数据。真实数据接入前，先确认以下内容：

- 指标名称和展示名称
- 计算口径
- 时间范围
- 分母、分子和去重方式
- 更新频率
- 空值和异常值展示方式
- 合格、观察、不合格的阈值
- 数据权限和内网访问方式

建议不要把真实数据、Token、Cookie、内网账号或接口密钥直接写入 HTML 或 Git 仓库。

推荐的数据接入顺序：

1. 先使用脱敏静态 JSON 验证页面展示。
2. 再接入内网 API 或服务端渲染数据。
3. 前端只保留展示逻辑，不在浏览器端暴露敏感凭证。
4. 在没有数据时显示 `—` 或“暂无数据”，不要用 0 代替未知值。

## 7. 内网部署交接

部署前确认：

- 服务器能够访问构建产物或仓库制品
- 静态文件服务的根目录
- 访问域名和端口
- 是否需要 HTTPS
- 是否需要单点登录或内网鉴权
- 是否需要反向代理到数据接口
- 是否配置缓存策略
- 是否保留上一版本用于回滚

纯静态页面的最小部署内容是：

```text
服务器静态目录/
└── index.html
```

如果使用 Nginx，页面可以由 `root` 指向包含 `dashboard/index.html` 的目录；如果使用内网发布平台，则将 `dashboard/` 作为静态产物目录上传。

发布后验证：

- 首页能正常打开
- CSS 和 JavaScript 加载无 404
- 浏览器控制台无报错
- 看板数据和页面版本一致
- 内网不同网络环境访问正常
- 旧版本可以恢复

## 8. 冲突处理

如果拉取时出现冲突，不要直接覆盖同事改动：

```bash
git status
git fetch origin
git rebase origin/main
```

解决冲突后：

```bash
git add 冲突文件
git rebase --continue
git push --force-with-lease
```

如果不确定如何处理，先保存当前分支并暂停合并：

```bash
git rebase --abort
```

## 9. 当前阻塞项

当前工作目录还不是 `fuhua-EHM` 的 Git clone，且本机无法完成目标 GitHub 仓库认证：

- HTTPS 访问需要 GitHub 登录凭证
- SSH 已有本机密钥，但尚未确认该密钥是否绑定了有仓库权限的 GitHub 账号
- 当前未执行任何远程推送

继续上传前，需要完成以下任一项：

1. 配置有 `MOYi63/fuhua-EHM` 写权限的 GitHub SSH Key；或
2. 在本机配置 GitHub CLI / HTTPS credential；或
3. 由仓库管理员邀请当前 GitHub 账号为 Collaborator。

认证完成后，在本目录执行：

```bash
git clone git@github.com:MOYi63/fuhua-EHM.git repo
cp dashboard/index.html repo/dashboard/index.html
cp COLLABORATION.md repo/COLLABORATION.md
cd repo
git add dashboard/index.html COLLABORATION.md
git commit -m "feat: add user experience health dashboard"
git push origin main
```

如果仓库已有 `dashboard/` 目录，请先检查并合并原文件，不要盲目覆盖。
