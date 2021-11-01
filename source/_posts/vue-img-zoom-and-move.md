---
title: 在vue中鼠标拖动图片，滚轮缩放图片
date: 2021-11-01 14:10:41
tags: ['vue']
cover: https://s3.bmp.ovh/imgs/2021/10/52fd491754edacb4.png
---

### 需求

在显示区域缩放，拖动一张图片。

###  代码

+ 图片

~~~html
<div class="mapbox" id="mapbox" :style="'width:'+width+';height:'+height">
    <table class="imgbox" id="imgbox" @mousedown="holdDown" @mouseup="holdUp" :style="'top: '+imgtop+'px;left: '+imgleft+'px;'" border="0" cellpadding="0" cellspacing="0" style="margin:0 auto;">
        <tr>
            <td>
                <img id="backgroundImg" draggable="false" :style="'transform:scale(' + imgScale + ',' + imgScale + ');'" :src="mapUrl" />
            </td>
        </tr>
    </table>
</div>
~~~

+ 参数

~~~javascript
props:{
    img: {
        type: String,
        default: "../../../assets/images/map.jpg",
    },
    //盒子的宽
    width: {
        type: String,
        default: "100%",
    },
    //盒子的高
    height: {
        type: String,
        default: "100%",
    },
},
data() {
    return {
        imgtop: -2800, //图片距离上边的距离。初始化距离
        imgleft: -3400, //图片距离左边的距离。初始化距离
        imgheight: 100, //图片高度百分比
        DownUp: false, //用来判断鼠标是否长按
        actualDistanceWidth: 6740, // 图片实际宽度
        actualDistanceHeight: 4920, // 图片实际高度
        imgScale:0.5 // 初始缩放比
    };
},
~~~



+ 方法

