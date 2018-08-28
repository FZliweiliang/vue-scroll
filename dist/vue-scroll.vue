<template>
	<div class="my-scroll" :class="[scrollState?'prohibit':'allow']" ref="myScroll" @scroll.passive="onScroll($event)" @touchstart="touchStart($event)" @touchmove="touchMove($event)" @touchend="touchEnd($event)"  >
		<div class="scroll-top" :style="{height:top+'px'}">
			<div v-if="aspect==2">
				<p v-if="state==6">
					下拉刷新
				</p>
				<p v-if="state==1">
					<i><img :src="Load"/></i>
					<br/>
					刷新中
				</p>
				<p v-if="state==2">松开刷新</p>
				<p v-if="state==3">
					<i><img :src="Load"/></i>
					<br/>
					刷新完成
				</p>
			</div>
		</div>
		<!-- top -->
		<div class="scroll-list" :style="{ transform: 'translate3d(0, ' + top + 'px, 0)'}">
			<slot name='scrollList'></slot>
			<div class="scroll-bottom">
				<div v-if="state==4">加载中</div>
				<div v-if="state==5">加载完成</div>
				<div v-if="state==7">没有更多</div>
			</div>
		</div>
	</div>
</template>
<script type="text/javascript">
import tween from '../lib/tween'
import Load from './assets/Load.gif'
	export default {
		name:'myScroll',
		props:{
			'page':{
				type:Object,  //counter:当前页  pageStart:开始页数  pageEnd:结束页数  total:总页数
			},
			'onRefresh':{ //刷新回调
				type:Function,
				require:true
			},
			'onPull':{ //加载回调
				type:Function,
				require:true
			},
			'getScrollTop':{ //获取滚动条位置
				type:Function
			},
			'setScrollPage':{ //改变滚动条位置
				type:Function
			},
			'scrollState':{//是否可滑动
				type:Boolean,
				require:true
			}
		},
		data(){
			return {
				Load,
				pageX:0,
				pageY:0,
				state:0, 
				scrollPosition:0,
				myScroll:null,
				myScrollList:null,
				top:0,
				aspect:0, //1:向下 2:向上
				listHeight:0,
 			}
		},
		created(){
			
		},
		methods:{
			ScrollTop(top){ //修改滚动条位置
				this.myScroll.scrollTop = top
			},
			/*
			 * 刷新中：1
			 * 松开刷新：2
			 * 刷新完成：3
			 * 加载中：4
			 * 加载完成：5
			 * 下拉刷新：6
			 * 没有更多：7
			 */
			setState(index){ //修改状态
				this.state = index
		        if(index == 5||index == 3){
					setTimeout(()=>{
						this.state = 0
						let timer = null;
						let that = this;
						var b=50,c=100,d=100,t=0;
						cancelAnimationFrame(timer);
						timer = requestAnimationFrame(function fn() {
							var oTop = that.top;
							if (that.top>0) {
								that.top = Math.ceil(tween.Quart.easeInOut(10,oTop,8,10)) - 15
								timer = requestAnimationFrame(fn);
							} else {
								cancelAnimationFrame(timer);
								that.top = 0
							}
						});
					},500)
				}
			},
			touchStart(e){ //触摸事件
				this.pageX = e.targetTouches[0].pageX
				this.pageY = e.targetTouches[0].pageY
			},
			touchMove(e){ //触摸滑动事件
				this.scrollPosition = this.myScroll.scrollTop //获取滚动条位置
				if(this.scrollState&&e.targetTouches[0].pageY>this.pageY){ //向上滑动
					this.aspect = 2
					if(this.myScroll.scrollTop==0){
						let diff = e.targetTouches[0].pageY - this.pageY - this.scrollPosition
						this.top = Math.pow(diff, 0.9)
						let ranget = diff/document.body.clientHeight*100 //计算在屏幕上滑动了多少
						if(ranget > 20){
							this.state = 2
						}else if(ranget < 15){
							this.state = 6
						}
						 e.preventDefault()
					}
				}else if(this.scrollState&&this.state!=4){ //向上滑动
					this.aspect = 1
				}
				
			},
			touchEnd(e){
				if(this.aspect == 2&&this.state == 2||this.state == 1){ //上拉处理
					this.top = 100
					this.state = 1
					this.topCallback()
				}else if(this.aspect == 2){
					this.state = 0
					this.top = 0
				}
			},
			onScroll(e){
				let listHeight = this.myScrollList.offsetHeight //列表总高度
				let listScrollTop = e.target.scrollTop + this.myScroll.offsetHeight //当前滚动条位置

				if(this.state == 0&&listHeight-listScrollTop < 100){
					this.bottomCallback()
				}

				if(this.getScrollTop)this.getScrollTop(e.target.scrollTop) //返回X，Y
			},
			topCallback(){ //刷新回调
				this.onRefresh(this.state)
			},
			bottomCallback(){ //加载回调
				if(this.state != 7){
					this.state = 4
					this.onPull(this.state)
				}
				
			},
		},
		mounted(){
			this.myScroll = this.$refs.myScroll //获取滑条dom
			this.myScrollList = this.myScroll.children[1] //获取列表dom
		}
	}
</script>
<style lang="scss" scoped>
	.allow{
		overflow:hidden;
		height: auto;
	}
	.prohibit{
		max-width: 100%;
		max-height: 100%;
	    height: 100%;
		overflow:hidden;
		overflow-y: scroll;
		-webkit-overflow-scrolling: touch;
		will-change: transform;
	    transition: all 450ms;
	    backface-visibility: hidden;
	    perspective: 1000;
	}
	.my-scroll{
	    position: relative;
		color: #fff;
	    .scroll-top{
	    	text-align: center;
	    	display:flex;
	    	position:absolute;
	    	top:0;
	    	left:0;
			width:100%;
			overflow: hidden;
	    	div{
	    		display:flex;
				height:auto;
	    		width:100%;
	    		justify-content: center;
	    		align-items:center;
	    		flex-wrap: wrap;
	    		i{
	    			flex:1 0 100%;
					display:block;
					height: 0.4rem;
	    		}
	    		img{
	    			width:0.6rem;
	    		}
				p{
					flex:1 0 100%;
				}
	    	}
	    }
		.scroll-list{
			overflow:hidden;
			min-height: 100%;
		}
		.scroll-bottom{
			text-align: center;
			line-height: 40px;
		}
	}
</style>