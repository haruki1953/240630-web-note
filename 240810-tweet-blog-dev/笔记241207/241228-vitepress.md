https://vitepress.dev/

创建项目 Tweblog，用来写 Tweblog 的官网，同时作为整个项目的主仓库
```
pnpm add -D vitepress
pnpm vitepress init
```

```
pnpm install -D sass
```

### medium-zoom 图片点击放大
https://github.com/vuejs/vitepress/issues/854
```
pnpm add -D vue medium-zoom
```

```ts
// .vitepress\theme\index.mts
import DefaultTheme from "vitepress/theme"
import { onMounted, watch, nextTick } from 'vue';
import { useRoute } from 'vitepress';
import mediumZoom from "medium-zoom"

import "./custom.css"

export default {
  ...DefaultTheme,

  setup() {
    const route = useRoute();
    const initZoom = () => {
      /**
       * Allow images to be zoomed in on click
       * https://github.com/vuejs/vitepress/issues/854
       */
      // mediumZoom('[data-zoomable]', { background: 'var(--vp-c-bg)' })
      mediumZoom('.main img', { background: 'var(--vp-c-bg)' });
    };
    onMounted(() => {
      initZoom();
    });
    watch(
      () => route.path,
      () => nextTick(() => initZoom())
    );
  },
};
```

```css
/* .vitepress\theme\custom.css */
:root {
  /* 首页标题样式 */
  --vp-home-hero-name-color: transparent;
  --vp-home-hero-name-background: linear-gradient(135deg, #7899cc 40%, #3e63dd 100%);

  /* 首页图标背景 */
  --vp-home-hero-image-background-image: radial-gradient(circle at 50% 50%,
      #5672cd 10%,
      #3e63dd 35%,
      transparent 50%);
  --vp-home-hero-image-filter: blur(150px);
}

.medium-zoom-overlay {
  z-index: 30;
}

.medium-zoom-image {
  z-index: 31;
}

:root {
  /* 调整 hero 部分图片大小 */
  .VPHero .image-src {
    width: 250px;
    /* 设置想要的宽度 */
    height: auto;
    /* 保持宽高比 */
  }
}
```

### 命令查看目录结构
```
tree /F /A
```

### 元信息
```html
  <!-- Open Graph Meta Tags -->
  <meta property="og:title" content="Tweblog">
  <meta property="og:description" content="一个自己的微博客">
  <meta property="og:image" content="https://tweblog.com/favicon.svg">
  <meta property="og:url" content="https://tweblog.com/">
  <meta property="og:type" content="website">
  <meta property="og:site_name" content="Tweblog">

  <!-- Twitter Card Meta Tags -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="Tweblog">
  <meta name="twitter:description" content="一个自己的微博客">
  <meta name="twitter:image" content="https://tweblog.com/favicon.svg">
  <meta name="twitter:site" content="@harukiO_0">
```

```ts
export default defineConfig({
  head: [
    ['link', { rel: 'icon', href: '/favicon.svg' }],
    ['meta', { property: 'og:title', content: 'Tweblog' }],
    ['meta', { property: 'og:description', content: '一个自己的微博客' }],
    ['meta', { property: 'og:image', content: 'https://tweblog.com/favicon.svg' }],
    ['meta', { property: 'og:url', content: 'https://tweblog.com/' }],
    ['meta', { property: 'og:type', content: 'website' }],
    ['meta', { property: 'og:site_name', content: 'Tweblog' }],
    ['meta', { name: 'twitter:card', content: 'summary_large_image' }],
    ['meta', { name: 'twitter:title', content: 'Tweblog' }],
    ['meta', { name: 'twitter:description', content: '一个自己的微博客' }],
    ['meta', { name: 'twitter:image', content: 'https://tweblog.com/favicon.svg' }],
    ['meta', { name: 'twitter:site', content: '@harukiO_0' }]
  ]
})
```