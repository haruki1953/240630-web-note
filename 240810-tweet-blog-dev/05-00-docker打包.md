第一次尝试打包
- 前端 tweet-blog-vue3 版本 v0.0.0
- 后端 tweet-blog-hono 版本 v0.0.0

将前端打包，放在后端的 /static 目录

后端根据hono文档配置 https://hono.dev/docs/getting-started/nodejs#dockerfile
```
package.json
{
  // ...
  "scripts": {
    // ...
    "build": "tsc"
  },
  "type": "module",
}

tsconfig.json
{
  "compilerOptions": {
    // ...
    "outDir": "./dist",
  },
  "exclude": ["node_modules"],
}

pnpm install typescript --save-dev
```

注意：设置 "type": "module" 后，`__dirname`会失效，应像如下解决
```ts
import path from 'path';
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

console.log(__dirname);
```