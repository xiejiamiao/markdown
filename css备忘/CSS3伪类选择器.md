## 结构伪类选择器
``` css
li:first-child{ /*选择第一个子节点*/
    color: pink;
}

li:last-child{ /*最后一个子节点*/
    color: pink;
}

li:nth-child(4){ /*第N个子节点*/
    color: pink;
}

li:nth-child(even){ /*所有偶数子节点*/
    color:pink; 
}

li:nth-child(odd){ /*所有基数子节点*/
    color: pink;
}

li:nth-child(2n){
    /*同样选择所有偶数子节点*/
    /*2n+1 -> 选择所有基数子节点*/
    /*3n -> 选择 0、3、6、9...等子节点*/
    /*可按照数列的方式进行计算*/
}

li:nth-last-child(even){
    /*从后往前数的偶数*/
}
```


## 目标伪类选择器
``` css
:target{ /*使用锚点的时候触发这个选择器,即URL右面会跟上#id*/
    color: red;
}
```

