<!-- 通用海报绘制组件 -->
<template>
	<view>
		<!-- 
		  绑定画布大小，单位为rpx
		  -->
		<canvas canvas-id="myCanvas" class="mycanvas" :style="{ width: canvasWidth + 'rpx', height: canvasHeight + 'rpx' }"></canvas>
	</view>
</template>
<script>
	export default {
		name: 'drawPicture',
		data() {
			return {
				ctx: null,
				bool:false,
			};
		},
		props: {
			/*
			 * 绘制数据
			 */
			drawData: {
				type: Array,
				default: () => []
			},
			/*
			 * 画布宽度
			 */
			canvasWidth: {
				type: Number,
				default: () => uni.upx2px(750)
			},
			/*
			 * 画布高度
			 */
			canvasHeight: {
				type: Number,
				default: () => uni.upx2px(500)
			}
		},
		mounted() {
			/* 创建画布上下文 */
			this.ctx = uni.createCanvasContext('myCanvas', this);
			// this.ctx = uni.createCanvasContext('myCanvas');
		},
		methods: {
			/* 绘制 */
			async draw() {
				if (!this.drawData || this.drawData.length === 0) {
					this.showToast('暂无信息,请重新生成海报');
					return;
				}
				uni.showLoading({
				    title: '生成中...',
					mask:true,
				});
				/* 遍历绘制 */
				for (let i = 0; i < this.drawData.length; i++) {
					await this.drawItem(this.drawData[i], i);
				}
				/* 结束绘制 */
				console.log('绘制完成!');
				this.ctx.draw();
				/* draw完不能立刻转存，需要等待一段时间 */
				setTimeout(() => {
					console.log('进来了!');
					uni.canvasToTempFilePath({
						canvasId: 'myCanvas',
						success: (res) => {
							// 在H5平台下，tempFilePath 为 base64
							console.log(res.tempFilePath);
							uni.saveImageToPhotosAlbum({
								filePath: res.tempFilePath,
								success:(ress)=> {
									console.log(ress.errMsg);
									this.showToast('已保存至本地相册,快去分享给好友吧')
									this.$root.canvasDisable = false
								},
								fail:(err)=> {
									console.log(err);
									this.showToast('保存失败,已取消保存')
									this.$root.canvasDisable = false
								}
							})
						},
						fail:(err) => {
							console.log('fail---',err);
							uni.showToast({
								title: '保存失败,已取消保存',
								icon: 'none'
							})
							this.$root.canvasDisable = false
						},
						complete: (errs) => {
							console.log(errs);
							this.$root.canvasDisable = false
						}
					}, this)
				}, 500);
			},
			drawItem(item, index) {
				return new Promise((resolve, reject) => {
					let typeName = item.type === 'text' ? '文本' : item.type === 'image' ? '图像': '不支持的类型';// : item.type === 'image1'? '图像' 
					console.log(`当前绘制队列: 第${index + 1}个,绘制类型为: ${typeName}`);
					if (typeName === '文本') {
						if(this.bool){
							console.log('this.bool',this.bool);
							this.ctx.restore()
						}
						console.log('绘制文本：' + item.text);
						this.ctx.save()
						this.ctx.setFillStyle(item.color);
						this.ctx.setFontSize(uni.upx2px(item.size));
						// item.font?this.ctx.font = item.font:this.ctx.font ='';
						this.ctx.fillText(item.text, uni.upx2px(item.x), uni.upx2px(item.y));
						resolve('success');
					}
					if (typeName === '图像') {
						console.log('绘制图片：' + item.path);
						if(item.path==undefined || item.path=='undefined'){
							uni.showModal({
								content:'图片生成失败，请重新尝试',
								showCancel:false,
								success: () => {}
							})
							return
						}
						uni.getImageInfo({
							src: item.path,
							success: res => {
								console.log('远程图像绘制完毕',this.ctx,res.path);
								// if(item.type === 'image1'){
								// 	// this.ctx.save()
								// 	this.ctx.beginPath()
								// 	this.ctx.arc(uni.upx2px(item.x) + (uni.upx2px(item.width) / 2), uni.upx2px(item.y)+(uni.upx2px(item.width) / 2), uni.upx2px(item.width) / 2, 0, 2 * Math.PI)
								// 	this.ctx.clip()	
								// }
								if(item.name=='head'){
									this.ctx.save()
									this.ctx.beginPath()
									this.ctx.arc(uni.upx2px(item.x) + (uni.upx2px(item.width) / 2), uni.upx2px(item.y)+(uni.upx2px(item.width) / 2), uni.upx2px(item.width) / 2, 0, 2 * Math.PI)
									this.ctx.clip()
									this.bool = true
									console.log('this.bool',this.bool);
								}
								this.ctx.drawImage(res.path, uni.upx2px(item.x), uni.upx2px(item.y), uni.upx2px(item.width), uni.upx2px(item.height));
									this.ctx.restore()
								resolve('success');
							}
						});
					}
				});
			}
		}
	};
</script>
<style lang="scss" scoped>
	.mycanvas{
		position:absolute;
		top:-100vh;
		z-index:-999999999999999;
	}
</style>
