<template>
	<div class="image-container" ref="imageContainerRef">
		<img
			class="img"
			src="../../assets/loginBackground.jpg"
			alt="Vue logo"
			ref="imgRef"
			@dragstart.prevent
			:onload="handleImgOnload"
		/>
		<div
			class="minimap-container"
			:style="minimapContainerStyle"
			ref="minimapContainerRef"
		>
			<div class="minimap-bg" :style="minimapBgStyle"></div>
			<div
				class="minimap-img"
				:style="minimapImgStyle"
				ref="minimapImgRef"
				@dragstart.prevent
			></div>
		</div>
	</div>
</template>

<script setup>
import { ref, onMounted, nextTick, onUnmounted } from "vue";

const imgRef = ref();
const imageContainerRef = ref();
const minimapContainerStyle = ref({});
const minimapBgStyle = ref({});
const minimapImgStyle = ref({});
const minimapScale = ref(0.2);
const imgOffsetX = ref(0);
const imgOffsetY = ref(0);
const scale = ref(1);
const scaleStep = ref(0.1);
const isScaling = ref(false);
const imgAspectRatio = ref(1);
const imgContainerAspectRatio = ref(1);
const originPaddingLeft = ref(0);
const originPaddingTop = ref(0);
const minimapContainerPaddingLeft = ref(0);
const minimapContainerPaddingTop = ref(0);
const minimapContainerRef = ref();
const minimapImgRef = ref();
const minimapImgOffsetX = ref(0);
const minimapImgOffsetY = ref(0);
const minimapMousedownX = ref(0);
const minimapMousedownY = ref(0);
const minimapMousedownLeftNum = ref(0);
const minimapMousedownTopNum = ref(0);

