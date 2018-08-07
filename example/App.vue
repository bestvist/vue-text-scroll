<template>
    <div id="app">
        <div>
            <my-class :data="listData" class="warp" :option="option">
                <ul class="item">
                    <a target="_blank" href="#" v-for="item in listData">
                        <span class="title" v-text="item.title"></span>
                        <span class="date" v-text="item.date"></span>
                    </a>
                </ul>
            </my-class>
        </div>
    </div>
</template>

<script>
import myClass from "src/components/myClass.vue";
export default {
    name: "app",
    components: {
        myClass
    },
    data() {
        return {
            listData: [
                {
                    title: "无缝滚动第一行无缝滚动第一行",
                    date: "2017-12-16"
                },
                {
                    title: "无缝滚动第二行无缝滚动第二行",
                    date: "2017-12-16"
                },
                {
                    title: "无缝滚动第三行无缝滚动第三行",
                    date: "2017-12-16"
                },
                {
                    title: "无缝滚动第四行无缝滚动第四行",
                    date: "2017-12-16"
                },
                {
                    title: "无缝滚动第五行无缝滚动第五行",
                    date: "2017-12-16"
                }
            ],
            option: {
                step: 0.5,
                limitMoveNum: 5,
                openTouch: false
            }
        };
    },
    computed: {},
    created() {}
};
</script>

<style lang="scss">
.clearfix {
    zoom: 1;
    &:after {
        display: block;
        height: 0;
        clear: both;
        content: ".";
        visibillity: hidden;
    }
}

.warp {
    height: 60px;
    width: 360px;
    overflow: hidden;
    ul {
        list-style: none;
        padding: 0;
        margin: 0 auto;
        li,
        a {
            display: block;
            height: 30px;
            line-height: 30px;
            display: flex;
            justify-content: space-between;
            font-size: 15px;
        }
    }
}
</style>
