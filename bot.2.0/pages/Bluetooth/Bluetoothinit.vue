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
				<view>
                <button class="button button-blue button-outline" @click="initBlue">初始化蓝牙</button>
				<br/>
                <button class="button button-yellow button-outline" @click="discovery">搜索附近蓝牙设备</button>
				<br />
				<view>
					<input class="input" data-id="code" key="code" type="text" v-model="code" placeholder="请输入指令"/>
				</view>
				<br />
				<button class="button button-outline button-green " @click="send(code)">发送指令</button>
				</view>

    </view>
</template>
​
<script setup>
	import { ref } from "vue"
	
	/* 
	 变量依次为：
	 所获取的蓝牙列表
	 设备id
	 服务id
	 特征值
	 指令
	 */
	
	const blueDeviceList=ref([])
	const bleDeviceid=ref('')
	const bleServiceid=ref('')
	const bleCharacteristicId=ref('')
	const code=ref('')
	
	//监测连接状态
	function getConnectionState(){
		uni.onBLEConnectionStateChange(res=>{
			if(res.connected){
				console.log(res.deviceId,":正常连接")
			}else{
				console.log("蓝牙连接断开")
			}
		})
	}
	
	//初始化蓝牙
	function initBlue() {
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
	
	
	//开始搜寻附近设备
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
	 
	//找到新设备就触发该方法
	function found(res) {
	    console.log(res)
	    blueDeviceList.value.push(res.devices[0])
	} 
	
	
	//蓝牙连接
	function connect(data){
		getConnectionState()
		bleDeviceid.value=data.deviceId
		uni.createBLEConnection({
			deviceId:bleDeviceid.value,
			success(res){
				console.log(res)
				console.log("连接成功")
			// 停止搜索
	            stopDiscovery()
			//获取服务
				getService()
			//获取特征值
				getCharacteristics()
				
			//开启监听
				notify()
				
				console.log("方法调用了吗？")
	        },
	        fail(err) {
	            console.log('连接失败')
	            console.error(err)
	        }
	    })
		}
		
	//停止扫描
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
	
	//获取服务
	function getService(){
		console.log('获取服务方法启动')
		uni.getBLEDeviceServices({
			deviceId:bleDeviceid.value,
			success(res){
				bleServiceid.value=res.services[0].uuid
				console.log("获取服务成功")
				console.log(res)
			},
			fail(err){
				console.log("获取服务失败")
				console.log(err)
			}
		})
	}
	
	//获取特征值
	function getCharacteristics(){
		console.log("获取特征值方法启动")
		uni.getBLEDeviceCharacteristics({
			deviceId:bleDeviceid.value,
			serviceId:bleServiceid.value,
			success(res){
				bleCharacteristicId.value=res.characteristics[0].uuid
				console.log('获取特征值成功')
				console.log(bleCharacteristicId.value,"值是否存在？")
				console.log(bleDeviceid)
				console.log(bleServiceid)
				console.log(res)
			},
			fail(err){
				console.log('获取特征值失败')
				console.log(err)
			}
		})
	}
	
	
	//开启监听
	function notify(){
		uni.notifyBLECharacteristicValueChange({
			state: true,
			deviceId:bleDeviceid.value,
			serviceId:bleServiceid.value,
			characteristicId:bleCharacteristicId.value,
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
	
	
	function send(code) {
	    // 向蓝牙设备发送一个0x00的16进制数据
	    const buffer = new ArrayBuffer(code.length)
	    const dataView = new DataView(buffer)
	    // dataView.setUint8(0, 0)
	    
	    for (var i = 0; i < code.length; i++) {
	      dataView.setUint8(i, code.charAt(i).charCodeAt())
	    }
	    
	    uni.writeBLECharacteristicValue({
	      deviceId: bleDeviceid.value, // 设备ID，在【4】里获取到
	      serviceId: bleServiceid.value, // 服务UUID，在【6】里能获取到
	      characteristicId: bleCharacteristicId.value, // 特征值，在【7】里能获取到
	      value: buffer,
	      success(res) {
	        console.log(res)
	      },
	      fail(err) {
	        console.error(err)
	      }
	    })
	}
		
		
	
</script>

<style> 
.box {
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
/* 基础按钮样式 */
.button {
                      
  margin: 5% 10% 5% 10%;
  
  padding: 10px 80px;
  font-size: 16px;
  
  text-align: center;
  text-decoration: none;
  outline: none;
  color: #fff;
  border: none;
  border-radius: 8px;
  background-color: #66afe9;
  
}



/* 按钮按下效果 */
.button:active {
  background-color: #3e8e41;
  box-shadow: 0 5px #666;
  transform: translateY(4px);
}




/* 基础样式 */
.input {
  height: 100%;
  width: 100%; /* 宽度100%以适应容器 */
  padding: 12px 20px; /* 内边距 */
  margin: 8px 0; /* 外边距 */
  box-sizing: border-box; /* 包括padding和border在width和height内 */
  border: 2px solid #ccc; /* 边框样式 */
  border-radius: 4px; /* 圆角 */
  background-color: #f8f8f8; /* 背景颜色 */
  font-size: 16px; /* 字体大小 */
  color: #333; /* 文字颜色 */
  transition: all 0.3s ease; /* 过渡动画 */
}



</style>
