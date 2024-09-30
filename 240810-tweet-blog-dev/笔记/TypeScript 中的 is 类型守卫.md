### TypeScript 中的 `is` 类型守卫

在 TypeScript 中，类型守卫（Type Guard）是一种用于在运行时检查变量类型的技术。通过类型守卫，TypeScript 编译器可以在特定的上下文中推断变量的具体类型，从而提高类型准确性和代码可靠性。`is` 类型守卫是一种自定义类型守卫，使用类型谓词（type predicate）来实现。

#### 什么是 `is` 类型守卫？

`is` 类型守卫是一种自定义类型守卫，它通过返回一个布尔值来告诉 TypeScript 编译器一个变量是否属于特定类型。其基本语法如下：

```typescript
function isType(value: any): value is SpecificType {
  return condition;
}
```

其中，`value is SpecificType` 是类型谓词，表示如果函数返回 `true`，则 `value` 的类型被缩小为 `SpecificType`。

#### 示例：使用 `is` 类型守卫

假设我们有一个包含不同形状的数组，我们希望过滤出所有的圆形对象。我们可以定义一个 `isCircle` 类型守卫来实现这一点：

```typescript
interface Circle {
  type: "CIRCLE";
  radius: number;
}

interface Square {
  type: "SQUARE";
  size: number;
}

type Shape = Circle | Square;

const shapes: Shape[] = [
  { type: "CIRCLE", radius: 10 },
  { type: "SQUARE", size: 5 },
  { type: "CIRCLE", radius: 20 },
];

function isCircle(shape: Shape): shape is Circle {
  return shape.type === "CIRCLE";
}

const circles = shapes.filter(isCircle);
console.log(circles); // 输出: [{ type: "CIRCLE", radius: 10 }, { type: "CIRCLE", radius: 20 }]
```

在这个示例中，`isCircle` 函数检查 `shape` 的 `type` 属性是否为 `"CIRCLE"`，如果是，则返回 `true`，并且 `shape` 的类型被缩小为 `Circle`。然后，我们可以使用 `filter` 方法来过滤出所有的圆形对象。

#### 在 `filter` 中使用 `is` 类型守卫

在 `filter` 方法中使用 `is` 类型守卫可以确保过滤后的数组元素具有正确的类型。以下是一个更详细的示例：

```typescript
interface Image {
  id: number;
  url: string;
}

interface ImageStoreData {
  posts: Image[];
  _count: {
    posts: number;
  };
}

const imageList = {
  value: [
    { id: 1, url: "image1.jpg" },
    { id: 2, url: "image2.jpg" },
    { id: 3, url: "image3.jpg" },
  ],
};

const resImage = { id: 2, url: "image2.jpg" };

function isNumber(value: number | null): value is number {
  return value !== null;
}

const findImageIndexList = imageList.value
  .map((image, index) => (image.id === resImage.id ? index : null))
  .filter(isNumber);

console.log(findImageIndexList); // 输出: [1]
```

在这个示例中，我们定义了一个 `isNumber` 类型守卫来检查值是否为 `number`。在 `filter` 方法中使用 `isNumber` 类型守卫，可以确保过滤后的数组元素类型为 `number`。

### 总结

使用 `is` 类型守卫可以帮助我们在 TypeScript 中更精确地处理类型检查和类型缩小。通过自定义类型守卫，我们可以在运行时检查变量类型，并在特定的上下文中推断变量的具体类型，从而提高代码的类型安全性和可维护性。

希望这篇笔记对你有所帮助！如果你有其他问题或需要进一步的解释，请随时告诉我！ 😊

¹: [LogRocket Blog](https://blog.logrocket.com/how-to-use-type-guards-typescript/)
²: [Spencer Miskoviak](https://www.skovy.dev/blog/typescript-filter-array-with-type-guard)
³: [GeeksforGeeks](https://www.geeksforgeeks.org/how-to-use-type-guards-in-typescript/)
⁴: [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/type-system/typeguard)
⁵: [DEV Community](https://zirkelc.dev/posts/typescript-filter-values-type-safe)

源: 与 Copilot 的对话， 2024/9/30
(1) How to use type guards in TypeScript - LogRocket Blog. https://blog.logrocket.com/how-to-use-type-guards-typescript/.
(2) Filtering arrays with TypeScript type guards | Spencer Miskoviak. https://www.skovy.dev/blog/typescript-filter-array-with-type-guard.
(3) How to use Type Guards in TypeScript - GeeksforGeeks. https://www.geeksforgeeks.org/how-to-use-type-guards-in-typescript/.
(4) Type-Safe Filter Function In TypeScript - DEV Community. https://zirkelc.dev/posts/typescript-filter-values-type-safe.
(5) Type Guard | TypeScript Deep Dive - GitBook. https://basarat.gitbook.io/typescript/type-system/typeguard.