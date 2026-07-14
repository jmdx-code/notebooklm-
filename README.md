# NotebookLM 转录登记

这是一个本地加载的 Chrome/Edge 扩展：从当前已登录的 NotebookLM 笔记本逐个读取来源转录稿，可选翻译为中文，并把结果登记到你的 Google Sheets 数据库服务。

## 功能

- 提取每个已有来源的名称和完整转录文字。
- 可选「谷歌翻译」，默认关闭；开启后会自动翻译已提取的所有未翻译文本。
- 复制三列表格到 Excel/Google Sheets；换行会保留在同一个单元格内。
- 登记到 Google Sheets 后端，并显示总结果与失败明细。
- 登记结果显示 `status_counts` 汇总；逐条列表只显示失败记录及后端返回的具体原因。
- 浮动面板可以拖动、最小化为品牌圆形按钮，并可从右下角拉伸宽度和高度。
- 标题与操作区始终固定，只有下方“结果与登记日志”折叠区参与滚动。
- 来源名在提取完成时立即移除文件后缀，界面、复制结果和登记字段保持一致。

登记时发送的记录字段如下：

| 字段 | 值 |
| --- | --- |
| `post_id` | 来源名移除末尾扩展名后的值，例如 `audio.mp3` → `audio` |
| `audio_content` | 完整转录文字 |
| `audio_content_zh` | 中文翻译；未开启翻译时为空字符串 |

## 配置表格服务

1. 在浏览器扩展栏点击「NotebookLM 转录登记」图标。
2. 输入 Apps Script Web App 的部署链接，格式应为 `https://script.google.com/macros/s/.../exec`，然后点击「保存设置」。
3. 在 NotebookLM 浮动面板顶部输入要写入的 Google 表格编辑链接；链接必须包含目标工作表的 `gid`，例如 `https://docs.google.com/spreadsheets/d/.../edit#gid=0`。
4. 表格链接会自动缓存，可以随时在浮动面板中更换。

点击面板的「登记表格」后，扩展将发送：

```text
POST <部署链接>?action=upsert
Content-Type: application/json
```

请求体包含 `database_url` 和 `records`。面板的「登记结果」会显示服务返回的成功/失败汇总及逐条状态。表格必须已按后端服务说明授权给其执行账号写入。

## 使用流程

1. 在 `chrome://extensions`（或 Edge 的 `edge://extensions`）开启开发者模式，并加载本文件夹。
2. 打开 NotebookLM 笔记本并刷新页面。
3. 在左侧浮动面板点击「提取转录」。
4. 如需中文翻译，勾选「谷歌翻译」；可在提取前或提取后勾选。
5. 需要写入时点击「登记表格」；需要手工粘贴时点击「复制」。

## 隐私与限制

- 扩展不上传文件、不创建来源、不修改 NotebookLM 内容，也不会直接读取 Cookie、密码或浏览器个人资料。
- 提取使用 NotebookLM 页面已登录上下文中的来源列表和来源详情请求；NotebookLM 的内部接口和数据结构可能变化。
- 启用翻译后，转录文字会发送到 Google 的免费翻译 Web 端点。该非密钥方式适合个人少量使用，可能会受频率限制或随服务变更而失效。
- 点击「登记表格」后，三列数据会发送到你在扩展图标中配置的 Apps Script 部署链接。

## 如何发布新版本

本项目使用 GitHub Actions 自动构建并发布。每次发布只需创建并推送一个版本 Tag；系统会自动打包扩展、生成构建溯源证明（Attestation），并创建 Release。

### 发布步骤

#### 1. 提交并推送代码

```bash
git status
git add .
git commit -m "你的改动说明"
git push origin main
```

#### 2. 创建并推送版本 Tag

版本号使用 `v主版本.次版本.修订版本` 格式，例如 `v1.0.1`。

```bash
git tag -a v1.0.1 -m "Release version 1.0.1"
git push origin v1.0.1
```

推送后，GitHub Actions 会自动：

1. 校验扩展的 `manifest.json`。
2. 打包根目录包含 `manifest.json` 的 ZIP 文件。
3. 为最终 ZIP 生成 Attestation。
4. 创建 GitHub Release 并由 `github-actions[bot]` 上传 ZIP。

#### 3. 查看结果

- 构建进度：在仓库的 **Actions** 页面查看。
- 发布文件：在仓库的 **Releases** 页面下载。

### 如果构建失败

1. 在 **Actions** 页面查看失败日志并修复代码或工作流。
2. 删除失败的本地和远程 Tag。
3. 重新创建相同版本的 Tag 并推送：

```bash
git tag -d v1.0.1
git push origin :refs/tags/v1.0.1
git tag -a v1.0.1 -m "Release version 1.0.1"
git push origin v1.0.1
```
