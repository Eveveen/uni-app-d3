<template>
	<!-- 监听变量 spath 的变化 -->
	<view style="map-container" :spath="spath" :change:spath="map.loadPath">
		<svg id="map">
		</svg>
	</view>
</template>

<script>
	import {
		pathData,
		detailData
	} from './data.js'
	export default {
		data() {
			return {
				spath: '',
			}
		},
		mounted() {
			// 修改 spath 的值
			this.spath = pathData;
		},
		methods: {
			/**
			 * 接收 map 中传回的数据
			 * @param {Object} id
			 */
			reciveMessage: function(id) {
				// 通过 id 匹配到对应的详细数据
				const name = detailData.find(item => item.id === id)?.name;
				uni.showToast({
					icon: 'none',
					title: name
				})
			}
		}
	}
</script>
<script module="map" lang="renderjs">
	import * as d3 from 'd3';

	export default {
		methods: {
			loadPath(newValue, oldValue, ownerInstance, instance) {
				// 监听 service 层数据变更
				// 初始化 绘图
				this.drawMap(newValue)
			},
			drawMap(list) {
				let pathList = [];
				list.forEach(item => {
					// 注：路径中不可以包含 换行符\n，M/L/A之后的空格都需要删除
					pathList.push({
						path: item.path,
						id: item.id,
						color: item.fill
					});
				})
				
				let svg = d3.select('#map')
					.append('svg')
					.attr('id', 'svgWrapper');

				// #map 为页面 svg 的 id
				const svgContainer = d3.select('#map').node();

				// 设置 放大缩小的倍数，这里设置的是 0.5-8 倍
				const zoom = d3.zoom()
					.scaleExtent([.5, 8])
					.on("zoom", zoomed);

				const g = svg.append("g").attr('id', 'gContainer');
				let g2 = g.append("g")
				var states = g2
					.selectAll("path")
					.data(pathList)
					.enter()
					.append("path")
					.attr("cursor", "pointer")
					.attr("d", function(d) {
						return d.path;
					})
					.style("fill", function(d) {
						return d.color || 'blue';
					})
					.on("click", clicked);


				// 下面这段代码是为了能够让绘制出来的图自适应屏幕大小，显示全部图形，如不需要可注释
				// ============== 自适应 开始 ====================
				let svgWrapper = d3.select('#svgWrapper').node();
				// 获取 svg 的 宽和高
				let svgWrapperWidth = Math.floor(svgWrapper.getBoundingClientRect().width);
				let svgWrapperHeight = Math.floor(svgWrapper.getBoundingClientRect().height);
				// 这里的 350 和 400 可以自行定义，暂未找到适用所有 svg 图形的比例，如有更好的方法可以提出来参考一下
				let scale = 350 / svgWrapperWidth;
				if (svgWrapperWidth < svgWrapperHeight) {
					scale = 400 / svgWrapperHeight
				}
				const newWidth = svgWrapperWidth * scale;
				const newHeight = svgWrapperHeight * scale;
				const x = svgWrapper.getBoundingClientRect().left * scale;
				const y = svgWrapper.getBoundingClientRect().top * scale;
				// 获取屏幕宽度， 屏幕高度
				const clientWidth = document.body.clientWidth;
				const clientHeight = document.body.clientHeight;
				g2.attr("transform",
					// 使左右、上下空白区域平分
					`translate(${-(x-(clientWidth-newWidth)/2)} ${(clientHeight-newHeight)/2-y}) scale(${scale})`)
				// ============== 自适应 结束 ====================	

				svg.call(zoom);

				// 上一次选中的信息
				let lastChecked = null;
				const that = this;

				// 点击操作
				function clicked(event, d) {
					event.stopPropagation();
					// 原来选中，则现在取消选中
					if (d.checked) {
						d3.select(this).transition().style("fill", lastChecked.color);
						lastChecked = null;
						d.checked = false;
					}
					// 原来未选中
					else {
						// 上次已经选中了其它块
						if (lastChecked) {
							// 恢复上次已选中的颜色
							d3.select(lastChecked.dom).transition().style("fill", lastChecked.color);
							lastChecked = null;
						}
						// 选中当前块 —— 设为红色
						d3.select(this).transition().style("fill", "red");
						// 存储信息
						lastChecked = d;
						// 存储 dom
						lastChecked.dom = this;
						d.checked = true;

						// 将选中的块的信息发送给父组件
						that.sendMsg(d.id)
					}
					d3.pointer(event, svg.node())
				}

				function zoomed(event) {
					const {
						transform
					} = event;
					g.attr("transform", transform);
					g.attr("stroke-width", 1 / transform.k);
				}
			},
			sendMsg(id) {
				// 向父组件传参
				this.$ownerInstance.callMethod('reciveMessage', id)
			},
		}
	}
</script>

<style>
	view {
		height: 100vh;
	}

	.map-container {
		width: 100%;
		height: 100%;
		min-height: 370px;
		max-height: 100vh;
	}

	svg {
		width: 100%;
		height: 100%;
		touch-action: none;
	}
</style>
