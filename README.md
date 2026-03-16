## MetaTube Hugging Face 部署步骤

### 目标

把一个**已经包含 `PR #309`、`PR #316` 修复**，并且**内置演员优先级配置**的 `metatube-server` 部署到 Hugging Face Space 上，完成可访问、可测试的后端服务。

本方案默认：

- 使用你自己的 GitHub fork
- 基于最新 `main` 源码构建
- 不直接依赖官方 `latest` 稳定镜像
- 内置以下演员优先级配置：
  - `MT_ACTOR_PROVIDER_PRIORITY_AV-LEAGUE=1000`
  - `MT_ACTOR_PROVIDER_AV-LEAGUE__PRIORITY=1000`

---

## 第 0 步：准备账号

你需要先准备：

- **GitHub 账号**
- **Hugging Face 账号**

建议先完成：

1. 注册并登录 GitHub
2. 注册并登录 Hugging Face
3. 在 GitHub 中 fork：`metatube-community/metatube-sdk-go`
4. 确认你的 fork 已同步 upstream 最新 `main`

> 这一步的意义是：让最终构建一定包含 `PR #309` 和 `PR #316`，而不是继续使用旧稳定镜像。

---

## 第 1 步：创建自己的 GitHub fork

1. 打开 `https://github.com/metatube-community/metatube-sdk-go`
2. 点击 **Fork**
3. 创建到你自己的账号下
4. 进入你的 fork
5. 确认默认分支是 `main`
6. 如果 fork 比较旧，先同步 upstream 最新 `main`

你最终要得到的是：

- 一个你自己名下的仓库
- 分支是最新 `main`
- 已经包含：
  - `PR #309`：`fc2hub` 跳转修复
  - `PR #316`：`fanza` GraphQL 修复

---

## 第 2 步：创建 Hugging Face Space

在 Hugging Face 中创建新 Space，推荐配置：

- **SDK**：`Docker`
- **Template**：`Blank`
- **Hardware**：`CPU Basic`
- **Visibility**：先按你的需求选 `Public` 或 `Private`

创建完成后，进入这个 Space 的文件区。

如果你打算通过 Git 推送到 Hugging Face，而不是网页直接编辑文件，请顺手在 Hugging Face 的 **Settings -> Access Tokens** 里创建一个 **Write** 权限的 token，后面推送时会用到。


---

## 第 3 步：把模板文件放进 Space

本仓库已经提供了可以直接使用的模板目录：`hf-space-template`

你需要把其中两个文件复制到 Hugging Face Space 根目录：

- `hf-space-template/README.md`
- `hf-space-template/Dockerfile`

复制后，**只改一个地方**：

- 把 `Dockerfile` 中的 `YOUR_GITHUB_USERNAME` 改成你自己的 GitHub 用户名

如果你的 fork 仓库名不是 `metatube-sdk-go`，也一并改掉。

---

## 第 4 步：确认模板里已经包含的关键内容

这套模板已经帮你做了这些事：

- 使用你的 GitHub fork 的最新 `main`
- 在 Hugging Face 上**从源码构建** `metatube-server`
- 默认监听 `8080`
- 使用 `/data/metatube.db` 作为 SQLite 数据库
- 自动打开数据库迁移：`DB_AUTO_MIGRATE=1`
- 内置演员优先级配置：
  - `MT_ACTOR_PROVIDER_PRIORITY_AV-LEAGUE=1000`
  - `MT_ACTOR_PROVIDER_AV-LEAGUE__PRIORITY=1000`

这意味着：

- `fanza` / `fc2hub` 的两个关键修复是通过**源码来源**保证的
- 演员重复问题是通过**优先级配置**一并融进去的

---

## 第 5 步：提交并等待构建

你可以直接在 Hugging Face 网页里保存文件，也可以通过 Git 推送。

保存后，Hugging Face 会自动开始构建。

你要观察的是：

- Space 状态是否从 Building 变成 **Running**
- 构建日志里是否出现编译错误

---

## 第 6 步：先做最小验证

当 Space 变成 **Running** 后，先访问根路径：

- `https://<你的hf域名>/`

