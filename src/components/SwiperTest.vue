<!--loop为true情况下，动态插入的slide会顺序错乱 1231423 向左滑将一直看不到4一直321有时能看到3重复 向右滑是 12314231423-->
<!--这个问题是swiper本身的bug，新增的数据直接进入dom的最后一个，而在loop里顺序是和无loop不同的，其实应该重新生成swiper，才能有正确的顺序-->
<!--靠在swiper 上加 v-if="swiperSlides.length"并不能够实现刷新，尝试通过调用api方法来实现，update方法和updateSlide都试了也不行-->
<!--最终的解决方案：this.swiperSlides = [];然后在nextTick 里再赋值，就能保证顺序的问题，4会插在3后面；v-if="swiperSlides.length>0"也要加，否则不能autoplay,如果本来就不需要autoplay则可以不加-->
<template>
    <div>
        <a href="#" class="append-slide" @click="appendSlides">Append Slide</a>

        <swiper v-if="swiperSlides.length>0" :options="swiperOption" ref="mySwiper">
            <!-- slides -->
            <swiper-slide v-for="(slide, index) in swiperSlides" :key="index">I'm Slide {{ slide }}</swiper-slide>
            <!--        <swiper-slide>I'm Slide 1</swiper-slide>-->
            <!--        <swiper-slide>I'm Slide 2</swiper-slide>-->
            <!--        <swiper-slide>I'm Slide 3</swiper-slide>-->
            <!--        <swiper-slide>I'm Slide 4</swiper-slide>-->
            <!--        <swiper-slide>I'm Slide 5</swiper-slide>-->
            <!--        <swiper-slide>I'm Slide 6</swiper-slide>-->
            <!--        <swiper-slide>I'm Slide 7</swiper-slide>-->
            <!-- Optional controls -->
            <div class="swiper-pagination"  slot="pagination"></div>
            <div class="swiper-button-prev" slot="button-prev"></div>
            <div class="swiper-button-next" slot="button-next"></div>
            <div class="swiper-scrollbar"   slot="scrollbar"></div>
        </swiper>
    </div>
</template>

<script>
    /* eslint-disable */
    import 'swiper/dist/css/swiper.css'

    import { swiper, swiperSlide } from 'vue-awesome-swiper'
    export default {
        name: "SwiperTest",
        components: {
            swiper,
            swiperSlide
        },
        data() {
            return {
                swiperOption: {
                    // some swiper options/callbacks
                    // 所有的参数同 swiper 官方 api 参数
                    loop: true,
                    autoplay: { delay: 1000 },
                    slidesPerView: 1,
                    centeredSlides: true,
                    spaceBetween: 30,
                    pagination: {
                        el: '.swiper-pagination',
                        clickable: true,
                    },
                    navigation: {
                        nextEl: '.swiper-button-next',
                        prevEl: '.swiper-button-prev',
                    },
                },
                swiperSlides: [1, 2, 3]
            }
        },
        methods: {
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
        },
        computed: {
            swiper() {
                return this.$refs.mySwiper.swiper
            }
        },
        mounted() {
            // current swiper instance
            // 然后你就可以使用当前上下文内的swiper对象去做你想做的事了
            console.log('this is current swiper instance object', this.swiper)
            // this.swiper.slideTo(3, 1000, false)
        }
    }
</script>

<style scoped>

</style>