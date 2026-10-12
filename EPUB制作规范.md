# 《怀旧的乌托邦》EPUB 结构拆解与制作规范

> 本规范根据用户提供的《怀旧的乌托邦.epub》拆解。书中正文只用作结构样本；示例中的正文和注释内容均为占位文字。文档区分了“样本观察”和“制作要求”，便于复刻原书结构或据此制作同类中文电子书。

## 1. 样本概览

| 项目 | 样本中的实现 |
|---|---|
| EPUB 版本 | EPUB 2.0，包文件为 `OEBPS/content.opf` |
| 压缩包入口 | `mimetype` 位于 ZIP 第一项，未压缩，内容为 `application/epub+zip` |
| 内容格式 | XHTML 1.1；正文按章节分成多个 `.xhtml` 文件 |
| 目录 | `OEBPS/toc.ncx`，由 OPF 的 `spine toc="ncx"` 引用 |
| 样式 | 一份共享 CSS：`OEBPS/Styles/stylesheets.css` |
| 图片与字体 | JPEG 封面、方正仿宋和楷体字体文件 |
| 脚注 | 各章正文末尾的注释段落；正文标记与注释条目用页内锚点双向链接 |
| 文件数量 | ZIP 内 18 项，包括元数据、目录、9 份 XHTML、CSS、封面、字体及辅助文件 |

样本的阅读顺序是封面、版权页、六章正文、译后记。章节文件名从 `chapter2.xhtml` 开始，文件名不代表章节序号；阅读顺序应以 OPF 的 `spine` 为准。六章共观察到 131 个脚注标记和对应注释段落，显示编号在每章内从 `[1]` 重新开始。

## 2. 建议的目录布局

```text
book.epub                         # ZIP 容器
├── mimetype                      # ZIP 第一项，存储不压缩
├── META-INF/
│   └── container.xml             # 指向 OPF 包文件
└── OEBPS/
    ├── content.opf               # 元数据、资源清单、阅读顺序
    ├── toc.ncx                   # EPUB 2 导航目录
    ├── Styles/
    │   └── stylesheets.css
    ├── Images/
    │   └── cover.jpg
    ├── Fonts/
    │   ├── fangzhengfangsong.ttf
    │   └── fangzhengkaiti.ttf
    └── Text/
        ├── cover.xhtml
        ├── Copyright.xhtml
        ├── chapter01.xhtml
        ├── chapter02.xhtml
        ├── ...
        └── afterword.xhtml
```

`META-INF/calibre_bookmarks.txt` 是样本携带的阅读器辅助文件，不是 EPUB 正文、目录或脚注结构的必需文件。制作时只加入确有用途的资源。

## 3. ZIP 与容器入口

### 3.1 `mimetype`

文件必须放在 ZIP 的第一项，且采用不压缩存储。文件内容只写这一行，不添加 BOM、空格或换行：

```text
application/epub+zip
```

### 3.2 `META-INF/container.xml`

容器文件指出包文档的位置。沿用样本时，包文档放在 `OEBPS/content.opf`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<container version="1.0"
  xmlns="urn:oasis:names:tc:opendocument:xmlns:container">
  <rootfiles>
    <rootfile full-path="OEBPS/content.opf"
      media-type="application/oebps-package+xml"/>
  </rootfiles>
