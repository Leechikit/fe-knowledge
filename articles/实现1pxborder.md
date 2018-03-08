# 实现 1px border

## 使用box-shadow模拟边框

利用css 对阴影处理的方式实现0.5px的效果。

```
.box-shadow-1px {
	box-shadow: inset 0px -1px 1px -1px #c8c7cc;
}
```

## viewport + rem 实现

根据devicePixelRatio来输出viewport，边框直接使用1px。

在devicePixelRatio = 2 时，输出viewport：
```
<meta name="viewport" content="initial-scale=0.5, maximum-scale=0.5, minimum-scale=0.5, user-scalable=no"/>
```

在devicePixelRatio = 3 时，输出viewport：
```
<meta name="viewport" content="initial-scale=0.3333, maximum-scale=0.3333, minimum-scale=0.3333, user-scalable=no"/>
```

## 伪类 + transform 实现

利用 :before 或者 :after 重做 border ，并 transform 的 scale 缩小一半，原先的元素相对定位，新做的 border 绝对定位。

```
.scale-1px{
	position: relative;
	border:none;
}
.scale-1px:after{
	content: '';
	position: absolute;
	bottom: 0;
	background: #000;
	width: 100%;
	height: 1px;
	transform: scaleY(0.5);
}
```