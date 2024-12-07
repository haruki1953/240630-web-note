
## 用 `dependencies.ts` 优化路径管理

### **背景问题**
在开发中，文件夹中的文件（如 `a.ts`）可能需要依赖其父文件夹中的其他文件（如 `b.ts`）。例如：
```
parent/
| b.ts
| child/
| | a.ts
```

```typescript
// child/a.ts
import { bbb } from '../b';
```

随着开发的深入，`a.ts` 可能变得庞大，并被拆分为一个独立的文件夹（如 `a/`），如下：
```
parent/
| b.ts
| child/
| | a/
| | | index.ts
```

此时，原有路径（`'../b'`）需要修改为 `'../../b'`，这在重构文件结构时容易导致路径调整出错或遗漏。

---

### **解决方案**
通过在文件夹中引入 `dependencies.ts` 文件，集中管理对父文件夹的路径依赖，从而避免路径的频繁调整。

---

### **方法实现**

在 `child` 文件夹中创建一个 `dependencies.ts` 文件，用于管理对 `parent` 文件夹中文件的导入：
```plaintext
parent/
| b.ts
| child/
| | a.ts
| | dependencies.ts
```

`dependencies.ts` 内容如下：
```typescript
// child/dependencies.ts
export * from '../b';
```

`a.ts` 通过 `dependencies.ts` 引入所需的内容：
```typescript
// child/a.ts
import { bbb } from './dependencies';
```


当需要将 `a.ts` 重构为 `a/` 文件夹，在a/文件夹中添加新的dependencies.ts即可：
```
parent/
| b.ts
| child/
| | a/
| | | index.ts
| | | dependencies.ts
| | dependencies.ts
```

```typescript
// child/a/dependencies.ts
export * from '../dependencies';
```

其他文件无需调整路径，只需继续通过 `dependencies.ts` 引入依赖：
```typescript
// child/a/index.ts
import { bbb } from './dependencies';
```
