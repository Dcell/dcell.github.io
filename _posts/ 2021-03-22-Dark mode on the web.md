

title: 'Dark mode on the web'

layout: post

comments: true

---

随着App混合开发的流行，一些复杂App往往会加载一些H5相关的网页；自从iOS 13加入暗黑模式以后，android的一些厂商也加入了暗黑模式。如果在暗黑模式中加载了一个正常的H5样式，那么对视觉的冲击是比较大的，所以我们需要H5 Web能随着系统的模式自动切换。

## Toggling Themes

#### Using a Body Class

我们通过css样式切换，可以简单的切换一个主题。

<iframe height="265" style="width: 100%;" scrolling="no" title="Method 1 - Class Swapping" src="https://codepen.io/adhuham/embed/dyodgPj?height=265&theme-id=light&default-tab=html,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/adhuham/pen/dyodgPj'>Method 1 - Class Swapping</a> by Mohamed Adhuham
  (<a href='https://codepen.io/adhuham'>@adhuham</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>



#### Using Separate Stylesheets

一个优秀的前端开发工程师，在做项目开发的时候，会做一套样式管理，合理利用CSS 样式继承。我们假如你的工程已经有了一套样式 ‘Light Theme’，那么我们开发Dark Theme，只需要按照现有的样式开发一套对立的样式。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Light theme stylesheet -->
  <link href="light-theme.css" rel="stylesheet" id="theme-link">
</head>
<!-- etc. -->
</html>
```

```js
// Select the button
const btn = document.querySelector(".btn-toggle");
// Select the stylesheet <link>
const theme = document.querySelector("#theme-link");

// Listen for a click on the button
btn.addEventListener("click", function() {
  // If the current URL contains "ligh-theme.css"
  if (theme.getAttribute("href") == "light-theme.css") {
    // ... then switch it to "dark-theme.css"
    theme.href = "dark-theme.css";
  // Otherwise...
  } else {
    // ... switch it to "light-theme.css"
    theme.href = "light-theme.css";
  }
});
```

[Demo](https://codepen.io/adhuham/project/editor/AqjdGV)

#### Using Custom Properties

我们也可以通过自定义属性来做样式的切换，如果你是多人协作开发，我认为这是个不错的方案。

<iframe height="265" style="width: 100%;" scrolling="no" title="KKaPMWM" src="https://codepen.io/dcell/embed/KKaPMWM?height=265&theme-id=light&default-tab=html,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/dcell/pen/KKaPMWM'>KKaPMWM</a> by Dcell
  (<a href='https://codepen.io/dcell'>@dcell</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>



## Dark Mode at the Operating System Level

上面我们讲了些，如何通过手动的方式来切换模式（当然你可以通过URL?mode=dark），那么有没有自动跟随系统的方式呢？

CSS提供了一个参数：**`prefers-color-scheme`** [媒体特性](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Media_Queries/Using_media_queries#media_features)用于检测用户是否有将系统的主题色设置为亮色或者暗色。

```css
@media (prefers-color-scheme: dark) {
  /* Dark theme styles go here */
}
@media (prefers-color-scheme: light) {
  /* Light theme styles go here */
}
```

<iframe height="265" style="width: 100%;" scrolling="no" title="Prefers Color Scheme: Demo Use Case" src="https://codepen.io/team/css-tricks/embed/mdVrQXV?height=265&theme-id=light&default-tab=html,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/team/css-tricks/pen/mdVrQXV'>Prefers Color Scheme: Demo Use Case</a> by CSS-Tricks
  (<a href='https://codepen.io/css-tricks'>@css-tricks</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

我们也可以检测当前浏览器是否有暗黑模式

```js
const prefersDarkScheme = window.matchMedia("(prefers-color-scheme: dark)");
if (prefersDarkScheme.matches) {
  //add dark theme
} else {
  //remove dark theme
}

```

这样一看真是非常的方便，但是我们忽略了一点，这个特性只支持 Chrome 76+, Firefox 67+, Chrome Android 76+, Safari 12.5+ (13+ on iOS), and Samsung Internet Browser.

## 总结

现阶段来说，如果要实现暗黑模式还是需要手动管理CSS样式，当然在高级版本可以自动做些切换，做一些体验上的优化。