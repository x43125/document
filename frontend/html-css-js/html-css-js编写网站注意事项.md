# html-css-js 注意事项

## 1 TDK

```html
<!-- TDK -->
<!-- 标题 -->
<title>品优购商城-正品低价、品质保障、配送及时、轻松购物！</title>
<!-- 描述 -->
<meta name="description" content="品优购商城-专业的综合网上购物商城，为您提供正品低价的购物选择、优质便捷的服务体验。
                                  商品来自全球数十万品牌商家，囊括家电、手机、电脑、服装、居家、母婴、美妆、个护、食品、生鲜等丰富品类，满足各种购物需求。" />
<!-- 关键字 -->
<meta name="Keywords" content="网上购物,网上商城,家电,手机,电脑,服装,居家,母婴,美妆,个护,食品,生鲜,品优购商城" />
```

通过给网页设置TDK三项来提升网站在搜索引擎中的可被搜索性，使得网站更容易被检索到。

TDK分别指：Title, Description, Keywords

## 2 favicon.ico

通过引入``favicon.ico`来给网页标题添加图标

```html
<!-- 引入网站标题图标 favicon图标 -->
<link rel="shortcut icon" href="favicon.ico">
```

## 3 base.css

每一个网站都应该有一些基础配置，用来使得网站风格统一。

比如京东的大致如下（有部分改动）

```css
/* 把所有内外边距先清除 */
* {
    margin: 0;
    padding: 0;
    /* css3盒子模型 */
    box-sizing: border-box;
}

/* em i 斜体文字不倾斜 */
em,
i {
    font-style: normal;
}

/* 去掉li的圆点 */
li {
    list-style: none;
}

img {
    /* 照顾低版本浏览器，如果图片外面包含了链接会有边框的问题 */
    border: 0;
    /* 取消图片底部有空隙的问题 */
    vertical-align: middle;
}

button {
    /* 当鼠标经过button 按钮的时候，鼠标变成 手 */
    cursor: pointer;
}

button,
input {
    /* 边框手动去掉 */
    border: 0;
    outline: none;
}

a {
    color: #666;
    /* 取消下划线 */
    text-decoration: none;
}

a:hover {
    color: #c81623;
}

body {
    /* 抗锯齿性，文字放大更加清晰 css3写法 */
    -webkit-font-smoothing: antialiased;
    background-color: #fff;
    font: 12px/1.5 Microsoft YaHei, Heiti SC, tahoma, arial, Hiragino Sans GB, "\5B8B\4F53", sans-serif;
    color: #666
}

/* 清除浮动 */
.clearfix::after {
    visibility: hidden;
    clear: both;
    display: block;
    content: ".";
    height: 0;
}

.clearfix {
    *zoom: 1;
}
```

## 4 common.css

通过将各网页公共部分样式提取出来的方式以供复用，减少代码编写，一般比如`header`部分`footer`部分等。

**所以一般每个网页最少会有3个css：base.css、common.css 以及自己的css文件**

## 5 logo优化：SEO优化

- 在logo盒子内部放一个h1标签把logo包起来；
- 给logo放一个首页链接；
- 给logo一些文字，比如网站名称，但不要显示出来；
- 给logo链接一个title属性，也可以用网站名称。

**例如：**

```html
<!-- logo模块 -->
<div class="logo">
    <h1>
        <a href="index.html" title="品优购商城">品优购商城</a>
    </h1>
</div>
```

```css
.logo {
    position: absolute;
    top: 25px;
    width: 171px;
    height: 61px;
}

.logo a {
    display: block;
    width: 171px;
    height: 61px;
    background: url(../images/logo.png) no-repeat;
    /* 有两种不显示文字的做法如下： */
    /* jd做法 */
    /* font-size: 0; */
    /* tb做法 */
    text-indent: -9999px;
    overflow: hidden;
}
```

## 6 引入字体图标

> 以icomoon的字体图标举例

在用到的css里面首先添加上以下声明来引入相应的字体图标资源

```css
@font-face {
    font-family: 'icomoon';
    /* 要注意:此处的url括号里的是对应文件的路径，不要太死板的照抄！！！只需要修改地址即可 */
    src: url('../fonts/icomoon.eot?tomleg');
    src: url('../fonts/icomoon.eot?tomleg#iefix') format('embedded-opentype'),
        url('../fonts/icomoon.ttf?tomleg') format('truetype'),
        url('../fonts/icomoon.woff?tomleg') format('woff'),
        url('../fonts/icomoon.svg?tomleg#icomoon') format('svg');
    font-weight: normal;
    font-style: normal;
    font-display: block;
}
```

然后在用到字体图标的内容处填写相应的字体图标值或对应的码值。如：

```css
.arrow-icon::after {
    content: '\e91e';
    /* content: ''; */
    /* 哪里用到哪里需要引入 */
    font-family: 'icomoon';
    margin-left: 6px;
}
```

- content: 里则是字体图标的对应编码值，或者可以像注释里那样直接将图标复制进来`/* content: ''; */`
- font-family: 然后再指定一下font-family即可

## 7 清除浮动

一般对于一些需要清除浮动的情况，我们会给需要清除浮动的对象添加一个以下样式的类名。为了方便可以将以下样式放到 `base.css`里（如上面京东就是这么做的）。

```css
/* 清除浮动 */
.clearfix::after {
    visibility: hidden;
    clear: both;
    display: block;
    content: ".";
    height: 0;
}

.clearfix {
    *zoom: 1;
}
```

```html
<div class="w sk_container">
    <div class="sk_hd"><img src="upload/bg_03.png" alt=""></div>
    <div class="sk_bd">
        <ul class="clearfix">
            <li><img src="upload/list.jpg" alt=""></li>
            <li><img src="upload/list.jpg" alt=""></li>
            <li><img src="upload/list.jpg" alt=""></li>
            <li><img src="upload/list.jpg" alt=""></li>
            <li><img src="upload/list.jpg" alt=""></li>
            <li><img src="upload/list.jpg" alt=""></li>
            <li><img src="upload/list.jpg" alt=""></li>
            <li><img src="upload/list.jpg" alt=""></li>
            <li><img src="upload/list.jpg" alt=""></li>
        </ul>
    </div>
</div>
```

如上在`ul`里就加了clearfix来控制盒子浮动问题