# swiper-async-loop

## Project setup
```
yarn install
```

### Compiles and hot-reloads for development
```
yarn run serve
```

### Compiles and minifies for production
```
yarn run build
```

### Run your tests
```
yarn run test
```

### Lints and fixes files
```
yarn run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

```html
<!-- Chinese version -->
```
Async data example in the README.md，
```html
<template>
  <swiper :options="swiperOption">
    <swiper-slide v-for="(slide, index) in swiperSlides" :key="index">I'm Slide {{ slide }}</swiper-slide>
    <div class="swiper-pagination" slot="pagination"></div>
  </swiper>
</template>

<script>
  export default {
    name: 'carrousel',
    data() {
      return {
        swiperOption: {
          pagination: {
            el: '.swiper-pagination'
          }
        },
        swiperSlides: [1, 2, 3, 4, 5]
      }
    },
    mounted() {
      setInterval(() => {
        console.log('simulate async data')
        if (this.swiperSlides.length < 10) {
          this.swiperSlides.push(this.swiperSlides.length + 1)
        }
      }, 3000)
    }
  }
</script>
```
Readme.md里的Async data example，有一个bug，在loop为true时，新加进来的slides顺序不是在最后，而是在复制的slide的后面，这个是有问题的。
举个例子，原本slides是123，loop的话是123123；新增一个4进来后，期望是12341234，然而readme里的this.swiperSlides.push方法后，会发现顺序是12314231423；并且只有向右是这样，如果向左滑动，发现4已经不见了。这不难理解，打开dom就发现，4被放在复制的1后面，3仍然被复制而4没有被复制，而且className都没有正确的设置。
solution：
```javascript
appendSlides(){
                let tempArray = this.swiperSlides.slice();// 如果需要就先把原先的内容暂存起来，如果后面接口返回的是全量数据则不用暂存
                this.swiperSlides = [];// 这个是关键，先把slides内容清空
                this.$nextTick(function () {
                    // 然后在nextTick里给swiperSlides赋值；用nextTick的目的是不让vue做自动合并，而是让swipe先清掉内容，再重新增加swiper-slide，这样就不会发生顺序问题。
                    tempArray.push(tempArray.length + 1);
                    this.swiperSlides = tempArray.slice();
                })
                // this.swiperSlides.push(this.swiperSlides.length + 1); // 只是这样增加内容，slide循环的顺序会有问题 123123 --> 12314231423
                // this.swiper.update();//靠update方法也无法使swiper重新加载
            }
```

此外，如果需要autoplay，还要在swiper里给一个 v-if, 比如：
```html
<swiper v-if="swiperSlides.length>0" :options="swiperOption" ref="mySwiper">
```
让它能够更新，否则autoplay在slides刷新后有丢失的bug。