~~~javascript
//鼠标按下
holdDown() {
    this.DownUp = true;
},
//鼠标松开
holdUp() {
    this.DownUp = false;
},
//ev：鼠标对象，id：盒子的id 判断鼠标是否在盒子内
inBoxIsoutbox(id, ev = event || window.event) {
    let map = document.getElementById(id);
    // 这里的 400，50 是 map.offsetLeft 和map.offsetTop
    // 如果有用还是建议使用map.offsetLeft 和map.offsetTop
    if (
        this.mousePosition(ev).x > 400 + map.offsetWidth ||
        this.mousePosition(ev).x < 400 ||
        this.mousePosition(ev).y > 50 + map.offsetHeight ||
        this.mousePosition(ev).y < 50
    ) {
        return false;
    } else {
        return true;
    }
},
//兼容后，返回X，Y
mousePosition(ev) {
    if (ev.pageX || ev.pageY) {
        return { x: ev.pageX, y: ev.pageY };
    }
    return {
        x:
        ev.clientX +
        document.body.scrollLeft -
        document.body.clientLeft,
        y:
        ev.clientY +
        document.body.scrollTop -
        document.body.clientTop,
    };
},
// 鼠标移动触发该方法
mouseMove(ev) {
    ev = ev || window.event;
    if (this.inBoxIsoutbox("mapbox", ev)) {
        // 鼠标在盒子内
        this.runWheel(true);
    } else {
        // 鼠标在盒子外
        this.runWheel(false);
        this.holdUp();
    }
    if (this.DownUp) {
        // 鼠标长按时改变图片位置
        this.imgtop = this.imgtop + ev.movementY;
        // 这里做限制，使图片移动时不会超过mapbox的范围
        if (this.imgtop > (this.actualDistanceHeight * (this.imgScale - 1))/2) {
            this.imgtop = (this.actualDistanceHeight * (this.imgScale - 1))/2
        }
        // 此处的参数是mapbox的实际宽度
        if (this.imgtop < ((this.actualDistanceHeight * (this.imgScale - 1))/2) - (this.actualDistanceHeight * this.imgScale) + 480) {
            this.imgtop = ((this.actualDistanceHeight * (this.imgScale - 1))/2) - (this.actualDistanceHeight * this.imgScale) + 480
        }
        this.imgleft = this.imgleft + ev.movementX;
        if (this.imgleft > (this.actualDistanceWidth * (this.imgScale - 1))/2) {
            this.imgleft = (this.actualDistanceWidth * (this.imgScale - 1))/2
        }
        // mapbox的实际高度，下同
        if (this.imgleft < ((this.actualDistanceWidth * (this.imgScale - 1))/2) - (this.actualDistanceWidth * this.imgScale) + 1053) {
            this.imgleft = ((this.actualDistanceWidth * (this.imgScale - 1))/2) - (this.actualDistanceWidth * this.imgScale) + 1053
        }
    }
},
// 开启监听鼠标滑轮
runWheel(isWheel) {
    //判断依据是 是否在盒子内部
    //true 就是开启
    if (isWheel) {
        document.documentElement.style.overflow = "hidden";
        //兼容性写法，该函数也是网上别人写的，不过找不到出处了，蛮好的，所有我也没有必要修改了
        //判断鼠标滚轮滚动方向
        if (window.addEventListener) {
            //FF,火狐浏览器会识别该方法
            window.addEventListener(
                "DOMMouseScroll",
                this.wheel,
                false
            );
        }
        window.onmousewheel = document.onmousewheel = this.wheel; //W3C
    } else {
        //false就是关闭
        document.documentElement.style.overflow = "";
        if (window.addEventListener) {
            //FF,火狐浏览器会识别该方法
            window.addEventListener("DOMMouseScroll", null, false);
        }
        window.onmousewheel = document.onmousewheel = null; //W3C
    }
},
//统一处理滚轮滚动事件，中间件
wheel(event) {
    let delta = 0;
    if (!event) event = window.event;
    if (event.wheelDelta) {
        //IE、chrome浏览器使用的是wheelDelta，并且值为“正负120”
        delta = event.wheelDelta / 120;
        if (window.opera) delta = -delta; //因为IE、chrome等向下滚动是负值，FF是正值，为了处理一致性，在此取反处理
    } else if (event.detail) {
        //FF浏览器使用的是detail,其值为“正负3”
        delta = -event.detail / 3;
    }
    if (delta) this.handle(delta);
},
//上下滚动时的具体处理函数
handle(delta) {
    if (delta < 0) {
        //向下滚动 缩小
        if (this.imgheight > 10) {
            this.imgheight = this.imgheight - 1;
            this.imgScale = this.imgScale  - 0.05;
            if (this.imgScale < 0.3) {
                this.imgScale = 0.3
            }
            // 同上，主要是缩小的时候会越过边界
            if (this.imgleft > (this.actualDistanceWidth * (this.imgScale - 1))/2) {
                this.imgleft = (this.actualDistanceWidth * (this.imgScale - 1))/2
            }
            if (this.imgleft < ((this.actualDistanceWidth * (this.imgScale - 1))/2) - (this.actualDistanceWidth * this.imgScale) + 1053) {
                this.imgleft = ((this.actualDistanceWidth * (this.imgScale - 1))/2) - (this.actualDistanceWidth * this.imgScale) + 1053
            }
            if (this.imgtop > (this.actualDistanceHeight * (this.imgScale - 1))/2) {
                this.imgtop = (this.actualDistanceHeight * (this.imgScale - 1))/2
            }
            if (this.imgtop < ((this.actualDistanceHeight * (this.imgScale - 1))/2) - (this.actualDistanceHeight * this.imgScale) + 480) {
                this.imgtop = ((this.actualDistanceHeight * (this.imgScale - 1))/2) - (this.actualDistanceHeight * this.imgScale) + 480
            }
        }
    } else {
        //向上滚动 放大
        if (this.imgheight < 500) {
            this.imgheight = this.imgheight + 1;
            this.imgScale = this.imgScale + 0.05;
            if (this.imgScale > 1.4) {
                this.imgScale = 1.4
            }
        }
    }
},
//获取背景图片的宽度或高度，和实际距离相除得到 系数distanceCoefficient
getbackgroundImgWidth() {
    if (this.actualDistanceWidth !== 0) {
        let backgroundImg = document.getElementById("backgroundImg");
        this.distanceCoefficient =
            backgroundImg.width / this.actualDistanceWidth;
    } else if (this.actualDistanceHeight !== 0) {
        let backgroundImg = document.getElementById("backgroundImg");
        this.distanceCoefficient =
            backgroundImg.height / this.actualDistanceWidth;
    }
},
~~~

### 说明

大部分内容参考自[此处](https://zhuanlan.zhihu.com/p/228355593)。这这篇文章中使用了imgheight的参数来百分比缩放图片。本文使用的是transform:scale(x,y)来进行缩放，并增加了缩放，拖动时候有边界的限制：

+ 左上方的限制：this.actualDistanceWidth * (this.imgScale - 1))/2

```javascript

if (this.imgleft > (this.actualDistanceWidth * (this.imgScale - 1))/2) {
	this.imgleft = (this.actualDistanceWidth * (this.imgScale - 1))/2
}
```

+ 右下方的限制： (this.actualDistanceWidth * (this.imgScale - 1))/2) - (this.actualDistanceWidth * this.imgScale) + (图片显示区域的宽度)

```javascript
        if (this.imgleft < ((this.actualDistanceWidth * (this.imgScale - 1))/2) - (this.actualDistanceWidth * this.imgScale) + 1053) {
            this.imgleft = ((this.actualDistanceWidth * (this.imgScale - 1))/2) - (this.actualDistanceWidth * this.imgScale) + 1053
        }
```

以左上方的限制为参考，减去（左上拖动）图片实际尺寸×缩放比再加上显示区域尺寸

高度的限制相同。

### 局限

滚轮缩放的原点不是鼠标位置。



### 参考

[Vue JS鼠标拖动图片 滚轮缩放图片](https://zhuanlan.zhihu.com/p/228355593)

