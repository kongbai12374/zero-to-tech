# 零基础到技术 · 学习笔记

> 目录
>
> - [2.3 从网址到网页：互联网是怎么工作的](#23-从网址到网页互联网是怎么工作的)
> - [2.4 把网页放到公网：云服务器与 Nginx](#24-把网页放到公网云服务器与-nginx)
> - [3.1 HTML / CSS / JavaScript 是什么](#31-html--css--javascript-是什么)
> - [3.2 三层分工总结与练习](#32-三层分工总结与练习)
> - [3.3 Git 版本管理](#33-git-版本管理)

---

## 2.3 从网址到网页：互联网是怎么工作的

### 2.3.1 IP 地址与域名

| 概念 | 说明 | 例子 |
| --- | --- | --- |
| IP 地址 | 网络中一台设备的唯一编号，机器用 | `93.184.216.34` |
| 域名 | 给人看的名字，好记 | `example.com` |
| DNS | 域名系统，负责把域名"翻译"成 IP，相当于互联网的电话簿 | — |

IP 地址不好记，所以有了域名；但真正通信靠的是 IP，所以需要 DNS 做转换。

### 2.3.2 域名解析（DNS）大致过程

1. 先查浏览器缓存 → 操作系统缓存 → `hosts` 文件
2. 都没有就问本地 DNS 服务器（通常由运营商提供）
3. 本地 DNS 依次询问：根域名服务器 → 顶级域服务器（如 `.com`）→ 权威 DNS 服务器
4. 拿到 IP 后逐层返回并缓存（缓存时间由 TTL 决定）

> 实际开发中，域名解析记录（A 记录）也是在域名服务商后台配置的，把域名指向你的服务器 IP。

### 2.3.3 端口：同一台电脑上的不同入口

服务器是"一栋楼"，IP 是楼的门牌号，端口就是楼里不同的房间门：

| 端口 | 服务 | 说明 |
| --- | --- | --- |
| 22 | SSH | 远程登录服务器 |
| 80 | HTTP | 明文网页 |
| 443 | HTTPS | 加密网页 |
| 3306 | MySQL | 数据库 |
| 6379 | Redis | 缓存 |
| 3000 / 5173 / 8000 | 开发服务 | 本地调试常用，非标准 |

### 2.3.4 网址（URL）的结构

完整示例 `https://www.example.com:443/blog/index.html?id=1#top` 可以拆成：

```text
协议   https://
域名   www.example.com
端口   :443
路径   /blog/index.html
参数   ?id=1
锚点   #top
```

| 部分 | 作用 | 能否省略 |
| --- | --- | --- |
| 协议 `https` | 用哪种方式通信 | 否 |
| 域名 `www.example.com` | 找到哪台服务器 | 否 |
| 端口 `443` | 找到服务器上的哪个服务 | 能（用协议默认端口） |
| 路径 `/blog/index.html` | 要哪个资源 | 能（默认 `/`） |
| 查询参数 `?id=1` | 传给服务器的额外信息 | 能 |
| 锚点 `#top` | 页面内定位，不发给服务器 | 能 |

**域名只是网址的一部分。**

### 2.3.5 输入网址后发生了什么

```text
输入网址
  → 浏览器查 DNS，得到域名对应的 IP
  → 与服务器建立 TCP 连接（HTTPS 还要先做 TLS 握手）
  → 浏览器发送请求（request）
  → 服务器处理请求，把网页内容作为响应（response）返回
  → 浏览器解析 HTML，再请求 CSS / JS / 图片等资源
  → 渲染页面，展示给用户
```

一次 HTTP 响应大致包含：**状态码 + 响应头 + 响应正文**。

| 状态码 | 含义 |
| --- | --- |
| 200 | 成功 |
| 301 / 302 | 永久 / 临时重定向 |
| 304 | 资源未修改，用本地缓存 |
| 403 | 没有权限 |
| 404 | 资源不存在 |
| 500 / 502 | 服务器内部错误 / 网关错误（Nginx 常见） |

### 2.3.6 HTTP 与 HTTPS

| | HTTP | HTTPS |
| --- | --- | --- |
| 默认端口 | 80 | 443 |
| 是否加密 | 否，明文传输 | 是，TLS 加密 |
| 证书 | 不需要 | 需要（Let's Encrypt 可免费申请） |
| 现状 | 已不推荐 | 标准做法 |

---

## 2.4 把网页放到公网：云服务器与 Nginx

### 2.4.1 云服务器选择

- **配置**：学习阶段 1 核 2G 内存足够，够跑 Nginx
- **系统**：Ubuntu 22.04 LTS（资料多、命令统一）
- **地域**：选离自己/用户近的机房，延迟更低
- **计费**：包年包月或按量付费，用完记得关机/释放
- **安全组**：至少放通 `22`（SSH）、`80`（HTTP）、`443`（HTTPS）

### 2.4.2 SSH 远程登录

```bash
# 密码登录
ssh root@<服务器公网IP>

# 使用密钥登录（推荐）
ssh -i ~/.ssh/id_rsa root@<服务器公网IP>
```

> ⚠️ **不要把真实的服务器密码、密钥写进笔记或仓库。** 这里只记录方法，密码用占位符代替。
>
> 安全建议：使用密钥登录、修改默认 22 端口、禁用 root 密码登录、开启防火墙。

### 2.4.3 安装并启动 Nginx

```bash
sudo apt update
sudo apt install -y nginx

sudo systemctl start nginx     # 启动
sudo systemctl enable nginx    # 开机自启
sudo systemctl status nginx    # 查看状态
```

安装后浏览器访问 `http://<服务器公网IP>`，看到 "Welcome to nginx!" 就成功了。

- 配置文件目录：`/etc/nginx`
- 主配置：`/etc/nginx/nginx.conf`
- 站点配置：`/etc/nginx/sites-available/`（启用放到 `sites-enabled/`）
- 默认站点根目录：`/var/www/html`

### 2.4.4 配置一个站点：监听 80 端口，根目录 /var/www/example.com

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/example.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

应用配置：

```bash
# 1. 上传网页文件（在本地执行）
scp -r ./index.html ./style.css ./script.js root@<服务器公网IP>:/var/www/example.com/

# 2. 检查配置语法
sudo nginx -t

# 3. 重载生效（不中断服务）
sudo systemctl reload nginx
```

> 注意路径写法：根目录是绝对路径 `/var/www/example.com`，前面必须有 `/`。

最后在域名服务商后台添加 **A 记录**，把 `example.com` 指向服务器 IP —— 这就和 2.3 的 DNS 接上了。

### 2.4.5 常见问题排查

| 现象 | 可能原因 |
| --- | --- |
| 浏览器打不开 | 安全组未放通 80、服务器防火墙拦截、Nginx 没启动 |
| 403 Forbidden | 目录权限不对，或缺少 `index.html` |
| 404 Not Found | `root` 路径写错、文件名不对 |
| 502 Bad Gateway | 后端服务没起来（配置了反向代理时） |
| 改了配置没生效 | 忘记 `nginx -t` + `reload` |

---

## 3.1 HTML / CSS / JavaScript 是什么

| 技术 | 负责 | 一句话 | 类比 |
| --- | --- | --- | --- |
| HTML | 内容和结构 | 页面上有什么 | 骨架 |
| CSS | 样式和外观 | 这些东西长什么样 | 皮肤和衣服 |
| JavaScript | 行为和交互 | 发生了什么、要做什么 | 神经和肌肉 |

### 3.1.1 第一个网页（单文件版）

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8">
    <title>我的第一个网页</title>
    <style>
      body {
        background-color: #f0f4f8;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        margin: 0;
        font-family: sans-serif;
      }

      .card {
        background: white;
        padding: 40px;
        border-radius: 12px;
        text-align: center;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
      }

      h1 {
        color: #2c3e50;
        margin-bottom: 12px;
      }

      p {
        color: #666;
      }

      button {
        margin-top: 20px;
        padding: 10px 24px;
        background-color: #3498db;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-size: 16px;
      }

      button:hover {
        background-color: #2980b9;
      }
    </style>
  </head>
  <body>
    <div class="card">
      <h1>你好，互联网</h1>
      <p id="msg">这是我的第一个网页，现在它还只是一个本地的 HTML 文件。</p>

      <button onclick="changeText()">点我试试</button>
    </div>
    <script>
      function changeText() {
        document.getElementById('msg').textContent = '你刚刚触发了一段 JavaScript。';
      }
    </script>
  </body>
</html>
```

> 常见错误：字符串引号要成对出现。`'你刚刚触发了一段 JavaScript;` 少了一个结尾的 `'`，会导致整段脚本报错、按钮点了没反应。

### 3.1.2 拆成三个文件（本项目结构）

把结构、样式、交互分开，代码更好维护：

```text
zero-to-tech/
├── index.html
├── style.css
└── script.js
```

`index.html`：用 `<link>` 引入 CSS，用 `<script src>` 引入 JS，标签内不再写样式和脚本。

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8">
    <title>我的第一个网页</title>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <div class="card">
      <h1>你好，互联网</h1>
      <p id="msg">这是我的第一个网页，现在它还只是一个本地的 HTML 文件。</p>

      <button onclick="changeText()">点我试试</button>
    </div>
    <script src="script.js"></script>
  </body>
</html>
```

`style.css`：内容和上面的 `<style>` 完全一致，只是搬到了单独文件。

`script.js`：

```javascript
function changeText() {
  document.getElementById('msg').textContent = '你刚刚触发了一段 JavaScript。';
}
```

要点：

- `<link rel="stylesheet" href="style.css">` 放在 `<head>` 里，样式先加载，避免页面闪一下
- `<script src="script.js"></script>` 放在 `</body>` 前，保证 HTML 元素已存在再执行
- `document.getElementById('msg')` 通过 `id` 找到元素，再用 `textContent` 修改文字

### 3.1.3 常用 HTML 标签速查

| 标签 | 作用 |
| --- | --- |
| `h1` ~ `h6` | 标题，逐级变小 |
| `p` | 段落 |
| `div` | 块级容器，用来布局 |
| `span` | 行内容器，用来包一段文字 |
| `a href="..."` | 超链接 |
| `img src="..." alt="..."` | 图片 |
| `ul` / `ol` / `li` | 无序 / 有序列表 |
| `button` | 按钮 |
| `input` / `textarea` | 输入框 |
| `form` | 表单，提交数据 |

### 3.1.4 学习资源

- [MDN Web 文档](https://developer.mozilla.org/zh-CN/) —— 最权威，查标签和属性首选
- [W3School 中文](https://www.w3school.com.cn/) —— 入门教程，例子多
- [菜鸟教程](https://www.runoob.com/) —— 速查手册式教程

---

## 3.2 三层分工总结与练习

### 3.2.1 一句话总结

- **HTML 负责结构**：用标签描述"有什么"
- **CSS 负责样式**：用选择器 + 属性描述"长什么样"
- **JavaScript 负责交互**：监听事件、修改 DOM、发请求

三者通过 **class / id / 标签名** 关联：HTML 提供"钩子"，CSS 和 JS 通过钩子找到元素。

### 3.2.2 动手练习

1. 把 `body` 背景色改成浅绿，观察 `style.css` 生效
2. 给 `.card` 加一张图片（`<img>`），练习路径写法
3. 再加一个按钮，点击后让数字从 0 开始递增（练习变量和事件）
4. 用 `fetch()` 请求一个公开 API，把返回的数据显示在页面上
5. 把整个文件夹通过 `scp` 传到服务器，用域名访问（打通 2.4）

### 3.2.3 下一步要学

- CSS 盒模型、Flex 布局、响应式（媒体查询）
- DOM 操作与事件（`addEventListener`）
- 浏览器开发者工具（F12：Elements / Console / Network）
- 用 Nginx 部署 + 申请 HTTPS 证书

---

## 3.3 Git 版本管理

### 3.3.1 Git 是什么

- **给代码存档**：每次提交都是一个"存档点"，可以随时查看历史、回到任意一个版本
- 分布式版本控制系统，官网：[git-scm.com](https://git-scm.com/)
- **管理范围**：Git 只跟踪它所在仓库目录及其子目录里的文件；仓库外的文件不受管理
- **适合文本类**：代码、Markdown、配置文件可以按行比较差异
- **不适合二进制文件**：图片、视频、压缩包无法按行比对，每次改动都相当于存一整份新文件，会让仓库迅速膨胀（确需管理可用 Git LFS）

### 3.3.2 首次使用：配置身份

```bash
git config --global user.name "kongbai12374"
git config --global user.email "you@example.com"
```

- 命令顺序是 `git config --global user.name`，不是 `git config global.name`
- `--global` 表示对当前用户全局生效，一台电脑配一次即可
- 配置的姓名和邮箱会写进每次提交记录，用来标识"是谁提交的"

### 3.3.3 三个区域与核心命令

```text
工作区（你编辑的文件）  --git add-->  暂存区  --git commit-->  版本库（历史记录）
```

| 命令 | 作用 |
| --- | --- |
| `git init` | 在当前目录初始化仓库，生成隐藏的 `.git` 目录 |
| `git status` | 查看当前状态：哪些文件被修改、哪些已暂存 |
| `git add <文件>` / `git add .` | 把改动放进暂存区 |
| `git commit -m "说明"` | 把暂存区内容提交为一个版本 |
| `git log --oneline` | 每条历史记录显示为一行，查看提交历史 |
| `git checkout <分支名>` | 切换到指定分支，如 `git checkout master` |
| `git checkout -- <文件>` | 丢弃工作区中该文件的修改，恢复到上次提交的状态 |

第一次提交的完整流程：

```bash
cd zero-to-tech
git init
git status              # 会看到 index.html、style.css 等未跟踪文件
git add .
git commit -m "第一个网页"
git log --oneline       # 输出：a1b2c3d 第一个网页
```

### 3.3.4 .gitignore：不跟踪哪些文件

在仓库根目录新建 `.gitignore`，按行写上不想被 Git 管理的文件或目录：

```gitignore
node_modules/
*.log
.env
dist/
```

- 用途：依赖目录、日志、编译产物、含密码的配置文件都不该进仓库
- 注意：`.gitignore` 是"忽略清单"，**不是暂存**；暂存区是 `git add` 把改动放进去的那个区域
- 已经被跟踪的文件后来才写进 `.gitignore` 不会自动失效，需要 `git rm --cached <文件>`

### 3.3.5 分支

- 每条独立的工作线叫分支，默认主分支在旧版本 Git 里叫 `master`，新版本默认叫 `main`
- `git checkout master` = 切回主干分支
- `git checkout` 在新版 Git 里拆成了 `git switch`（切分支）和 `git restore`（恢复文件），功能相同

### 3.3.6 学习资源与练习

- [Git 官方文档 / Pro Git 中文版](https://git-scm.com/book/zh/v2) —— 系统全面
- [廖雪峰官网](https://liaoxuefeng.com/) —— 有完整的中文 Git 教程，适合入门
- 练习：在本目录执行 `git init` → `git add .` → `git commit -m "初始版本"`，再故意改一行 CSS，用 `git status` 和 `git log --oneline` 观察变化
