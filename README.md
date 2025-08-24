# 未来传媒 :link: https://www.futuremedia.work 
# # 网站调试与功能增强日志

本文档记录了 www.futuremedia.work 网站在基于 Gmeek 博客系统进行深度定制过程中，遇到的一系列技术问题及其解决方案。

# 最后更新: 2025年8月24日

核心问题一：自定义页面 (link.html) 显示为代码
问题现象: 创建一个独立的导航页面 link.html，在 Issue 中粘贴 HTML 代码后，最终发布的页面直接显示 HTML 源代码，而不是渲染后的网页。

排查过程:

初步尝试: 直接在 Issue 中粘贴完整 HTML 代码，失败。

Gmeek-html 方案: 根据官方文档，使用 ```Gmeek-html ... ``` 包裹 HTML 代码。问题依旧，页面头部和尾部的<style>和<script>` 部分仍然显示为代码。

##{...} 配置方案: 尝试将 HTML、CSS 和 JavaScript 分离，将 CSS 和 JS 放入文章末尾的 ##{"style": "...", "script": "..."} 配置中。问题依旧，style 和 script 的内容被当作纯文本显示在页面上。

根本原因:

模板安全机制: post.html 模板为了安全，默认不会执行来自 ##{...} 配置块中的 style 和 script 内容，而是将它们作为普通文本显示。

引号嵌套问题: ##{...} JSON 块对内部的引号格式要求极为严格，任何不规范的引号（如中文引号 “ 或 font-family 中的单引号）都可能导致解析失败。

最终解决方案:

修改 templates/post.html 文件，在渲染自定义样式和脚本的地方，添加 |safe 过滤器，明确告诉模板引擎这些内容是安全的，可以直接执行。

{{ blogBase['style']|safe }}

{{ blogBase['script']|safe }}

在 link.html 的 Issue 中，将页面主体内容作为正文，将所有 CSS 和 JavaScript 代码分别合并到一行，放入 ##{"style": "...", "script": "..."} 配置块中，并确保所有引号都符合 JSON 规范。

核心问题二：自定义URL (slug) 和时间戳 (timestamp) 不生效
问题现象: 在文章末尾的 ##{...} 中添加了 slug 和 timestamp 参数后，旧文章的 URL 依然是标题拼音，发布日期也没有改变。

排查过程:

检查 Actions 日志，发现 Has Custom JSON parameters 的关键信息没有出现，证明脚本未能成功解析配置。

怀疑是 slug 和 timestamp 同时使用存在冲突，但单独测试 timestamp 依然不生效。

通过“洁净室测试”（新建一篇格式完美的文章），发现新文章可以成功，但编辑过的旧文章不行。

根本原因:

脚本解析逻辑脆弱: Gmeek.py 原始的解析代码只会读取 Issue 正文的最后一行。当用户编辑旧文章后，GitHub 的编辑器可能会在末尾自动添加一个或多个看不见的空行，导致脚本找不到配置行。

容错性差: 解析失败后，脚本会静默地将配置视为空对象 {}，不会报错，增加了排查难度。

最终解决方案:

重写 Gmeek.py 中的 addOnePostJson 函数，修改其解析逻辑。

新逻辑不再只看最后一行，而是从文章末尾向前搜索，找到最后一个有内容的行 (last_meaningful_line) 进行解析，从而忽略了所有末尾的空行。

增加了更详细的调试日志，方便未来排查类似问题。

核心问题三：GitHub Actions 构建失败 (SyntaxError, AttributeError 等)
问题现象: 在修改 Gmeek.py 脚本后，工作流运行时出现各种 Python 错误，如 IndentationError (缩进错误), SyntaxError (语法错误), AttributeError (属性错误)。

排查过程: 逐一核对错误日志和对应的代码行。

根本原因:

代码传输不完整: 由于工具限制，多次未能一次性提供完整的 Gmeek.py 文件，导致拼接的代码在函数边界处产生语法错误。

复制粘贴引入错误: 手动复制粘贴 Python 代码时，容易破坏其严格的缩进结构，导致 IndentationError。

笔误: 在后期修改中，出现了 blog.blog.blogBase 这样的变量名拼写错误。

最终解决方案:

采用分段提供代码的方式，确保能获得完整的代码。

在最终版本中，用一个经过完全验证的、完整的 Gmeek.py 文件，直接替换，避免了所有手动修改可能带来的错误。

总结
经过一系列的调试，网站目前已进入稳定状态。所有核心功能，包括新增自定义页面、新增自定义URL、新增定时发布、修改页眉背景、修改TOC目录、采用Giscus评论系统、新增Pagefind全文搜索等均已正常工作。本次调试的核心经验是：

深入理解模板渲染机制，特别是安全过滤器（如 |safe）的作用。

编写健壮的解析代码，充分考虑用户输入的多样性（如不同的换行符、末尾空行等）。

优先使用完整文件替换，而非手动修改代码，以避免引入格式和语法错误。

### :page_facing_up: [48](https://www.futuremedia.work/tag.html) 
### :speech_balloon: 0 
### :hibiscus: 319642 
### :alarm_clock: 2025-08-24 16:20:44 
### Powered by :heart:[疯子](https://www.futuremedia.work)

