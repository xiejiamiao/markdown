### outline

元素聚焦之后的边框线，一般使用 outline: 0; 取消

### textarea禁止拖拽

```css
textarea{
    resize: none;
}
```

### vertical-algin

不影响块级元素中的内容对齐，只针对行内元素或者行内块元素，特别是行内块元素，**通常用来控制图片和表单文字的对齐**

```html
<style>
    img{
        vertical-algin: middle;
    }
</style>
<div>
    <img src='a.jpg'/>中线对齐
    <hr/>
    用户留言：<textarea></textarea>
</div>
```

