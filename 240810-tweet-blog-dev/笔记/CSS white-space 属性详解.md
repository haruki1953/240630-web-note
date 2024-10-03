
## CSS `white-space` 属性详解

`white-space` 属性用于控制元素内空白字符的处理方式。根据不同的配置选项，可以实现不同的文本显示效果。以下是各个配置选项的详细说明：

### 1. `normal`
- **效果**: 连续的空白符会被合并。源码中的换行符会被当作空白符来处理。文本会根据需要换行。
- **示例**:
  ```css
  .example {
    white-space: normal;
  }
  ```

### 2. `nowrap`
- **效果**: 连续的空白符会被合并，但文本不会换行。
- **示例**:
  ```css
  .example {
    white-space: nowrap;
  }
  ```

### 3. `pre`
- **效果**: 连续的空白符会被保留。文本只会在换行符或 `<br>` 元素处换行。
- **示例**:
  ```css
  .example {
    white-space: pre;
  }
  ```

### 4. `pre-wrap`
- **效果**: 连续的空白符会被保留。文本会在换行符、 `<br>` 元素处以及根据需要换行。
- **示例**:
  ```css
  .example {
    white-space: pre-wrap;
  }
  ```

### 5. `pre-line`
- **效果**: 连续的空白符会被合并。文本会在换行符、 `<br>` 元素处以及根据需要换行。
- **示例**:
  ```css
  .example {
    white-space: pre-line;
  }
  ```

### 6. `break-spaces`
- **效果**: 类似于 `pre-wrap`，但任何保留的空白序列总是占用空间，包括行末的空白符。每个保留的空白字符后都可以换行。
- **示例**:
  ```css
  .example {
    white-space: break-spaces;
  }
  ```

### 总结
`white-space` 属性提供了多种选项来控制文本的显示方式。根据具体需求选择合适的配置，可以实现不同的文本布局效果。
