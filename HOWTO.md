# 博客操作手册

> 站点地址：https://blog-debug-car.pages.dev
> 本地目录：`~/blog`
> 本文写给自己看的 —— 忘了怎么发文章就翻这里。

---

## 一、日常写文章（最常用）

```bash
cd ~/blog

# 1. 用模板开一篇新文章
cp content/posts/_template.md content/posts/我给这篇起的名字.md

# 2. 写完后本地预览（能实时刷新）
nix shell nixpkgs#hugo --command hugo server -D
#    浏览器打开 http://127.0.0.1:1313
#    看完按 Ctrl+C 停止

# 3. 满意了就发布
git add -A
git commit -m "新增：文章标题"
git push
```

**推上去之后什么都不用做**，Cloudflare 会在 1-2 分钟内自动构建上线。

> ⚠️ `nix shell` 那一长串里，**参数不要放进引号**，否则会报 `No such file or directory`。

---

## 二、几个必须知道的点

### 1. 文件名决定网址

```
content/posts/oled-menu.md  →  https://blog-debug-car.pages.dev/posts/oled-menu/
```

- **不要用中文文件名**（会变成 URL 编码，很难看）；
- 用短横线连接：`line-tracking-pid.md` 而不是 `巡线PID.md`。

### 2. `_template.md` 是模板，且是草稿

它开头写着 `draft: true`，**线上看不到它**，只有加 `-D` 本地预览才会显示。

- 复制它来写新文章 ✅
- 直接改它 ❌（模板就没了）

### 3. 文章开头的 `draft` 字段

```yaml
draft: false   # 发布
draft: true    # 草稿，线上不显示
```

**写完记得改成 `false`**，否则推上去了但看不到，容易懵。

### 4. 本地预览和线上不一致是正常的

| | 本地 `hugo server -D` | 线上 |
|---|---|---|
| 草稿 | 显示 | 不显示 |
| 内容 | 实时刷新 | 要等重新构建 |

---

## 三、图片怎么放

把图片丢进 `static/images/`，文章里这样引用：

```markdown
![描述文字](/images/图片名.png)
```

⚠️ **贴终端截图前一定打码**，确认没有：

- 服务器 IP（就是那台香港机器的地址，自己心里有数就行，**别写进任何文件**）
- SSH 命令、密钥路径、主机别名
- 任何跟代理/节点有关的内容

这是这个博客**唯一的红线**：只写项目技术，不写翻墙。

> 这条同样适用于**这个仓库的所有文件** —— 包括本文。
> 这个仓库是 Public 的，**写进来的东西全世界都能看到**。
> 提交前可以自查一遍：`git grep -nE '([0-9]{1,3}\.){3}[0-9]{1,3}'`

---

## 四、改站点本身

配置文件：`hugo.yaml`

| 想改什么 | 改哪里 |
|---|---|
| 站名 | `title: "我的项目笔记"` |
| 站点描述 | `params.description` |
| 作者名 | `params.author` |
| 默认深浅色 | `params.defaultTheme`（`auto`/`dark`/`light`） |
| 文章目录 | `params.ShowToc` |
| 代码复制按钮 | `params.ShowCodeCopyButtons` |
| 菜单 | `menu.main` 那一段 |

改完一样是 `git push` 生效。

---

## 五、常见问题

### 推上去但线上没变化？
1. 等 2 分钟（构建有延迟）；
2. 去 Cloudflare → `Workers & Pages` → 你的项目 → `Deployments` 看构建状态；
3. 构建失败就点那次部署 → `View build log` 看日志。

### 忘了 `baseURL` / 想换域名？
改 `hugo.yaml` 里的 `baseURL`，推送即可。换域名时：

```yaml
baseURL: "https://你的新域名/"
```

### 想撤销一次改动？
```bash
git log --oneline          # 看历史，找到目标提交
git revert <提交号>        # 生成一个反向提交，安全
git push
```

### 想强制重新部署？
在 Cloudflare 的 Deployments 页面点 `Retry deployment`，或者随便提交一点改动推上去。

---

## 六、目录说明（哪些能动）

```
~/blog/
├── hugo.yaml              ← 站点配置（改动需谨慎）
├── content/
│   ├── about.md           ← 关于页
│   └── posts/             ← ★ 你只写这里
├── static/images/         ← 放图片
├── themes/PaperMod/       ← 主题，别动
└── public/                ← 构建产物，自动生成，不用管
```

**日常只需要碰 `content/posts/` 一个目录。**

---

## 七、环境备忘

| 项目 | 说明 |
|---|---|
| Hugo 版本 | `0.166.0`（Cloudflare 那边由 `HUGO_VERSION` 环境变量锁定，不用管） |
| 本地运行 Hugo | 用 `nix shell nixpkgs#hugo --command ...`，不装进系统 |
| 主题 | PaperMod，已作为普通文件入库（**不是 submodule**，构建零外部依赖） |
| 构建命令 | `hugo --minify` |
| 输出目录 | `public` |
| 分支 | `main` |
| git 代理 | **没有配置**，靠 Clash TUN 模式处理 |
