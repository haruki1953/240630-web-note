
### 事件监听器中的 `passive` 选项

在前端开发中，事件监听器（Event Listener）是处理用户交互的关键工具。`passive` 选项是事件监听器的一个属性，用于优化滚动性能。本文将详细介绍 `passive` 选项的作用及其使用场景。

#### 什么是 `passive` 选项？

`passive` 选项是事件监听器的一个配置属性。当你设置 `passive: true` 时，浏览器会知道这个事件监听器不会调用 `preventDefault()` 方法，从而可以优化滚动性能。相反，如果设置 `passive: false`，浏览器会认为可能会调用 `preventDefault()`，因此不会进行相应的优化。

#### 为什么使用 `passive` 选项？

在处理滚动事件（如 `wheel`、`touchstart` 和 `touchmove`）时，`passive` 选项尤为重要。默认情况下，浏览器在处理这些事件时会等待事件监听器执行完毕，以确定是否调用 `preventDefault()` 来阻止默认滚动行为。这种等待会导致滚动性能下降，特别是在移动设备上。

通过设置 `passive: true`，你可以告诉浏览器不需要等待，从而提高滚动性能。

#### 如何使用 `passive` 选项？

以下是一个使用 `passive` 选项的示例：

```javascript
overlayBox?.addEventListener('wheel', handleWheelScroll, {
  passive: true // 如果 handleWheelScroll 不调用 preventDefault()，则设置为 true
});
```

在这个示例中，如果 `handleWheelScroll` 方法没有调用 `preventDefault()`，你可以将 `passive` 设置为 `true`，以提高滚动性能。如果 `handleWheelScroll` 调用了 `preventDefault()`，则需要将 `passive` 设置为 `false`。

#### 总结

`passive` 选项是一个简单但强大的工具，可以显著提高滚动事件的性能。在编写事件监听器时，合理使用 `passive` 选项不仅可以提升用户体验，还能优化应用的整体性能。
