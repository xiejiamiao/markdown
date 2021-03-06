### CSS3 动画属性

| **属性**                  | **描述**                                               | **css** |
| ------------------------- | ------------------------------------------------------ | ------- |
| @keyframes                | 规定动画                                               | 3       |
| animation                 | 所有动画属性的简写属性，除了 animation-play-state 属性 | 3       |
| animation-name            | 规定 @keyframes 动画的名称                             | 3       |
| animation-duration        | 规定动画完成一个周期所花费的秒或毫秒，默认是0          | 3       |
| animation-timing-function | 规定动画的速度曲线，默认是 ease                        | 3       |
| animation-delay           | 规定动画何时开始，默认是0                              | 3       |
| animation-iteration-count | 规定动画被播放的次数，默认是1                          | 3       |
| animation-direction       | 规定动画是否在下一周期逆向播放，默认是normal           | 3       |
| animation-play-state      | 规定动画是否正在运行或暂停，默认是 running             | 3       |
| animation-fill-mode       | 规定对象动画时间之外的状态                             | 3       |

```css
div{
    animation:动画名称 动画时间 运动曲线 何时开始 播放次数 是否反方向;
}
@keyframes 动画名称 { /*定义名称*/
    from{ /*from 和 to也可以用百分比*/
        transform: translateX(0);
    }
    to{
        transform: translateX(600px);
    }
}
```

