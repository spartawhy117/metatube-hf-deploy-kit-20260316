## MetaTube Hugging Face 构建与运行排错

### 适用范围

当你在 Hugging Face Space 上部署 `metatube-server` 时，如果遇到：

- 构建失败
- Space 一直不进入 Running
- 根路径打不开
- `/v1/providers` 异常
- `fanza` / `fc2hub` 仍然失败
- 演员还是重复

就按这份文档排查。

---

## 一、构建阶段报错怎么办

### 1. `git clone` 失败

常见原因：

- `Dockerfile` 里还没把 `YOUR_GITHUB_USERNAME` 改掉
- 你的 fork 仓库名不是 `metatube-sdk-go`
- 你的 fork 是 private，HF 无法匿名拉取

处理方式：

- 确认 `GIT_REPO` 写成你自己的真实 GitHub 仓库地址
- 如果只是为了先跑通，建议先把 fork 设为 `public`

---

### 2. `go mod download` 失败

常见原因：

- 构建时网络抖动
- 上游依赖临时不可用

处理方式：

- 先重试一次构建
- 如果连续失败，再看是不是上游依赖源暂时异常

---

### 3. `go build` 失败

常见原因：

- 你的 fork 还没同步到可正常构建的最新 `main`
- 你改坏了 `Dockerfile`
- 上游 main 某个时点本身就有暂时性问题

处理方式：

1. 先确认你的 fork 已同步 upstream 最新 `main`
2. 确认你使用的是仓库模板中的 `Dockerfile`
3. 再次触发构建
4. 如果还是失败，回看日志中的第一条实际错误，而不是最后一条报错

---

## 二、运行阶段报错怎么办

### 1. Space 显示 Running，但根路径 `/` 打不开

优先检查：

- `hf-space-template/README.md` 是否正确复制到了 Space 根目录
- frontmatter 中是否有：
  - `sdk: docker`
  - `app_port: 8080`
- `Dockerfile` 中是否有：
  - `ENV PORT=8080`
  - `EXPOSE 8080`

如果这些都对，再看日志中服务是否启动后立刻退出。

---

### 2. 根路径能打开，但 `/v1/providers` 失败

优先检查：

- 你是不是太早加了 `TOKEN`
- 服务是否完成数据库初始化
- 模板中的 `DB_AUTO_MIGRATE=1` 是否被删掉了

建议：

- 第一轮部署先不要加 `TOKEN`
- 先把基础接口测通，再做鉴权

---

## 三、功能验证失败怎么办

### 1. `fanza` 还是失败

优先检查：

- 你的 fork 是否真的已经包含 `PR #316`
- 你是否实际部署的是源码构建方案，而不是旧 `latest` 思路
- 你的测试样本是否确实命中 `fanza`

处理方式：

1. 到你自己的 fork 里确认 `main` 最新提交
2. 重新触发 Hugging Face 构建
3. 用同一个样本重新测试

---

### 2. `fc2hub` 还是失败

优先检查：

- 你的 fork 是否真的已经包含 `PR #309`
- 你的测试样本是否对应 `fc2hub`
- 你是否还是在旧服务、旧缓存或旧插件配置上测试

处理方式：

1. 确认来源正确
2. 确认 HF Space 已重建成功
3. 确认插件实际连接的是新地址

---

### 3. 演员还是重复

优先检查：

- 模板中的这两个变量是否还在：
  - `MT_ACTOR_PROVIDER_PRIORITY_AV-LEAGUE=1000`
  - `MT_ACTOR_PROVIDER_AV-LEAGUE__PRIORITY=1000`
- 是否已经重新部署并生效
- 是否用的是**同一个重复样本**复测

处理方式：

1. 确认变量仍存在
2. 重新构建或重启
3. 用同一部影片重新测
4. 看 `/v1/providers` 返回里 provider 顺序有没有变化

---

## 四、插件接入后还是不对怎么办

### 1. 插件连到了旧服务

常见原因：

- 插件里还填着旧 URL
- 旧服务还在响应

处理方式：

- 把插件里的 `Server URL` 改成新的 Hugging Face 域名
- 保存后重试

---

### 2. 插件配置缓存没刷新

处理方式：

- 保存插件配置后重启 Jellyfin / Emby
- 再次手动识别或刷新元数据

---

## 五、如果加 `TOKEN` 后测试变复杂怎么办

建议做法：

- **首次部署先不加 `TOKEN`**
- 先完成所有接口和插件测试
- 确认一切正常后，再加 `TOKEN`

如果你已经提前加了 `TOKEN`，又不知道哪里出了问题：

1. 先临时去掉 `TOKEN`
2. 重新验证 `/` 和 `/v1/providers`
3. 等功能完全正常后，再加回去

---

## 六、最常用的排查顺序

遇到问题时，不要乱跳步骤，按这个顺序来：

1. **先看 Hugging Face 构建日志**
2. **先测根路径 `/`**
3. **再测 `/v1/providers`**
4. **再测插件中的 `fanza`**
5. **再测插件中的 `fc2hub`**
6. **最后再测演员重复样本**
7. **最后最后再加 `TOKEN`**

这样排查最快，也最不容易把问题混在一起。
