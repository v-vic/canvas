# canvas
uniapp使用canvas生成画布图片，并下载至本地，安卓IOS小程序通用
page.vue中引入
`<drawPicture ref="myDraw" :drawData="drawData" :canvasWidth="canvasWidth" :canvasHeight="canvasHeight"></drawPicture>`
data中定义
`data() {
		return {
			drawData:[],
			canvasWidth:656,
			canvasHeight:350,
			}
		}`
		
@click事件使用
`this.drawData.push({
				path:图片地址,
				x: 横轴坐标,
				y: 纵轴坐标,
				width: 图片宽,
				height: 图片高度,
				type: 'image',//固定用法
			})
			this.drawData.push({
				text: 文字信息,
				size: 文字的fontSize,
				x: 横轴坐标,
				y: 纵轴坐标,
				color:文字颜色,
				type: 'text',//固定用法
			})`
最后执行
`setTimeout(()=>{this.$refs.myDraw.draw()},500)`