// 样式px转换数字
const pxToNumber = (px) => {
	return Number(px.slice(0, -2));
};
// 图片加载完成事件
const handleImgOnload = () => {
	// 图片宽高比
	imgAspectRatio.value = imgRef.value.clientWidth / imgRef.value.clientHeight;
	// 图片容器宽高比
	imgContainerAspectRatio.value =
		imageContainerRef.value.clientWidth / imageContainerRef.value.clientHeight;
	// 小地图容器间距
	minimapContainerPaddingLeft.value = imageContainerRef.value.clientHeight / 10;
	minimapContainerPaddingTop.value = imageContainerRef.value.clientWidth / 10;
	// 小地图样式
	minimapContainerStyle.value = {
		transform: `scale(${minimapScale.value})`,
		transformOrigin: "right bottom",
		width: `${
			imageContainerRef.value.clientWidth +
			minimapContainerPaddingLeft.value * 2
		}px`,
		height: `${
			imageContainerRef.value.clientHeight +
			minimapContainerPaddingTop.value * 2
		}px`,
	};
	minimapBgStyle.value = {
		width: `${imageContainerRef.value.clientWidth}px`,
		height: `${imageContainerRef.value.clientHeight}px`,
		left: `${minimapContainerPaddingLeft.value}px`,
		top: `${minimapContainerPaddingTop.value}px`,
	};
	minimapImgStyle.value = {
		width: `${imgRef.value.clientWidth}px`,
		height: `${imgRef.value.clientHeight}px`,
		top: `${imgRef.value.offsetTop + minimapContainerPaddingTop.value}px`,
		left: `${imgRef.value.offsetLeft + minimapContainerPaddingLeft.value}px`,
	};
	imgRef.value.style.left = `${imgRef.value.offsetLeft}px`;
	imgRef.value.style.top = `${imgRef.value.offsetTop}px`;
	imgRef.value.style.width = `${imgRef.value.clientWidth}px`;
	imgRef.value.style.height = `${imgRef.value.clientHeight}px`;
	originPaddingLeft.value =
		pxToNumber(minimapContainerStyle.value.width) -
		pxToNumber(minimapBgStyle.value.width);
	originPaddingTop.value =
		pxToNumber(minimapContainerStyle.value.height) -
		pxToNumber(minimapBgStyle.value.height);
};
// 图片鼠标按下事件
const handleImgMousedown = (e) => {
	document.addEventListener("mousemove", handleImgMousemove);
	imgOffsetX.value = e.clientX - imgRef.value.offsetLeft;
	imgOffsetY.value = e.clientY - imgRef.value.offsetTop;
};
// 图片鼠标移动事件
const handleImgMousemove = (e) => {
	e.preventDefault();
	imgRef.value.style.left = `${e.clientX - imgOffsetX.value}px`;
	imgRef.value.style.top = `${e.clientY - imgOffsetY.value}px`;
};
// 图片鼠标抬起事件
const handleImgMouseup = () => {
	document.removeEventListener("mousemove", handleImgMousemove);
};
// 监听图片样式变化
const observer = new MutationObserver((mutations) => {
	mutations.forEach((mutation) => {
		if (mutation.attributeName === "style") {
			const { left, top, transform, width, height } =
				mutation.target.style || {};
			const leftNum = pxToNumber(left);
			const topNum = pxToNumber(top);
			const widthNum = pxToNumber(width);
			const heightNum = pxToNumber(height);
			// 同图片缩放
			minimapImgStyle.value.transform = transform;
			// minimapBg根据minimapImg超出屏幕部分，宽度和偏移量做相应计算，使小地图minimapImg不超出边界
			if (leftNum < 0) {
				// 左侧超出边界
				minimapImgStyle.value.left = `${minimapContainerPaddingLeft.value}px`;
				minimapBgStyle.value.width = `${
					imageContainerRef.value.clientWidth + leftNum
				}px`;
				minimapBgStyle.value.left = `${
					minimapContainerPaddingLeft.value - leftNum
				}px`;
			} else if (leftNum + widthNum <= imageContainerRef.value.clientWidth) {
				// 在边界内
				minimapImgStyle.value.left = `${
					leftNum + minimapContainerPaddingLeft.value
				}px`;
				minimapBgStyle.value.width = `${imageContainerRef.value.clientWidth}px`;
				minimapBgStyle.value.left = `${minimapContainerPaddingLeft.value}px`;
			} else {
				// 右侧超出边界
				minimapBgStyle.value.width = `${
					imageContainerRef.value.clientWidth -
					(leftNum + widthNum - imageContainerRef.value.clientWidth)
				}px`;
			}
			if (topNum < 0) {
				// 上方超出边界
				minimapImgStyle.value.top = `${minimapContainerPaddingTop.value}px`;
				minimapBgStyle.value.height = `${
					imageContainerRef.value.clientHeight + topNum
				}px`;
				minimapBgStyle.value.top = `${
					minimapContainerPaddingTop.value - topNum
				}px`;
			} else if (topNum + heightNum <= imageContainerRef.value.clientHeight) {
				// 边界内
				minimapImgStyle.value.top = `${
					topNum + minimapContainerPaddingTop.value
				}px`;
				minimapBgStyle.value.height = `${imageContainerRef.value.clientHeight}px`;
				minimapBgStyle.value.top = `${minimapContainerPaddingTop.value}px`;
			} else {
				// 下方超出边界
				minimapBgStyle.value.height = `${
					imageContainerRef.value.clientHeight -
					(topNum + heightNum - imageContainerRef.value.clientHeight)
				}px`;
			}
		}
	});
});
// 图片鼠标滚轮事件
const handleImgWheel = (e) => {
	if (e.deltaY < 0) {
		scale.value += scaleStep.value;
	} else {
		scale.value -= scaleStep.value;
	}
	// 最大3，最小0.5
	scale.value = Math.min(Math.max(0.5, scale.value), 3);
	imgRef.value.style.transform = `scale(${scale.value})`;
	if (!isScaling.value) {
		isScaling.value = true;
		requestAnimationFrame(scaleImage);
	}
};
// 缩放图片
const scaleImage = () => {
	imgRef.value.style.transform = `scale(${scale.value})`;
	isScaling.value = false;
};
// minimap鼠标按下事件
const handleMinimapImgMousedown = (e) => {
	e.stopPropagation();
	document.addEventListener("mousemove", handleMinimapImgMousemove);
	minimapImgOffsetX.value = e.clientX - minimapImgRef.value.offsetLeft;
	minimapImgOffsetY.value = e.clientY - minimapImgRef.value.offsetTop;
	// 记录鼠标点击时，鼠标位置及minimapImg初始位置
	minimapMousedownX.value = e.clientX;
	minimapMousedownY.value = e.clientY;
	minimapMousedownLeftNum.value = pxToNumber(minimapImgStyle.value.left);
	minimapMousedownTopNum.value = pxToNumber(minimapImgStyle.value.top);
};
// minimap鼠标移动事件
const handleMinimapImgMousemove = (e) => {
	e.stopPropagation();
	e.preventDefault();
	const leftNum =
		minimapMousedownLeftNum.value +
		(e.clientX - minimapMousedownX.value) / minimapScale.value;
	const topNum =
		minimapMousedownTopNum.value +
		(e.clientY - minimapMousedownY.value) / minimapScale.value;
	const miniLeftNum = pxToNumber(minimapBgStyle.value.left);
	const miniTopNum = pxToNumber(minimapBgStyle.value.top);
	// 由于minimap是缩放0.2显示，所以移动偏移量需要以0.2还原
	// 限制边界区域，不允许超出
	minimapImgStyle.value.left = `${
		leftNum < miniLeftNum ? miniLeftNum : leftNum
	}px`;
	minimapImgStyle.value.top = `${topNum < miniTopNum ? miniTopNum : topNum}px`;
};
// minimap鼠标抬起事件
const handleMinimapImgMouseup = (e) => {
	e.stopPropagation();
	document.removeEventListener("mousemove", handleMinimapImgMousemove);
};

onMounted(() => {
	nextTick(() => {
		document.addEventListener("mousedown", handleImgMousedown);
		document.addEventListener("mouseup", handleImgMouseup);
		observer.observe(imgRef.value, { attributes: true });
		imgRef.value.addEventListener("wheel", handleImgWheel);
		minimapContainerRef.value.addEventListener(
			"mousedown",
			handleMinimapImgMousedown
		);
		document.addEventListener("mouseup", handleMinimapImgMouseup);
	});
});

onUnmounted(() => {
	document.removeEventListener("mousedown", handleImgMousedown);
	document.removeEventListener("mouseup", handleImgMouseup);
	observer?.unobserve?.(imgRef.value);
	imgRef.value.removeEventListener("wheel", handleImgWheel);
	minimapContainerRef.value.removeEventListener(
		"mousedown",
		handleMinimapImgMousedown
	);
	document.removeEventListener("mouseup", handleMinimapImgMouseup);
});
</script>

<style scoped lang="scss">
@use "./index.scss";
</style>