正常时应该能看到 JSON 响应。

在 Windows PowerShell 中也可以测试：

```powershell
Invoke-RestMethod "https://<你的hf域名>/"
```

如果根路径都不通，先不要测插件，直接看 `TROUBLESHOOTING.md`。

---

## 第 7 步：验证 provider 是否正常加载

再测试：

- `https://<你的hf域名>/v1/providers`

PowerShell 示例：

```powershell
Invoke-RestMethod "https://<你的hf域名>/v1/providers"
```

你主要看：

- 能否成功返回 provider 列表
- 列表里是否能看到你关心的 provider
- 至少注意这些来源：
  - `fanza`
  - `fc2hub`
  - `AV-LEAGUE`
  - `GFRIENDS`

---

## 第 8 步：先不要加 `TOKEN`

第一次部署和第一次测试，**建议先不设置 `TOKEN`**。

原因：

- 先跑通最重要
- `movies` 相关接口在私有路由组下面
- 先不开鉴权，排错最简单

推荐顺序是：

1. 先部署成功
2. 先测 `/`
3. 再测 `/v1/providers`
4. 再进插件测试
5. 最后再决定要不要加 `TOKEN`

---

## 第 9 步：在 Jellyfin / Emby 插件中接入

在 MetaTube 插件配置页中填写：

- **Server URL**：`https://<你的hf域名>`
- **Server Token**：第一轮测试先留空

保存后，开始做插件层面的实际验证。

---

## 第 10 步：测试 `fanza`

建议优先用你手上已知会命中 `fanza` 的样本。

测试方式：

1. 打开插件中的手动识别
2. 优先尝试两种输入：
   - 直接粘贴影片完整 URL
   - 输入影片番号后搜索
3. 观察是否能正确返回：
   - 封面
   - 背景
   - 演员
   - 其他元数据

你在这一步要确认的是：

- 旧版 `fanza` GraphQL 报错是否已经消失

---

## 第 11 步：测试 `fc2hub`

再用一个你之前失败过的 `fc2hub` 样本测试。

测试方式：

1. 在插件手动识别里粘贴完整 URL
2. 执行识别
3. 观察是否能正常抓取并返回数据

你在这一步要确认的是：

- HTTP → HTTPS 跳转问题是否已经消失

---

## 第 12 步：测试演员重复问题

如果你之前遇到过演员重复，再拿**同一个样本**重新测一次。

你要观察的是：

- 演员是否仍然重复
- 是否仍然串到错误演员源

本仓库模板已经把下列配置融入：

- `MT_ACTOR_PROVIDER_PRIORITY_AV-LEAGUE=1000`
- `MT_ACTOR_PROVIDER_AV-LEAGUE__PRIORITY=1000`

所以正常情况下，这个问题应当比默认配置更稳。

---

## 第 13 步：一切正常后，再决定是否加 `TOKEN`

如果你只是测试，可以先不加。

如果你准备长期使用，尤其是公网访问，建议在 Hugging Face Space 的变量中增加：

- `TOKEN=<你自己的密钥>`

然后在插件中同步填入同样的 `server token`。

---

## 本仓库里的文件说明

- **`README.md`**：精简部署步骤
- **`TROUBLESHOOTING.md`**：构建失败 / 运行失败时怎么办
- **`hf-space-template/README.md`**：直接复制到 HF Space 的 README 模板
- **`hf-space-template/Dockerfile`**：直接复制到 HF Space 的 Dockerfile 模板

---

## 最短执行路径

如果你只想记最短路径，就照这个顺序：

1. fork 官方仓库并同步最新 `main`
2. 创建 Hugging Face Docker Blank Space
3. 把 `hf-space-template` 下两个文件复制进去
4. 修改 `Dockerfile` 里的 GitHub 用户名
5. 保存并等待 Running
6. 测 `https://<你的hf域名>/`
7. 测 `https://<你的hf域名>/v1/providers`
8. 在插件里测 `fanza`
9. 在插件里测 `fc2hub`
10. 在插件里复测演员重复样本
11. 最后再决定是否加 `TOKEN`
