<template>
    <view>
        <scroll-view
                    scroll-y
                    class="box"
                >
                    <view class="item" v-for="item in blueDeviceList" @click="connect(item)">
                        <view>
                            <text>id: {{ item.deviceId }}</text>    
                        </view>
                        <view>
                            <text>name: {{ item.name }}</text>  
                        </view>
                    </view>
                </scroll-view>
                
                <button @click="initBlue">初始化蓝牙</button>
                <button @click="discovery">搜索附近蓝牙设备</button>
				<button @click="getService">获取蓝牙服务</button>
				<button @click="getCharacteristics">获取特征值</button>
				<button @click="notify">开启消息监听</button>
				<button @click="getConnectionState"> 监测连接状态</button>

    </view>
</template>
​
<script setup>
	import { ref } from "vue"
	const blueDeviceList=ref([])
	const bleDeviceid=ref('')
	const bleServiceid=ref('')
	const bleCharacteristicId=ref('')
	
	
	function getConnectionState(){
		uni.onBLEConnectionStateChange(res=>{
			console.log(res)
			if(res.connected){
				console.log(res.deviceId,"正常连接")
			}else{
				console.log("连接断开")
			}
		})
	}
	
	// 【1】初始化蓝牙
	function initBlue() {
		uni.getAppAuthorizeSetting()
	    uni.openBluetoothAdapter({
	        success(res) {
	            console.log('初始化蓝牙成功')
	            console.log(res)
	        },
	        fail(err) {
	            console.log('初始化蓝牙失败')
	            console.error(err)
	        }
	    })
	}
	
	
	// 【2】开始搜寻附近设备
	function discovery() {
	    uni.startBluetoothDevicesDiscovery({
	        success(res) {
	            console.log('开始搜索')
	            
	            // 开启监听回调
	            uni.onBluetoothDeviceFound(found)
	        },
	        fail(err) {
	            console.log('搜索失败')
	            console.error(err)
	        }
	    })
	}
	 
	// 【3】找到新设备就触发该方法
	function found(res) {
	    console.log(res)
	    blueDeviceList.value.push(res.devices[0])
	} 
	
	
	//要连接的蓝牙deviceid
	
	
	//【4】蓝牙连接
	function connect(data){
		bleDeviceid.value=data.deviceId
		bleServiceid.value=data.serviceId
		uni.createBLEConnection({
			deviceId:bleDeviceid.value,
			success(res){
				console.log("连接成功")
			// 停止搜索
	            stopDiscovery()
	        },
	        fail(err) {
	            console.log('连接失败')
	            console.error(err)
	        }
	    })
	function stopDiscovery(){
		uni.stopBluetoothDevicesDiscovery({
				success(res){
					console.log("关闭成功")
					console.log(res)
				},
				fail(err){
					console.log("关闭失败")
					console.log(err)
				}
			}
		)}
		
	function getService(){
		console.log('获取服务方法启动')
		uni.getBLEDeviceServices({
			deviceId:bleDeviceid.value,
			success(res){
				console.log("获取服务成功")
				console.log(res)
			},
			fail(err){
				console.log("获取服务失败")
				console.log(err)
			}
		})
	}
	
	//【7】获取特征值
	function getCharacteristics(){
		console.log("获取特征值方法启动")
		uni.getBLEDeviceCharacteristics({
			deviceId:bleDeviceid.value,
			serviceId:bleServiceid,
			success(res){
				bleCharacteristicId.value=res.characteristics[0].uuid
				console.log('获取特征值成功')
				console.log(res)
			},
			fail(err){
				console.log('获取特征值失败')
				console.log(err)
			}
		})
	}
		
	function notify(){
		console.log("获取监听方法启动")
		uni.notifyBLECharacteristicValueChange({
			deviceId:bleDeviceid,
			serviceId:bleServiceid,
			 characteristicId:bleCharacteristicId,
			 success(res){
				 console.log("监听成功")
				 console.log(res)
				 listenValueChange()
			 },
			fail(err){
				console.log('监听失败')
				console.log(err)
			}
		
		})
	}
		
	function listenValueChange(){
		uni.onBLECharacteristicValueChange(res=>{
			console.log(res)
			console.log('消息变化')
		})
	}
		
		
		
	}
</script>

<style> .box {
    width: 100%;
    height: 400rpx;
    box-sizing: border-box;
    margin-bottom: 20rpx;
    border: 2px solid dodgerblue;
}
.item {
    box-sizing: border-box;
    padding: 10rpx;
    border-bottom: 1px solid #ccc;
}
button {
    margin-bottom: 20rpx;
} </style>
