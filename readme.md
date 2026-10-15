# umeng_link_testLink · 友盟超链测试页

为 [fl_umeng_link](https://github.com/Wayaer/fl_umeng/tree/main/fl_umeng_link)（含鸿蒙 OHOS 适配）设计的深链测试落地页。

**纯静态单文件**（`index.html`，无构建步骤、无外部依赖），部署在 Cloudflare Pages 后得到一个公网 HTTPS 地址，用于在真机 / 模拟器上验证超链回调参数。

## 功能

| 功能 | 说明 |
| --- | --- |
| 链接构造 | scheme / host / path / 自定义 key-value 参数，内置 3 种预设 |
| 解析预览 | 与 OHOS 端 `url.URL.parseURL` 同语义（WHATWG），直接对照 App 端 `UMLinkResult.toMap()` 的 `path / uri / linkParams / installParams` |
| 打开 App | 页面直接跳转深链，测试热启动 `onLink` 回调 |
| 二维码 | 「深链直达」扫码即拉起；「落地页中转」更接近友盟 ULink 线上链路，支持 `auto=1` 自动跳转 |
| 配置分享 | 落地页链接自带 `?scheme=&host=&path=&k=v`，打开即恢复配置 |
| 历史记录 | 最近 8 条链接（仅保存在本机浏览器 localStorage） |

### 内置预设（取自 example_auto OHOS skills 实配）

1. **OHOS 自定义 scheme**：`flumenglinkdemo://demo/page/detail?page=home&id=1001`
2. **OHOS App Linking**：`https://example.com/demo/https?from=cloudflare`（需 AGC 域名校验）
3. **Android scheme（README 示例）**：`wellyoung:///page/detail?id=1001`

## 本地预览

直接双击 `index.html` 打开即可；或启动本地服务：

```bash
python -m http.server 8000
# 浏览器打开 http://localhost:8000
```

## 提交到 GitHub

在项目目录执行（示例，远程地址换成你自己的仓库）：

```bash
git init
git add .
git commit -m "feat: 友盟超链测试页 umeng_link_testLink"
git branch -M main
git remote add origin https://github.com/<你的用户名>/umeng_link_testLink.git
git push -u origin main
```

## 部署到 Cloudflare Pages

### 方式 A：连接 GitHub（推荐，push 自动部署）

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/) → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
2. 授权并选择 `umeng_link_testLink` 仓库
3. 构建配置全部留空：
   - Framework preset: `None`
   - Build command: （空）
   - Build output directory: `/`
4. Save and Deploy，得到地址 `https://<项目名>.pages.dev`

之后每次 `git push` 自动重新部署。

### 方式 B：wrangler CLI 直接上传

```bash
npx wrangler login
npx wrangler pages deploy . --project-name umeng-link-test
```

首次执行会提示创建项目，生产地址为 `https://umeng-link-test.pages.dev`。

## 与 fl_umeng_link 插件的对应关系

| 页面字段 / 预览 | App 端（Dart） | OHOS 端来源 |
| --- | --- | --- |
| path 预览 | `UMLinkResult.path` | `url.URL.parseURL(uri).pathname` |
| linkParams 预览 | `UMLinkResult.linkParams` | `parsed.params`（query 逐项 decode） |
| 完整链接 | `UMLinkResult.uri`（getLaunchParams / onInstall） | Want.uri / deferredLink |
| installParams | `UMLinkResult.installParams` | `deferredLink.popDeferredLink()`，需应用市场配置 |

OHOS 端 skills 配置（`module.json5`）需与链接匹配：

```json5
"skills": [{
  "entities": ["entity.system.browsable"],
  "actions": ["ohos.want.action.viewData"],
  "uris": [{ "scheme": "flumenglinkdemo", "host": "demo" }]
}]
```

## 测试场景速查

- **热启动 onLink**：App 后台 → 页面「打开 App」或扫码 → `onLink` 的 path / linkParams 与页面解析预览一致
- **冷启动 getLaunchParams**：App 未运行 → 链接拉起 → 快照含完整 path / uri / linkParams
- **onInstall（deferred link）**：需应用市场配置延期获取链接，本页无法模拟；先验证 onLink 链路，再上真机验证
