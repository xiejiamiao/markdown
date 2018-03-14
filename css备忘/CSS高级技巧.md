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
    textarea{
        vertical-algin: middle;
    }
</style>
<div>
    <img src='a.jpg'/>中线对齐
    <hr/>
    用户留言：<textarea></textarea>
</div>
```

### 溢出的文字处理

#### word-break：自动换行 

normal:使用浏览器默认的换行规则

break-all:允许在单词内换行(允许单词拆开显示)

keep-all:只能在半角空格或连字符处换行

> 主要处理英文单词

#### white-space

设置或检索对象内文本显示方式，通常用来强制一行显示内容

normal:默认处理方式

nowrap:强制在同一行内显示所有文本，知道文本结束或遭遇到br对象才换行

#### text-overflow：文字溢出

设置或检索是否使用一个省略标记(...)