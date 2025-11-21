# Gemini Manga Localizer

一个完全由 `manga_localizer.html` 组成的单页工具，驱动 Gemini 模型完成漫画翻译、质检和嵌字。直接用浏览器打开即可，无需安装依赖或启动后端。

## 项目结构
- `manga_localizer.html`：包含全部 UI、逻辑和样式。
- `qr_small.b64` 及其他 `.png/.jpg`：页面用到的静态资源。
- `.github/workflows/deploy.yml`：将最新代码部署到 GitHub Pages 的 CI 配置。

## 本地使用
1. 在 Chrome/Edge 等现代浏览器中直接打开 `manga_localizer.html`。
2. 填写翻译/图像模型的 Base URL、Endpoint、API Key，上传漫画原图。
3. 可选：导入或手动编辑术语表、补充翻译指令。
4. 点击「开始汉化」，根据左侧日志和步骤提示完成整个流程。

## 将 Base URL / Key 写死在 HTML
如果你想把常用的 Base URL、Endpoint 或 API Key 直接嵌入 `manga_localizer.html`，可以修改文件底部 `ENCRYPTED_*` 常量。页面会在加载时使用 `decryptValue` 将这些值解密成默认配置。

1. 打开 `manga_localizer.html`，找到如下常量：
   ```js
   const ENCRYPTED_DEFAULT_BASE_URL = "...";
   const ENCRYPTED_TRANSLATION_ENDPOINT = "...";
   const ENCRYPTED_IMAGE_ENDPOINT = "...";
   const ENCRYPTED_TRANSLATION_API_KEY = "...";
   const ENCRYPTED_IMAGE_API_KEY = "...";
   ```
2. 使用以下 Node.js 片段生成新的密文（会与页面里的 `SECRET_TOKEN` 完全一致）：
   ```bash
   node - <<'NODE'
   const secret = "gemini-manga-localizer";
   const encrypt = (value) => {
     let masked = "";
     for (let i = 0; i < value.length; i++) {
       const code = value.charCodeAt(i) ^ secret.charCodeAt(i % secret.length);
       masked += String.fromCharCode(code);
     }
     return Buffer.from(masked, "binary").toString("base64");
   };
   console.log("BASE_URL =", encrypt("https://your-api-host.com"));
   console.log("TRANSLATION_ENDPOINT =", encrypt("/v1beta/models/...:generateContent"));
   console.log("IMAGE_ENDPOINT =", encrypt("/v1beta/models/...:generateContent"));
   console.log("TRANSLATION_KEY =", encrypt("sk-xxx"));
   console.log("IMAGE_KEY =", encrypt("sk-yyy"));
   NODE
   ```
3. 用输出结果替换 HTML 中对应的 `ENCRYPTED_*` 字符串即可。部署时浏览器会直接带出这些默认值，方便私有部署或内网环境使用。

## 部署到 GitHub Pages
1. 创建名为 `gemini-manga-localizer` 的仓库，并将本目录作为仓库根目录推送到 `master` 分支。
2. 确认仓库已开启 GitHub Actions 功能。
3. 推送后，`.github/workflows/deploy.yml` 会自动运行：
   - checkout 当前仓库内容；
   - 上传整个仓库作为 Pages 构建产物；
   - 通过 `actions/deploy-pages` 发布到 `gh-pages` 分支。
4. 在仓库设置中选择 `GitHub Pages -> gh-pages` 作为发布源，即可在 `https://<用户名>.github.io/gemini-manga-localizer/` 访问。

## 备注
- 如果仓库默认分支不是 `master`，记得同步修改 workflow 中的触发分支。
- 发布前请自行补充许可证说明（例如 MIT）。
