# NEXREALM 静态页面部署说明

这个版本已经把原来的本地路径 `file:///D:/...` 改成了线上部署可用的相对路径。

## 目录结构

```text
nexrealm-online/
├─ index.html
└─ assets/
   ├─ bg.png       # 背景图，自己替换
   └─ intro.mp4   # 视频，自己替换
```

## 必须做的事

1. 把背景图片放到 `assets` 文件夹，命名为：`bg.png`
2. 把视频放到 `assets` 文件夹，命名为：`intro.mp4`
3. 整个 `nexrealm-online` 文件夹上传到 Vercel / Netlify / GitHub Pages。

## 用 Vercel 最省事的方式

1. 打开 Vercel，新建 Project。
2. 如果你有 GitHub 仓库，就把整个 `nexrealm-online` 文件夹提交到仓库，然后导入。
3. 如果不用 GitHub，也可以用 Vercel CLI：

```bash
npm i -g vercel
cd nexrealm-online
vercel
```

4. 部署成功后，会得到一个类似 `https://xxxx.vercel.app` 的链接。别人点击这个链接，或用二维码扫描这个链接，就可以打开页面。

## 二维码怎么生成

部署完成后，把 Vercel 链接复制出来，使用任意二维码生成器即可。也可以在命令行使用：

```bash
npx qrcode-terminal https://你的链接.vercel.app
```

## 另一种方式：视频用公网 URL

页面支持 URL 参数覆盖默认视频，例如：

```text
https://你的链接.vercel.app/?video=https%3A%2F%2Fexample.com%2Fintro.mp4
```

也支持背景图：

```text
https://你的链接.vercel.app/?bg=https%3A%2F%2Fexample.com%2Fbg.png&video=https%3A%2F%2Fexample.com%2Fintro.mp4
```

这种方式适合把大视频放到对象存储、Cloudinary、Supabase Storage 或 CDN，再让网页读取公网视频链接。
