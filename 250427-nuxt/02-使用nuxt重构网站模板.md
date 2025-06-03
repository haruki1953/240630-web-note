模板网站
- https://html5up.net/
- https://startbootstrap.com/
- https://templatemo.com/
- https://htmlrev.com/

已重构
- https://github.com/haruki1953/anchor-bootstrap-ui-kit-nuxt
- https://github.com/haruki1953/startbootstrap-agency-nuxt
- https://github.com/haruki1953/startbootstrap-creative-nuxt
- https://github.com/haruki1953/startbootstrap-grayscale-nuxt


## 使用nuxt重构网站模板的操作

### 项目初始化
参考 https://github.com/haruki1953/nuxt-starter-for-html-template
```
重点，使用 nuxt-purgecss 缩减 css。尤其是使用bootsharp的网站模板，不缩减css会导致打包后的index.html过大
```

### 复制资源
复制模板网站的css
```
如 startbootstrap-agency
复制其 dist\css\styles.css
到nuxt项目的 src\assets\css\main.css
（具体路径要看自己的 nuxt.config.ts）
```

复制模板网站的图片、图标
```
如 startbootstrap-agency
复制其 dist\assets\img 文件夹
到nuxt项目的 src\assets\img
图标复制至 public
```

### 调整nuxt.config.ts
仿照模板网站设置头信息

### 整理html
将模板网站的html拆分为一个个组件，模板网站的结构一般是这样的：
```html
<!-- Navigation-->
<nav class="navbar navbar-expand-lg navbar-dark fixed-top" id="mainNav">
</nav>
<!-- Masthead-->
<header class="masthead">
</header>
<!-- Services-->
<section class="page-section" id="services">
</section>
<!-- Portfolio Grid-->
<section class="page-section bg-light" id="portfolio">
</section>
<!-- About-->
<section class="page-section" id="about">
</section>
<!-- Team-->
<section class="page-section bg-light" id="team">
</section>
<!-- Clients-->
<div class="py-5">
</div>
<!-- Contact-->
<section class="page-section" id="contact">
</section>
<!-- Footer-->
<footer class="footer py-4">
</footer>
```

### pnpm generate



