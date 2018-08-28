# vue-scroll
## Usage

#### Use:

```html
<template>
  <div>
    <my-scroll ref="myScroll" :on-refresh="onRefresh" :on-pull="onPull" :get-scroll-top="getTop" :scroll-state="scrollState">
					<div slot="scrollList">
            <!-- 列表 -->
					</div>
		</my-scroll>
  </div>
</template>
<script>

import myScroll from "vue-scroll.vue";

new Vue({
  el: '#app',
  data(){
    return{
      scrollState: true, //是否可以滑动
      indexScrollTop:0,
      listdata:[]
    }
  },
  components: {
    myScroll
  },
  methods:{
        onRefresh(mun) { // 刷新
            this.listParams.p = 1;
            this.$axios
                .get(apiUrl.noticeList, {
                    params: this.listParams,
                    isLoading: false
                })
                .then(res => {
                    if (res.code == 10000) {
                        this.listParams.p++;
                        this.listdata = res.data;
                        this.$refs.myScroll.setState(3);
                    } else {
                        this.$refs.myScroll.setState(3);
                    }
                });
        },
        onPull(mun) { //加载
            this.$axios
                .get(apiUrl.noticeList, {
                    params: this.listParams,
                    isLoading: false
                })
                .then(res => {
                    if (res.code == 10000 && res.data.length > 0) {
                        this.listParams.p++;
                        res.data.map((v, k) => {
                            this.listdata.push(v);
                        });
                        this.$refs.myScroll.setState(5);
                    } else {
                        this.$refs.myScroll.setState(7);
                    }
                });
        },
        getTop(y) {//滚动条位置
            this.indexScrollTop = y;
        },
  },
});
</script>
```
<br>


## Options

### Props
| Props        | Type     | Default | Description                                                        |
| ------------ | :------- | ------- | ------------------------------------------------------------------ |
| page         | Object   | auto    | counter:当前页  pageStart:开始页数  pageEnd:结束页数  total:总页数 |
| onRefresh    | Function | true    | 刷新回调                                                           |
| onPull       | Function | true    | 加载回调                                                           |
| getScrollTop | Function | auto    | 获取滚动条位置                                                     |
| scrollState  | Boolean  | true    | 获取滚动条位置                                                     |

### Function
| Name         | Type   | Description                                                                                   |
| ------------ | :----- | --------------------------------------------------------------------------------------------- |
| setState     | Number | setState(状态) 刷新中：1 松开刷新：2 刷新完成：3加载中：4 加载完成：5 下拉刷新：6 没有更多：7 |
| setScrollTop | Number | setScrollTop(滚动条位置)                                                                      |

### Screenshot
<img src="https://github.com/474782977/vue-slider/blob/master/screenshot/1.jpg" width="320px" style="display:inline;">

