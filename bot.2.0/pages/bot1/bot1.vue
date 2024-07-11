<template>
	<view>
		第二页面（弹窗访问）
	</view>
	<view class="box" hover-class="boxHover">是否获取摄像头和蓝牙权限</view>
	<view class="box2" hover-class="box2Hover"hover-stay-time="0">拒绝</view>
	<view class="box3" hover-class="box3Hover"hover-stay-time="0">允许</view>
	
	

</template>

<script>
	export default {
		data() {
			return {
				
			}
		},
		methods: {
			
		}
	}
</script>

<style lang="scss">
	.box{
		position: absolute;
		bottom: 500px;
		right:115px;
		width:200px;
		height:100px;
		background-color: #b8b8b8;
	}
	.box2{
		position: absolute;
		bottom:480px;
		right:215px;
		width:100px;
		height:20px;
		background-color: #a2f3ff;
	}
	.box2Hover{
		background-color: lightgreen;
	}
	.box3{
		position: absolute;
		bottom: 480px;
		right:115px;
		width:100px;
		height:20px;
		background-color: #a2f3ff;
	}
	.box3Hover{
		background-color: lightgreen;
	}
	
	

</style>
