

## 1. 什么是条件类型

条件类型是 TypeScript 中的一种高级类型，它允许根据条件来选择类型。条件类型的基本语法如下：

```typescript
T extends U ? X : Y
```

这表示：如果类型 `T` 可以赋值给类型 `U`，那么结果类型是 `X`，否则是 `Y`。

## 2. 条件类型的基本用法

### 示例

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false
```

在这个示例中，`IsString` 是一个条件类型，它检查类型 `T` 是否是 `string`。如果是，则结果类型为 `true`，否则为 `false`。

## 3. 条件类型的实际应用

### 提取联合类型中的特定类型

假设我们有一个联合类型 `ForwardSettingItem`，其中包含两种不同的对象类型：

```typescript
type ForwardSettingItem = {
    uuid: string;
    name: string;
    platform: "X";
    data: {
        'API Key': string;
        'API Key Secret': string;
        'Access Token': string;
        'Access Token Secret': string;
    };
} | {
    uuid: string;
    name: string;
    platform: "T";
    data: {
        token: string;
    };
};
```

我们希望提取 `platform` 为 `"X"` 的对象类型。可以使用条件类型来实现：

```typescript
type ExtractXPlatform<T> = T extends { platform: "X" } ? T : never;

type XPlatformItem = ExtractXPlatform<ForwardSettingItem>;

// 自己又进行了优化
// 通过传入平台所代表字段类型，来获取对应的类型
type ExtractPlatform<
  Platform extends PlatformKeyEnumValues, Item
> = Item extends { platform: Platform } ? Item : never
export type ForwardSettingPlatform<
  Platform extends PlatformKeyEnumValues
> = ExtractPlatform<Platform, ForwardSettingItem>

// 测试
type x = ForwardSettingPlatform<'X'>
```

### 解释

1. **泛型参数 `T`**：
   - `T` 是一个泛型参数，表示可以传入任何类型。

2. **条件类型 `T extends { platform: "X" } ? T : never`**：
   - 这里的条件是检查 `T` 是否可以赋值给 `{ platform: "X" }`。
   - 如果 `T` 符合 `{ platform: "X" }` 的结构（即 `T` 中包含 `platform` 属性且其值为 `"X"`），那么结果类型就是 `T`。
   - 否则，结果类型是 `never`。表示一种“不可达”的类型

### 应用示例

```typescript
type XPlatformItem = ExtractXPlatform<ForwardSettingItem>;

const item: XPlatformItem = {
    uuid: "123",
    name: "Example",
    platform: "X",
    data: {
        'API Key': "exampleKey",
        'API Key Secret': "exampleSecret",
        'Access Token': "exampleToken",
        'Access Token Secret': "exampleTokenSecret"
    }
};
```

在这个例子中，`ExtractXPlatform<ForwardSettingItem>` 会检查 `ForwardSettingItem` 类型中的每个成员：

- 对于 `{ platform: "X" }` 的成员，结果类型是该成员本身。
- 对于 `{ platform: "T" }` 的成员，结果类型是 `never`。

最终，`XPlatformItem` 类型会是：

```typescript
{
    uuid: string;
    name: string;
    platform: "X";
    data: {
        'API Key': string;
        'API Key Secret': string;
        'Access Token': string;
        'Access Token Secret': string;
    };
}
```

## 4. 条件类型的其他用法

### 分发条件类型

条件类型在联合类型上会自动分发。例如：

```typescript
type ToArray<T> = T extends any ? T[] : never;

type StrArrOrNumArr = ToArray<string | number>; // string[] | number[]
```

在这个例子中，`ToArray<string | number>` 会被分发为 `ToArray<string> | ToArray<number>`，结果是 `string[] | number[]`。

### 内置条件类型

TypeScript 提供了一些内置的条件类型，例如：

- `Exclude<T, U>`：从 `T` 中排除可以赋值给 `U` 的类型。
- `Extract<T, U>`：从 `T` 中提取可以赋值给 `U` 的类型。
- `NonNullable<T>`：从 `T` 中排除 `null` 和 `undefined`。
- `ReturnType<T>`：获取函数类型 `T` 的返回类型。
- `InstanceType<T>`：获取构造函数类型 `T` 的实例类型。

### 示例

```typescript
type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T1 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T2 = NonNullable<string | number | undefined>;  // string | number
type T3 = ReturnType<() => string>;  // string
type T4 = InstanceType<typeof Date>;  // Date
```

## 5. 总结

条件类型是 TypeScript 中非常强大且灵活的工具，允许我们根据类型的特定条件来选择或过滤类型。通过条件类型，我们可以更精确地控制类型系统，处理复杂的类型逻辑。

希望这篇笔记对你理解 TypeScript 条件类型有所帮助！如果你有任何问题，欢迎随时提问。😊