</container>
```

## 4. OPF 包文件：元数据、资源、阅读顺序

样本采用 EPUB 2.0 的 OPF。所有正文、图片、CSS、字体和 NCX 都应在 `manifest` 中登记；`spine` 按读者实际阅读顺序引用对应的 `idref`。不要根据文件名排序来推断书内顺序。

简化示例：

```xml
<package version="2.0" unique-identifier="BookId"
  xmlns="http://www.idpf.org/2007/opf">
  <metadata xmlns:dc="http://purl.org/dc/elements/1.1/"
    xmlns:opf="http://www.idpf.org/2007/opf">
    <dc:identifier id="BookId" opf:scheme="UUID">urn:uuid:唯一值</dc:identifier>
    <dc:title>书名</dc:title>
    <dc:creator>作者</dc:creator>
    <dc:language>zh-Hans</dc:language>
    <dc:publisher>出版社</dc:publisher>
    <meta name="cover" content="Cover"/>
  </metadata>
  <manifest>
    <item id="ncx" href="toc.ncx" media-type="application/x-dtbncx+xml"/>
    <item id="css" href="Styles/stylesheets.css" media-type="text/css"/>
    <item id="Cover" href="Images/cover.jpg" media-type="image/jpeg"/>
    <item id="cover-page" href="Text/cover.xhtml" media-type="application/xhtml+xml"/>
    <item id="copyright" href="Text/Copyright.xhtml" media-type="application/xhtml+xml"/>
    <item id="chapter01" href="Text/chapter01.xhtml" media-type="application/xhtml+xml"/>
  </manifest>
  <spine toc="ncx">
    <itemref idref="cover-page"/>
    <itemref idref="copyright"/>
    <itemref idref="chapter01"/>
  </spine>
</package>
```

制作时需做到：

- `unique-identifier` 指向一个实际存在的 `dc:identifier` 的 `id`；每本书使用唯一标识。
- 登记标题、作者、语言；可按出版信息补充出版者、ISBN、日期、译者等元数据。
- 封面图片的 manifest `id` 应与 `<meta name="cover" content="..."/>` 一致。
- manifest 中每个 `href` 都能在压缩包中找到，媒体类型应与文件相符。
- `spine` 覆盖所有需要按顺序阅读的 XHTML 页面。非线性附件按 EPUB 2 规范标注 `linear="no"`，不要漏登正文页。

## 5. 导航目录 NCX

样本使用 EPUB 2 的 `toc.ncx`，每个目录条目通过 `content src` 指向章节文件及标题锚点。`playOrder` 按阅读顺序递增，`navPoint` 的 `id` 保持唯一。

```xml
<navPoint id="navPoint-1" playOrder="1">
  <navLabel><text>第一章 章节标题</text></navLabel>
  <content src="Text/chapter01.xhtml#chapter-title-01"/>
</navPoint>
```

为封面、版权页及正文页分别决定是否出现在目录中；样本的 NCX 从版权页开始，封面在 spine 中但没有列入 NCX。目录项目标必须与 XHTML 内真实存在的 `id` 完全一致。

## 6. XHTML 正文与 CSS

每章单独保存为有效 XHTML 文档，统一使用 UTF-8、XHTML 命名空间和共享样式表。样本正文设置了 `xml:lang="zh-CN"`，用 `h1` 标章标题，以 `id` 作为 NCX 跳转目标。

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN"
  "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="zh-CN">
<head>
  <title>第一章 章节标题</title>
  <link href="../Styles/stylesheets.css" rel="stylesheet" type="text/css"/>
</head>
<body>
  <h1 class="bodycontent-first-title" id="chapter-title-01">第一章 章节标题</h1>
  <p class="bodycontent-text">正文段落。</p>
</body>
</html>
```

样本主要通过语义标签搭配 CSS 类控制版式，例如：

- `h1.bodycontent-first-title`：章标题；`h2`、`h3` 等类用于更低级标题。
- `p.bodycontent-text`：正文，首行缩进 2em。
- `p.bodycontent-text_yinwen`：引用或诗文段落。
- `p.zhusi`：脚注正文。
- `span.math-super`：正文中的上标脚注标记。

样本 CSS 全局段落行高约为 `1.84`，正文段落默认首行缩进；`p.zhusi` 使用约 10px 上边距、2em 首行缩进。脚注标记以 0.8em 上标显示，左右留白。设备字号可变，避免用固定像素字号锁死整页排版；重点是保留清楚的标题层级、正文段落和脚注样式。

若嵌入字体，需在 manifest 登记字体文件，并通过 `@font-face` 引用。先确认字体许可允许电子书嵌入；未获授权时使用阅读器默认字体或可合法嵌入的字体。

## 7. 脚注写法与链接规则

### 7.1 样本结构

样本没有独立的脚注 XHTML 文件，也没有将脚注放在书末。每条脚注放在所属章节正文之后，仍处于该章 XHTML 文件内。正文标记通过 `href` 跳到注释段落；注释开头的编号再通过 `href` 返回原标记。

