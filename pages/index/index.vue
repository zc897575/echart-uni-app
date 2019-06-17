<template>
	<view class="content">
		<ec-canvas id="mychart-dom-bar" canvas-id="mychart-bar"></ec-canvas>
	</view>
</template>

<script>
	import ecCanvas from "../../plugins/echarts/ec-canvas.vue"
	import * as echarts from '../../plugins/echarts/echarts.js';


	export default {
		data() {
			return {
				ec: {
					onInit: this.initChart
				}
			}
		},
		components: {
			ecCanvas
		},
		onLoad() {
			setTimeout(() => {
				this.show = true
			}, 1000)
			console.log(this.ec)
		},
		methods: {
			initChart(canvas, width, height) {
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
				console.log(chart)
				return chart;
			}
		}
	}
</script>

<style>
	.content {
		text-align: center;
		width: 100%;
		height: 400upx;
	}
</style>
