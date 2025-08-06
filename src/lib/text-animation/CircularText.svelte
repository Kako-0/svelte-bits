<script module lang="ts">
	interface CircularTextProps {
		text: string;
		spinDuration?: number;
		onHover?: 'slowDown' | 'speedUp' | 'pause' | 'goBonkers';
		className?: string;
	}
</script>

<script lang="ts">
	import gsap from 'gsap';

	let {
		text,
		spinDuration = 20,
		onHover = 'speedUp',
		className = ''
	}: CircularTextProps = $props();

	let containerRef: HTMLDivElement | null = null;
	let rotationTween: gsap.core.Tween = gsap.to(containerRef, {});

	const startRotation = (duration: number, scale = 1) => {
		if (!containerRef) return;

		rotationTween.kill();

		rotationTween = gsap.to(containerRef, {
			rotation: '+=360',
			duration,
			ease: 'none',
			repeat: -1,
			scale,
			transformOrigin: 'center center'
		});
	};

	$effect(() => {
		startRotation(spinDuration);
		return () => rotationTween.kill();
	});

	const handleHoverStart = () => {
		if (!containerRef || !onHover) return;

		let newDuration = spinDuration;
		let newScale = 1;

		switch (onHover) {
			case 'slowDown':
				newDuration = spinDuration * 2;
				break;
			case 'speedUp':
				newDuration = spinDuration / 4;
				break;
			case 'pause':
				rotationTween?.pause();
				return;
			case 'goBonkers':
				newDuration = spinDuration / 20;
				newScale = 0.8;
				break;
		}

		startRotation(newDuration, newScale);
	};

	const handleHoverEnd = () => {
		if (onHover === 'pause') {
			rotationTween?.resume();
		} else {
			startRotation(spinDuration, 1);
		}
	};

	const letters = Array.from(text);
</script>

<div
	bind:this={containerRef}
	class={`circular-text ${className}`}
	role="presentation"
	onmouseenter={handleHoverStart}
	onmouseleave={handleHoverEnd}
	style="display: inline-block;"
>
	{#each letters as letter, index (index)}
		{@const rotationDeg = (360 / letters.length) * index}
		{@const factor = Math.PI / letters.length}
		{@const x = factor * index}
		{@const y = factor * index}
		{@const transform = `rotateZ(${rotationDeg}deg) translate3d(${x}px, ${y}px, 0)`}
		<span style={`transform: ${transform};  WebkitTransform: ${transform}`}>
			{letter}
		</span>
	{/each}
</div>

<style>
	.circular-text {
		margin: 0 auto;
		border-radius: 50%;
		width: 200px;
		position: relative;
		height: 200px;
		font-weight: bold;
		font-weight: 900;
		text-align: center;
		cursor: pointer;
		transform-origin: 50% 50%;
		-webkit-transform-origin: 50% 50%;
	}

	.circular-text span {
		position: absolute;
		display: inline-block;
		left: 0;
		right: 0;
		top: 0;
		bottom: 0;
		font-size: 24px;
		transition: all 0.5s cubic-bezier(0, 0, 0, 1);
	}
</style>
