 <!--
 * @Author: HJQ
 * @Date: 2021-11-17 11:32:04
 * @LastEditTime: 2021-11-17 11:32:04
 * @Description: 实现数字翻牌动态效果，css 属性 writing-mode: vertical-lr，使数字竖直排版，
        2d移动 transform: translate(-50%, -40%); y值控制移动至哪个数字，transition
        控制transform属性有动画效果
 -->
<template>
    <div class="chartNum">
        <div class="box-item">
            <li :class="{'number-item': !isNaN(item), 'mark-item': isNaN(item) }"
                :style="{backgroundImage: (!isNaN(item) ? `url('${img}')` : ''), width: (!isNaN(item) ? `${width}px` : `${commaWidth}px`), fontSize: `${fontSize}px`}"
                v-for="(item,index) in orderNum"
                :key="index">
                    <span v-if="!isNaN(item)">
                        <i class="number" :class="'number-index-' + index" ref="numberItem">0123456789</i>
                    </span>
                    <span class="comma" v-else>{{item}}</span>
            </li>
        </div>
    </div>
</template>
<script>
export default {
    name: 'number-flip',
    props: {
        num: { // 展示的数值，可以是  123    '123'   '1,234
            type: [String, Number],
            required: true
        },
        time: {
            type: Number,
            default: 5000
        },
        itemCount: { // 每次改变的值
            type: [String, Number]
        },
        img: {
            required: false
        },
        width: {
            type: [String, Number],
            default: 50
        },
        fontSize: {
            type: [String, Number],
            default: 40
        },
        commaWidth: {
            type: [String, Number],
            default: 10
        }
    },
    data () {
        return {
            currentNum: '',
            orderNum: [], // 默认订单总数
            interval: null
        };
    },
    watch: {
        num (val) {
            this.currentNum = val;
            this.init();
        }
    },
    mounted () {
        if (this.num) {
            this.currentNum = this.num;
            this.init();
        }
    },
    methods: {
        // 初始化数值，并绑定定时器
        init () {
            this.interval && clearInterval(this.interval);
            setTimeout(() => {
                this.toOrderNum(this.currentNum);
                if (this.itemCount) {
                    this.interval = setInterval(() => {
                        this.changeNum();
                    }, this.time);
                    this.$once('hook:beforeDestroy', () => {
                        clearInterval(this.interval);
                    });
                }
            }, 100);
        },
        // 改变数值
        changeNum () {
            if ((this.currentNum + '').indexOf(',') !== -1) { // 数值有千分位
                this.currentNum = this.commafyback(this.currentNum);
                this.currentNum = parseFloat(this.currentNum) + parseFloat(this.itemCount);
                this.currentNum = this.comdify(this.currentNum);
            } else { // 数值没有千分位
                this.currentNum = parseFloat(this.currentNum) + parseFloat(this.itemCount);
            }
            this.toOrderNum(this.currentNum);
        },
        // 设置千分位
        comdify (n) {
            n = n + '';
            var re = /\d{1,3}(?=(\d{3})+$)/g;
            var n1 = n.replace(/^(\d+)((\.\d+)?)$/, function (s, s1, s2) { return s1.replace(re, '$&,') + s2; });
            return n1;
        },
        // 去掉千分位
        commafyback (num) {
            num = num + '';
            var x = num.split(',');
            return x.join('');
        },
        /**
         * 设置文字滚动
         * 1、通过ref获取每一项数字的dom，并转成数组arr
         * 2、将数值切割后的数组过滤掉逗号并保存到numberArr
         * 3、将arr里面的dom根据  number-index-x  进行从小到大的排序，并将排序后的数组赋值给numberItems
         * 4、遍历numberItems里面的dom元素，该数字值是多少，则设置translateY为  x*10%
         */
        setNumberTransform () {
            let arr = [...this.$refs['numberItem']];
            // let numberItems = [...this.$refs['numberItem']]; // 拿到数字的ref，计算元素数量
            const numberArr = this.orderNum.filter(item => !isNaN(item));
            let numberItems = arr.sort((a, b) => {
                if (parseInt(a.classList[1].split('-')[2]) - parseInt(b.classList[1].split('-')[2]) > 0) {
                    return 1;
                } else if (parseInt(a.classList[1].split('-')[2]) - parseInt(b.classList[1].split('-')[2]) < 0) {
                    return -1;
                }
            });
            // 结合CSS 对数字字符进行滚动,显示订单数量
            for (let index = 0; index < numberItems.length; index++) {
                const elem = numberItems[index];
                elem.style.transform = `translate(-50%, -${numberArr[index] * 10}%)`;
            }
        },
        // 切割数值
        toOrderNum (num) {
            num = num.toString();
            // 把订单数变成字符串
            // if (num.length < this.max) {
            //     num = '0' + num; // 如未满八位数，添加"0"补位
            //     this.toOrderNum(num); // 递归添加"0"补位
            // } else if (num.length === this.max) {
            //     let orderNum = this.commafyback(num);
            //     orderNum = this.comdify(orderNum);
            //     // 订单数中加入逗号
            //     this.orderNum = orderNum.split(''); // 将其便变成数据，渲染至滚动数组
            //     this.$nextTick(() => {
            //         this.setNumberTransform();
            //     });
            // } else {
            //     // 订单总量数字超过八位显示异常
            //     this.$message.warning('总量数字过大');
            // }
            this.orderNum = num.split(''); // 将其便变成数据，渲染至滚动数组
            this.$nextTick(() => {
                this.setNumberTransform();
            });
        }
    }
};
</script>
<style scoped lang='scss'>
.chartNum {
    width: 100%;
    height: 100%;
    /*订单总量滚动数字设置*/
    .box-item {
        position: relative;
        width: 100%;
        height: 100%;
        font-size: 40px;
        text-align: center;
        list-style: none;
        color: rgba($color: #fff, $alpha: 0.85);
        writing-mode: vertical-lr;
        text-orientation: upright;
        /*文字禁止编辑*/
        -moz-user-select: none; /*火狐*/
        -webkit-user-select: none; /*webkit浏览器*/
        -ms-user-select: none; /*IE10*/
        -khtml-user-select: none; /*早期浏览器*/
        user-select: none;
        overflow: hidden;
    }
    /* 默认逗号设置 */
    .mark-item {
        height: 100%;
        margin-right: 5px;
        line-height: 10px;
        // font-size: 40px;
        position: relative;
        & > span {
            position: absolute;
            width: 100%;
            bottom: 0;
            writing-mode: vertical-rl;
            text-orientation: upright;
        }
    }
    /*滚动数字设置*/
    .number-item {
        height: 100%;
        background-size: 100% 100%;
        background-repeat: no-repeat;
        background-position: center center;
        list-style: none;
        margin-right: 5px;
        border-radius:4px;
        & > span {
            position: relative;
            display: inline-block;
            margin-right: 10px;
            width: 100%;
            height: 100%;
            writing-mode: vertical-rl;
            text-orientation: upright;
            overflow: hidden;
            & > i {
                font-style: normal;
                position: absolute;
                top: 6px;
                left: 50%;
                transform: translate(-50%,0);
                transition: transform 1s ease-in-out;
                letter-spacing: 10px;
            }
        }
    }
    .number-item:last-child {
        margin-right: 0;
    }
}
</style>
