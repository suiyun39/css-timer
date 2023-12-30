# css-timer

纯 CSS 实现的计时器

## 原理解析

要实现这个需求，我们需要解决一个问题：当我们拿掉 JavaScript 之后，我们应该如何存储和更新计时器的状态？

通常情况下，我们不会将 HTML 和 CSS 视为真正的编程语言，但它们的一些特性组合起来却可以表现出一定的计算能力。在 Stack
Overflow 上就有人使用 HTML 和 CSS 实现了
[Rule 110 自动机](https://en.wikipedia.org/wiki/Rule_110) 以证明其图灵完备性。虽然这个实现存在一些争议，但是它的确证明了
HTML 和 CSS 能够进行一定程度的计算。

### 时间状态

对于计时功能，我们会很自然的想到使用 CSS Animation 来实现。然而，在 CSS 中存储状态则超出了日常开发的知识范畴。在这里，我们需要认识一个全新的概念：
[CSS Houdini](https://developer.mozilla.org/zh-CN/docs/Web/Guide/Houdini)。

这是一组底层 API，公开了 CSS
引擎的各个部分。其中的 [@property](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@property) 提供了自定义属性和变量的能力。
我们可以利用它来存储计时器的时间计数。以秒数为例，我们可以创建一个自定义属性：

```css
@property --second {
  syntax: "<number>";
  inherits: false;
  initial-value: 0;
}
```

接下来，我们可以利用 CSS Animation 来更新这个自定义属性。同时，利用 **执行时间**
和 [缓动函数](https://developer.mozilla.org/zh-CN/docs/Web/CSS/easing-function)
控制更新频率，从而模拟时间的流逝。

```css
@keyframes update-second {
  from {
    --second: 0;
  }

  to {
    --second: 59;
  }
}

.clock {
  animation: update-second 60s steps(60) infinite;
}
```

最后，我们使用 [CSS 计数器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_counter_styles/Using_CSS_counters) 和
**伪元素** 将其显示到页面上。

```css
.clock {
  counter-reset: second var(--second);

  &::before {
    content: counter(second, decimal-leading-zero);
  }
}
```

> 这里使用了 counter()
> 函数来优化显示，相关知识可参见 [counter()](https://developer.mozilla.org/zh-CN/docs/Web/CSS/counter)

### 控制状态

HTML 拥有一个天然的状态存储器，那就是复选框 (checkbox)
。我们可以利用它的选中与否来表示计时器的运行状态。同时，通过使用 `:checked` 伪类，
我们可以在 CSS 中捕获到这个状态，并通过兄弟选择器将这个状态传递到其他元素上。

```html
<div>
  <input type="checkbox" id="Running" />
  <label for="Running" class="btn-start"></label>
</div>
```

```css
.btn-start::before {
  content: "开始";
}

#Running:checked ~ .btn-start::before {
  content: "暂停";
}
```

同样的，我们可以利用这个方法来影响 CSS Animation，从而控制计时逻辑的运行。

```css
.clock {
  animation-play-state: paused;
}

#Running:checked ~ .clock {
  animation-play-state: running;
}
```

最后，我们利用 `:active` 伪类来捕获重置按钮的点击，并将计时器的动画状态重置。

```css
.btn-reset:active ~ .clock {
  animation: none;
}
```

## 参考与感谢

- [牛！竟然只用 CSS 就可以实现功能完整的计时器功能～](https://www.bilibili.com/video/BV1bN411u7nZ/)
