<template>
	<NodeViewWrapper
		class="image-resizer"
		:class="{
			'ProseMirror-selectednode':
				props.selected && props.editor.isEditable,
			'image-err': isImgErr,
			'inline': options.inline,
		}">
		<img
			ref="imgRef"
			v-if="isEnableLazyload"
			:data-src="attrs.src"
			:alt="attrs.alt"
			:title="attrs.title"
			:class="{ maxw100: !isLoaded }"
			:style="{
				width: attrs.width,
				transform: `rotate(${attrs.rotate}deg) ${
					translate[attrs.rotate]
				}`,
			}"
			@load="handleImgLoad"
			@error="handleImgError" />
		<img
			ref="imgRef"
			v-else
			:src="attrs.src"
			:alt="attrs.alt"
			:title="attrs.title"
			:class="{ maxw100: !isLoaded }"
			:style="{
				width: attrs.width,
				transform: `rotate(${attrs.rotate}deg) ${
					translate[attrs.rotate]
				}`,
			}"
			@load="handleImgLoad"
			@error="handleImgError" />

		<ImageResizeComponent
			v-if="isLoaded && options.resize"
			:imgRef="imgRef" />

		<ImageRotateComponent
			v-if="isLoaded && options.rotate"
			:imgRef="imgRef" />

		<div
			class="image-err-extra"
			ref="extRef"></div>
	</NodeViewWrapper>
</template>

<script setup>
	import { computed, onMounted, provide, ref } from 'vue';
	import { NodeViewWrapper } from '@tiptap/vue-3';
	import ImageResizeComponent from './ImageResizeComponent.vue';
	import ImageRotateComponent from './ImageRotateComponent.vue';

	const props = defineProps({
		editor: { type: Object, required: true },
		node: { type: Object, required: true },
		extension: { type: Object, required: true },
	});
	provide('tiptapProps', props);

	const attrs = computed(() => props.node.attrs);
	const options = computed(() => props.extension.options);
	const imgRef = ref(null);
	const translate = {
		'0': 'translate(0,0)',
		'-90': 'translate(-100%,0)',
		'-180': 'translate(-100%,-100%)',
		'-270': 'translate(0,-100%)',
	};

	const isLoaded = ref(false);
	function handleImgLoad(e) {
		isLoaded.value = true;
		const imgW = e.target.clientWidth;
		let maxW = props.editor.view.dom.clientWidth;
	   if ( imgW >= maxW )
	   {
	    console.log('updateAttributes in ImageWithTool': updateAttributes);
			props.updateAttributes({ width: `${maxW}px` });
		}
	}

	// #region
	const extRef = ref(null);
	onMounted(() => options.value.onExtraCreated(extRef.value, imgRef.value));
	// #endregion

	// #region
	const isImgErr = ref(false);
	function handleImgError() {
		isImgErr.value = true;
		options.value.onError(extRef.value);
	}
	// #endregion

	// #region
	const isEnableLazyload = !!IntersectionObserver;
	onMounted(() => {
		if (!isEnableLazyload) return;
		let observerImg = new IntersectionObserver((entries, observer) => {
			entries.forEach((entry) => {
				if (entry.isIntersecting !== false) {
					entry.target.src = entry.target.dataset.src;
					observer.unobserve(entry.target);
				}
			});
		});
		observerImg.observe(imgRef.value);
	});
	// #endregion
</script>

<style lang="scss">
	.image-resizer {
		max-width: 100%;
		position: relative;
		width: fit-content;

		&.inline {
			display: inline-block;
		}

		img {
			transform-origin: left top;
		}

		.maxw100 {
			width: 100%;
		}
	}
</style>
