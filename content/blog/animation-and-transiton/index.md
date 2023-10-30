---
title: 动画和过渡!
date: "2023-10-31"
---

# 动画和过渡

## 1.动画相关

#### 1-1.动画会在什么时候生效

1. 页面加载时：可以设置动画在页面加载完成后立即开始。例如，可以将动画样式应用于某个元素，并在页面加载时使其显示动画效果。

   ```
   .element {
     animation: myAnimation 1s;
   }
   ```

2. 事件触发时：动画可以在特定事件触发时开始，如鼠标悬停、点击或滚动等。可以使用伪类选择器（如`:hover`）或JavaScript事件监听器来触发动画。

   ```css
   .element:hover {
     animation: myAnimation 1s;
   }
   ```

3. CSS类添加/移除时：可以通过添加或移除CSS类来触发动画效果。当将包含动画样式的类应用于某个元素时，动画将开始。

   ```css
   .element.myClass {
     animation: myAnimation 1s;
   }
   ```

4. 通过JavaScript控制：可以使用JavaScript来动态添加动画样式或操作动画的属性，以在特定情况下触发动画。例如，可以使用JavaScript修改元素的样式属性或添加/移除CSS类来启动动画。

   ```javascript
   var element = document.getElementById('myElement');
   element.style.animation = 'myAnimation 1s';
   ```

   > 总之，CSS动画可以在页面加载、事件触发、CSS类添加/移除或通过JavaScript控制时生效。
