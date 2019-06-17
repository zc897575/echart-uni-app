<template>
	<view class="chart-box">
		<!-- #ifdef MP-WEIXIN -->
		<text>{{ ec.disableTouch ? '111' :'touchMove' }}</text>
		<canvas class="ec-canvas" :canvas-id="canvasId" @init="init" @touchstart="touchStart"
		 @touchmove="touchMove" @ouchend="touchEnd">
		</canvas>
		<!-- #endif -->

	</view>
</template>

<script>
	import WxCanvas from './wx-canvas';
	import * as echarts from './echarts';

	let ctx;
	export default {
		data() {
			return {
				ec: {
					onInit(canvas, width, height) {
						const chart = echarts.init(canvas, null, {
							width: width,
							height: height
						});
						canvas.setChart(chart);

						var option = {
							title: {
								text: '测试下面legend的红色区域不应被裁剪',
								left: 'center'
							},
							color: ["#37A2DA", "#67E0E3", "#9FE6B8"],
							legend: {
								data: ['A', 'B', 'C'],
								top: 50,
								left: 'center',
								backgroundColor: 'red',
								z: 100
							},
							grid: {
								containLabel: true
							},
							tooltip: {
								show: true,
								trigger: 'axis'
							},
							xAxis: {
								type: 'category',
								boundaryGap: false,
								data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
								// show: false
							},
							yAxis: {
								x: 'center',
								type: 'value',
								splitLine: {
									lineStyle: {
										type: 'dashed'
									}
								}
								// show: false
							},
							series: [{
								name: 'A',
								type: 'line',
								smooth: true,
								data: [18, 36, 65, 30, 78, 40, 33]
							}, {
								name: 'B',
								type: 'line',
								smooth: true,
								data: [12, 50, 51, 35, 70, 30, 20]
							}, {
								name: 'C',
								type: 'line',
								smooth: true,
								data: [10, 30, 31, 50, 40, 20, 10]
							}]
						};
						chart.setOption(option);
						console.log("chart", chart)
						return chart;
					}
				}
			}
		},
		props: {
			canvasId: {
				type: String,
				default: 'ec-canvas'
			}
		},
		watch: {
			ec() {
				console.log(this.ec)
			}
		},
		onReady: function() {
			if (!this.ec) {
				console.warn('组件需绑定 ec 变量，例：<ec-canvas id="mychart-dom-bar" ' +
					'canvas-id="mychart-bar" ec="{{ ec }}"></ec-canvas>');
				return;
			}
			if (!this.ec.lazyLoad) {
				this.init();
			}
		},
		methods: {
			init: function(callback) {
				const version = wx.version.version.split('.').map(n => parseInt(n, 10));
				const isValid = version[0] > 1 || (version[0] === 1 && version[1] > 9) ||
					(version[0] === 1 && version[1] === 9 && version[2] >= 91);
				if (!isValid) {
					console.error('微信基础库版本过低，需大于等于 1.9.91。' +
						'参见：https://github.com/ecomfe/echarts-for-weixin' +
						'#%E5%BE%AE%E4%BF%A1%E7%89%88%E6%9C%AC%E8%A6%81%E6%B1%82');
					return;
				}

				ctx = wx.createCanvasContext(this.canvasId, this);

				const canvas = new WxCanvas(ctx, this.canvasId);

				echarts.setCanvasCreator(() => {
					return canvas;
				});

				var query = wx.createSelectorQuery().in(this);
				query.select('.ec-canvas').boundingClientRect(res => {
					if (typeof callback === 'function') {
						this.chart = callback(canvas, res.width, res.height);
					} else if (this.ec && typeof this.ec.onInit === 'function') {
						this.chart = this.ec.onInit(canvas, res.width, res.height);
						console.log(this.ec)
					} else {
						this.$scope.triggerEvent('init', {
							canvas: canvas,
							width: res.width,
							height: res.height
						});
					}
				}).exec();
			},

			canvasToTempFilePath(opt) {
				if (!opt.canvasId) {
					opt.canvasId = this.canvasId;
				}

				ctx.draw(true, () => {
					wx.canvasToTempFilePath(opt, this);
				});
			},

			touchStart(e) {
				console.log(e)
				if (this.chart && e.touches.length > 0) {
					var touch = e.touches[0];
					var handler = this.chart.getZr().handler;
					handler.dispatch('mousedown', {
						zrX: touch.x,
						zrY: touch.y
					});
					handler.dispatch('mousemove', {
						zrX: touch.x,
						zrY: touch.y
					});
					handler.processGesture(wrapTouch(e), 'start');
				}
			},

			touchMove(e) {
				console.log(e)
				if (this.chart && e.touches.length > 0) {
					var touch = e.touches[0];
					var handler = this.chart.getZr().handler;
					handler.dispatch('mousemove', {
						zrX: touch.x,
						zrY: touch.y
					});
					handler.processGesture(wrapTouch(e), 'change');
				}
			},

			touchEnd(e) {
				if (this.chart) {
					const touch = e.changedTouches ? e.changedTouches[0] : {};
					var handler = this.chart.getZr().handler;
					handler.dispatch('mouseup', {
						zrX: touch.x,
						zrY: touch.y
					});
					handler.dispatch('click', {
						zrX: touch.x,
						zrY: touch.y
					});
					handler.processGesture(wrapTouch(e), 'end');
				}
			}
		}
	}

	function wrapTouch(event) {
		for (let i = 0; i < event.touches.length; ++i) {
			const touch = event.touches[i];
			touch.offsetX = touch.x;
			touch.offsetY = touch.y;
		}
		return event;
	}
</script>

<style>
	.ec-canvas {
		width: 100%;
		height: 100%;
	}
	
	.chart-box {
		width: 100%;
		height: 100%;
	}
</style>
