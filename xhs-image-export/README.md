# 小红书图文导出包（xhs-image-export）

> 把 HTML 页面里的图卡，一键批量导出为高清 PNG，直接发小红书图文（抖音竖版同样适用）。
> 解决"截图要缩小、缩小后画质模糊"的问题。

---

## 这是什么

做小红书/抖音图文时，先用 HTML 写好图卡（每个 div 就是一张图），点一下按钮，浏览器自动把每张图卡导出为 2 倍高清 PNG。不需要截图软件，不需要手动裁剪，画质无损。

原理：用 [html-to-image](https://github.com/bubkoo/html-to-image) 库，直接对原始尺寸的 DOM 元素渲染成 PNG blob，通过浏览器自动下载。

- 截图：先缩小页面再截，像素被压缩，画质模糊
- 本方案：`pixelRatio: 2` 输出 2 倍高清，不损失任何像素

---

## 快速开始（3 步）

1. 把本目录的 `demo.html` 和 `lib/` 文件夹下载到**同一个目录**（保持相对路径不变）
2. 用 Chrome 打开 `demo.html`
3. 点右上角紫色按钮「导出全部图片」——浏览器自动逐张下载 PNG，下载完成弹出提示

图片保存在浏览器默认下载目录，文件名格式：`图文_第1张.png`、`图文_第2张.png`...

---

## HTML 结构要求

页面里每张图用一个带 `.page` 类的 div 标记，固定尺寸（小红书 3:4 = 1080×1440px）：

```html
<div class="page" style="width:1080px;height:1440px;"> 第1张图的内容 </div>
<div class="page" style="width:1080px;height:1440px;"> 第2张图的内容 </div>
```

一个 HTML 文件放几张 `.page` 就导出几张图，不需要拆多个文件。

## 核心代码

在 `</body>` 前加入以下代码（就是 demo.html 里的，可以直接抄）：

```html
<script src="lib/html-to-image.min.js"></script>
<script>
window.onload = ()=>{
  const btn = document.createElement('button');
  btn.innerText='导出全部图片';
  btn.className='btn btn-all';
  btn.onclick=async ()=>{
    btn.innerText='生成中...';
    const pages = document.querySelectorAll('.page');
    for(let i=0;i<pages.length;i++){
      const blob = await htmlToImage.toBlob(pages[i], {pixelRatio: 2, width: 1080, height: 1440});
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = `图文_第${i+1}张.png`;
      a.click();
      await new Promise(r=>setTimeout(r,800));
    }
    btn.innerText='导出全部图片';
    alert('图片下载完成！');
  };
  document.body.appendChild(btn);
};
</script>
```

## 关键参数

| 参数 | 值 | 作用 |
|------|-----|------|
| `pixelRatio` | `2` | 2倍高清输出，1080×1440 的页面导出为 2160×2880 的 PNG |
| `width` / `height` | `1080` / `1440` | 导出区域固定尺寸（小红书 3:4）。抖音竖版改成 1080×1920 |
| `setTimeout` | `800` | 每张图间隔 800ms，避免浏览器下载队列堵塞 |
| `.page` | CSS 选择器 | 匹配所有图卡元素，可改成 `.card` 等其他选择器 |

## 踩坑记录（重要）

### 1. CDN 引入库，按钮点击没反应

- **现象**：用 `<script src="https://unpkg.com/...">` 引入库，按钮出现了但点击没反应
- **根因**：Edge/Chrome 的跟踪防护拦截了 CDN 请求，库没加载成功
- **解决**：把 `html-to-image.min.js` 下载到本地，用相对路径引用（本包就是这么给的）

### 2. unpkg 路径 404

- **现象**：`https://unpkg.com/html-to-image@1.11.11/dist/html-to-image.min.js` 返回 404
- **根因**：unpkg 的路径结构可能变化，或包版本路径不对
- **解决**：换 jsdelivr：`https://cdn.jsdelivr.net/npm/html-to-image@1.11.11/dist/html-to-image.min.js`（但还是本地引入最稳）

### 3. 页面打开是白的/样式错乱

- **现象**：直接双击打开时浏览器窗口小于 1080px，看起来错乱
- **解决**：不影响导出结果——导出按固定的 width/height 渲染，跟浏览器窗口大小无关。缩放浏览器到 50% 看全貌即可

## 改造成你自己的

1. 改 `.page` 里面的内容和样式（每张图就是一个普通 div，随便写 HTML/CSS）
2. 图卡的尺寸按平台调：小红书 3:4（1080×1440），抖音 9:16（1080×1920）
3. 图片文件名里的文案按需改（`a.download = ...`）
4. 导出图片里要嵌照片时，把图片放同目录，HTML 里用文件名引用：`<img src="xxx.png" onerror="this.style.display='none'">`

## 适用场景

- 小红书图文（3:4 图卡）
- 抖音图文（9:16 竖版卡片）
- 任何需要把 HTML 页面导出为图片的场景（海报、清单图、分享卡片等）

## 目录结构

```
xhs-image-export/
├── README.md                  # 本文件
├── demo.html                  # 可直接运行的示例（2张图卡 + 导出按钮）
└── lib/
    └── html-to-image.min.js   # 本地库文件（必须跟 HTML 放在一起引用）
```

## License

MIT
