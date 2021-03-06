### 2D变形(CSS3) transform

transform可以实现元素的位移、旋转、倾斜、缩放、甚至支持矩阵方式，配合过渡和动画，可以取代大量之前只能靠flash才能实现的效果

#### translate (移动)

```css
div{
    width: 100px;
    height: 100px;
    background-color: #ccc;
    transition: all 0.5s;
}
div:active{
    transform: translate(50px, 50px); /*transform: translate(x, y);*/
}
```

让定位的盒子居中对齐的完美解决方法

```css
div{
    width: 200px;
    height: 200px;
    background-color: #ccc;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%); /*translate后面跟百分比是以自己的宽高来进行计算*/
}
```



#### scale(缩放)

```css
div{
    width: 100px;
    height: 100px;
    background-color: #ccc;
    transition: all 0.5s;
}
div:hover{
    transform: scale(1.2, 1.5); /*x=水平缩放 y=垂直缩放*/
}
```



#### rotate (旋转)

```css
img{
    transition: all 0.5s;
    border-radius: 50%;
    border: 10px solid #ccc;
}
img:hover{
    transform: rotate(90deg); /*deg=度数*/
}
```

指定变形中心点

```css
img{
    transition: all 0.5s;
    border-radius: 50%;
    border: 10px solid #ccc;
    transition: all 0.5s;
    transform-origin: top left; /*设置变形中心点*/
}
img:hover{
    transform: rotate(180deg);
}
```



#### skew (倾斜)

```css
div{
    transform: skew(30deg, 0deg);
}
```



#### 3D变形

##### rotateXYZ

```css
img{
    transition: all 0.5s;
}
img:hover{
    transform: rotateX(180deg); /*X轴旋转*/
    transform: rotateY(180deg); /*Y轴旋转*/
    transform: rotateZ(180deg); /*Z轴旋转*/
}
```

#### 透视

```css
body{
    perspective: 500px; /*视距*/
}
```

