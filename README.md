# AI 文案助手

> 单文件纯前端页面：**随机分配 AI 智能体** → **页面内协作完成任务** → **跳转问卷**。

被试打开页面，点击按钮随机分配到三位 AI 智能体中的一位，在页面内嵌的聊天窗口中与 AI 协作完成营销文案，完成后点击按钮进入问卷。

零依赖、零构建、无后端，整个页面就是一个 `index.html`（约 16 KB）。

---

## 页面做了什么

1. **随机分配**：点击「🎲 点击随机分配智能体」，等概率抽中三位智能体之一。
2. **锁定条件**：分配后按钮立即禁用，无法重抽、无法自选，防止被试挑选条件。
3. **记住结果**：分配结果存入浏览器 `localStorage`，刷新或重新打开页面会自动恢复并提示「已恢复随机分配结果」。
4. **内嵌对话**：用 `<iframe>` 嵌入智能体聊天界面，被试无需跳转外部页面；提供「🔄 新对话」按钮可重置会话。
5. **跳转问卷**：协作完成后点击底部按钮，跳转到配置好的问卷链接。

---

## 功能特性

| 特性 | 说明 |
| --- | --- |
| 安全随机 | 使用 `crypto.getRandomValues` + 拒绝采样，消除取模偏置，各条件概率严格相等 |
| 条件锁定 | 以 `currentTab` 作为唯一状态源，已分配则直接拦截重抽请求 |
| 刷新恢复 | 读取 `localStorage` 中的分配结果还原界面，并对值做白名单校验 |
| 会话不缓存 | 加载 iframe 时追加 `_t=<时间戳>`，避免打开到旧的缓存会话 |
| 加载兜底 | 加载动画在 iframe `load` 事件后隐藏，并设 8 秒超时强制隐藏 |
| 防选错提示 | 分配结果区、任务提示区、跳转按钮旁三处提示当前使用的是哪位智能体 |
| 响应式 | 移动端自动调整卡片布局、Tab 尺寸与对话窗口高度 |

---

## 使用与部署

### 本地预览

直接双击 `index.html` 即可；更推荐用本地静态服务器（避免 `file://` 下 iframe 与 `localStorage` 的限制）：

```bash
python -m http.server 8000
# 然后访问 http://localhost:8000
```

### 部署到 GitHub Pages

1. 把 `index.html` 推到仓库默认分支（如 `main`）；
2. 仓库 **Settings → Pages**；
3. Source 选 `Deploy from a branch`，Branch 选 `main` / `/(root)`；
4. 访问 `https://<用户名>.github.io/<仓库名>/`。

同样适用于 Vercel / Netlify / Cloudflare Pages 等任意静态托管：上传 `index.html` 就行。

---

## 配置说明

所有可配置项都在 `index.html` 底部 `<script>` 的开头：

```js
var agents = [
    { position:'左侧', url:'https://udify.app/chat/xxxxxxxxxxxx' },
    { position:'中间', url:'https://udify.app/chat/yyyyyyyyyyyy' },
    { position:'右侧', url:'https://udify.app/chat/zzzzzzzzzzzz' },
];

var ASSIGNMENT_KEY = 'experimentAgentAssignment_v2'; // localStorage 键名
var SURVEY_URL = 'https://www.credamo.com/s/xxxxxx/'; // 问卷跳转地址
```

| 配置项 | 说明 |
| --- | --- |
| `agents[].position` | 智能体的位置标签，显示在 Tab 和提示文案中；建议与问卷中的选项文字保持一致 |
| `agents[].url` | 智能体的对话分享链接，需支持 iframe 嵌入 |
| `ASSIGNMENT_KEY` | `localStorage` 键名。**改成新值（如 `_v3`）即可让所有被试重新分配** |
| `SURVEY_URL` | 协作完成后跳转的问卷地址 |

### 关于内嵌链接

- 当前用的是 **Coze（扣子）智能体分享链接**（`udify.app/chat/...`），页面已为 iframe 设置 `referrerpolicy="origin"` 和 `allow="microphone"`。
- **正式使用前请务必在目标浏览器实测嵌入效果**：部分平台会通过 `X-Frame-Options` 或 CSP `frame-ancestors` 禁止被第三方页面嵌入，被拦截时需换成平台提供的 embed 链接。

---

## 常见问题

<details>
<summary><b>被试刷新页面会不会被重新分配？</b></summary>

不会。结果存在 `localStorage`，同一浏览器再次打开会恢复原分配。
</details>

<details>
<summary><b>换浏览器或换设备进来会不会重新抽签？</b></summary>

会。`localStorage` 与浏览器绑定，无法跨设备同步。若必须跨设备锁定，需要服务端支持，或改用问卷平台自带的随机分流。
</details>

<details>
<summary><b>怎么让下一轮实验所有人都重新分配？</b></summary>

改 `ASSIGNMENT_KEY`（例如 `_v2` 改 `_v3`）。旧记录因键名不同会被忽略。
</details>

<details>
<summary><b>对话窗口一直显示"正在连接智能助手…"</b></summary>

可能是链接失效、平台禁止 iframe 嵌入（控制台会有 `X-Frame-Options` / CSP 报错）或网络问题。8 秒后加载动画会自动消失，可点「🔄 新对话」重试。
</details>

<details>
<summary><b>能不能改成两位或四位智能体？</b></summary>

可以。在 `agents` 数组增删条目，并同步在 `<nav class="tab-nav">` 中增删对应的 `<button class="tab-btn" data-tab="N">`。随机与恢复逻辑都由数组长度驱动，无需其他改动。
</details>

---

## 文件说明

```
.
├── index.html   # 页面本体，唯一必需文件
└── README.md
```

> 目录下的 `index_修改前备份_20260830.html`、`index_随机分配披露人格版备份_20260830.html` 与 `ai-personality-experiment/` 是历史版本，公开仓库建议移除：
>
> ```bash
> git rm -r --cached index_修改前备份_20260830.html \
>                index_随机分配披露人格版备份_20260830.html \
>                ai-personality-experiment
> ```

---

## 许可证

MIT@lsenufy
