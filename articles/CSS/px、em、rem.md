## 区别
1. px是固定的像素，一旦设置了就无法因为适应页面大小而改变
2. em和rem相对于px更具有灵活性，它们是相对长度单位，即长度不固定，适合响应式布局
3. em是相对其父元素来设置字体大小的，一般都是以body的font-size为基准。这样会有一个问题，进行任何元素设置，
都可能需要知道他父元素的大小。
4. rem是相对于根元素html的font-size；


### em相对于父元素
```html
<div>
    父元素div
    <p>
       子元素p
        <span>孙元素span</span>
    </p>
</div>
```
```css
div {
        font-size: 40px;
        width: 7.5em; /* 300px */
        height: 7.5em;
        border: solid 2px black;
    }
 p {
        font-size: 0.5em; /* 20px */
        width: 7.5em; /* 150px */
        height: 7.5em;
        border: solid 2px blue ;
        color: blue;
    }span {
        font-size: 0.5em;
        width: 7em;
        height: 6em;
        border: solid 1px red;
        display: block;
        color: red;
    }
```


### rem相对于更元素 html。

```html
<div>
    父元素div
    <p>
       子元素p
        <span>孙元素span</span>
    </p>
</div>
```

```css
html {
    font-size: 10px;
}
div {
    font-size: 4rem; /* 40px */
    width: 20rem;  /* 200px */
    height: 20rem;
    border: solid 2px black;
}
p {
    font-size: 2rem; /* 20px */
    width: 10rem;
    height: 10rem;
    border: solid 1px blue;
    color:blue ;
}
span {
    font-size: 1.5rem;
    width: 7rem;
    height: 6rem;
    border: solid 2px red;
    display: block;
    color: red;
}
```