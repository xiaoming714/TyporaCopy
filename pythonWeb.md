# 前端

## html 

```html
head
<meta charset="utf-8">文件编码
title 浏览器标签名

body
hn 块级标签，标题系列，n为一个数字，
div 文字块级标签，其中内容占一整行
span 文字行内标签/内联标签，占其中内容本身大小
<a href="要跳转的网站">文本</a> 超链接，跳转自己网站可以将域名省略
<image src="地址"/> 图片（自闭和），自己的图片放到static目录中，可以使用a标签包裹image实现点击图片跳转
a表中写target=_blank另起tab打开页面
ul li列表中的一列li ul 无序列表
ol li li ol 有序列表
table thead tr th表头中的一行中的一个元素th tr trthead table 表体用tbody
input系列，输入框
select option下拉框中的其中一个选项option select select后加mutiple开启多选
textarea多行文本
form表单
```

```css
css应用方式：在标签中使用，在head中定义style（便于复用），写在文件中
选择器：.c1叫类选择器，#c2叫id选择器，li叫标签选择器（li可以是任何标签），属性选择器（input中的type等属性的样式），后代选择器
height 高度
width 宽度（支持百分比），高度和宽度对行内标签默认无效，对块级标签默认空着也不能用
display=inline-block，中和行内标签和块级标签的特点
也可以通过inline和block来对标签进行的属性进行修改
color 字体属性
font-size 字体大小
font--weight 字体加粗
font-family 字体
border 边框
text-align 字体水平位置
line-height 字体竖直位置（如果想要对其可以把整个块都占上）
```