正文标记使用方括号数字，并包在上标样式中：

```xml
<p class="bodycontent-text">
  正文中需要注释的词语
  <span class="math-super"><a href="#zhu2" id="w2">[1]</a></span>
  后续正文。
</p>
```

章节末尾对应的注释段落：

```xml
<p class="zhusi"><a href="#w2" id="zhu2">[1]</a> 注释或出处说明。</p>
```

以上用 `w2` 和 `zhu2` 演示样本的锚点命名方式；实际制作必须为每个锚点分配本书内唯一的 `id`。两处方括号编号须一致，两个方向的 `href` 必须分别命中对应 `id`。

### 7.2 编号与内容

- 按章节顺序为脚注编号；样本在每章重新从 `[1]` 编号。
- 锚点 `id` 不能重复。样本的内部锚点后缀在全书范围递增，而显示给读者的脚注编号按章重置；建议沿用这种区分，避免跨章节冲突。
- 每个正文脚注标记恰好对应一个注释段落。重复引用同一来源时，按编辑规则决定复用目标还是另建条目，并保持编号清楚。
- 注释可以写文献出处、网址、解释或译者注。译者注的文字标识（如“——译者注”）属于注释内容的一部分，不要塞进链接属性。
- 书名、期刊名等需要斜体时，用 `<i>...</i>` 等 XHTML 标签；文本中的 `&` 必须转义为 `&amp;`。
- 注释宜放在该章正文末尾、下一章标题之前；不要把脚注段落插到下一章或其他 XHTML 文件中。

### 7.3 脚注核对清单

- [ ] 正文标记为 `<a href="#注释ID" id="正文ID">[数字]</a>`。
- [ ] 注释段落以 `<a href="#正文ID" id="注释ID">[同一数字]</a>` 开始。
- [ ] 本章每个 `href` 目标都存在；正文 ID 和注释 ID 没有重复。
- [ ] 每个正文标记均有注释，每条注释均可返回正文标记。
- [ ] 每章编号连续且按约定重置；编号没有跳号、重号或显示不一致。
- [ ] XHTML 标签闭合，特殊字符已转义，所有章节脚注都留在所属章节。

## 8. 封面、版权页与其他资源

样本的封面页为单独的 `cover.xhtml`，显示 `Images/cover.jpg`；封面 XHTML、图片本身都在 OPF manifest 登记，OPF 元数据用 `meta name="cover"` 指明封面图片。版权页也单独放在 `Text/Copyright.xhtml`。制作时按出版需求提供版权页、译者/编者信息及后记，并在 spine 和 NCX 中安排合适的顺序。

封面比例、分辨率和版式应满足目标发行平台要求。CSS、图片、字体的相对路径以引用它们的 XHTML 或 CSS 文件所在目录为基准；移动文件时同步更新路径和 manifest。

## 9. 打包前检查

- ZIP 第一项是未压缩的 `mimetype`，内容准确。
- `container.xml` 指向正确 OPF；OPF XML、NCX、所有 XHTML 均格式良好。
- OPF 的 manifest 无缺失或错误路径；spine 和 NCX 顺序一致，目录锚点存在。
- 每章标题有稳定且唯一的 `id`，NCX 目标均可跳转。
- 脚注双向链接完整，正文编号与注释编号一致，章节内编号连续。
- 图片、字体、CSS 路径有效；嵌入字体具有可用许可。
- 用 EPUB 2 检查器或阅读器打开，检查封面、目录、章节跳转、脚注往返和不同屏幕宽度下的排版。

## 10. 版本选择说明

若目标是复刻本书，沿用 EPUB 2.0、XHTML 1.1 和 NCX 的结构即可。若改制为 EPUB 3，应整体迁移到 EPUB 3 的包版本、导航文档和脚注语义标记（如 `epub:type="noteref"` / `epub:type="footnote"`），并重新检查阅读器兼容性；不要只把 OPF 版本号改成 3.0 而保留不匹配的导航和语义结构。
