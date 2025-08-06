<script module lang="ts">
	interface CurvedLoopProps {
		marqueeText?: string;
		speed?: number;
		className?: string;
		curveAmount?: number;
		direction?: 'left' | 'right';
		interactive?: boolean;
	}
</script>

<script lang="ts">
	let {
		marqueeText = '',
		speed = 2,
		className,
		curveAmount = 400,
		direction = 'left',
		interactive = true
	}: CurvedLoopProps = $props();

	let text = $derived.by(() => {
		const hasTrailing = /\s|\u00A0$/.test(marqueeText);
		return (hasTrailing ? marqueeText.replace(/\s+$/, '') : marqueeText) + '\u00A0';
	});

	let measureRef: SVGTextElement | null = null;
	let textPathRef: SVGTextPathElement | null = $state(null);
	let pathRef: SVGPathElement | null = null;
	let spacing = $state(0);
	let offset = $state(0);
	const uid = $props.id();
	let pathId = $derived(`curve-${uid}`);
	let pathD = $derived(`M-100,40 Q500,${40 + curveAmount} 1540,40`);

	let dragRef = false;
	let lastXRef = 0;
	let dirRef: 'left' | 'right' = direction;
	let velRef = 0;

	let textLength = $derived(spacing);
	let totalText = $derived(
		textLength
			? Array(Math.ceil(1800 / textLength) + 2)
					.fill(text)
					.join('')
			: text
	);
	let ready = $derived(spacing > 0);

	$effect(() => {
		let isChanged = text && className;
		if (measureRef) spacing = measureRef.getComputedTextLength();
	});

	$effect(() => {
		if (!spacing || !ready) return;
		let frame = 0;
		const step = () => {
			if (!dragRef && textPathRef) {
				const delta = dirRef === 'right' ? speed : -speed;
				const currentOffset = parseFloat(textPathRef.getAttribute('startOffset') || '0');
				let newOffset = currentOffset + delta;

				const wrapPoint = spacing;
				if (newOffset <= -wrapPoint) newOffset += wrapPoint;
				if (newOffset >= wrapPoint) newOffset -= wrapPoint;

				textPathRef.setAttribute('startOffset', newOffset + 'px');
				offset = newOffset;
			}
			frame = requestAnimationFrame(step);
		};
		step();
		return () => cancelAnimationFrame(frame);
	});

	const onPointerDown = (e: PointerEvent) => {
		if (!interactive) return;
		dragRef = true;
		lastXRef = e.clientX;
		velRef = 0;
		(e.target as HTMLElement).setPointerCapture(e.pointerId);
	};

	const onPointerMove = (e: PointerEvent) => {
		if (!interactive || !dragRef || !textPathRef) return;
		const dx = e.clientX - lastXRef;
		lastXRef = e.clientX;
		velRef = dx;

		const currentOffset = parseFloat(textPathRef.getAttribute('startOffset') || '0');
		let newOffset = currentOffset + dx;

		const wrapPoint = spacing;
		if (newOffset <= -wrapPoint) newOffset += wrapPoint;
		if (newOffset >= wrapPoint) newOffset -= wrapPoint;

		textPathRef.setAttribute('startOffset', newOffset + 'px');
		offset = newOffset;
	};

	const endDrag = () => {
		if (!interactive) return;
		dragRef = false;
		dirRef = velRef > 0 ? 'right' : 'left';
	};

	const cursorStyle = interactive ? (dragRef ? 'grabbing' : 'grab') : 'auto';
</script>

<div
	class="curved-loop-jacket"
	style={`visibility: ${ready ? 'visible' : 'hidden'}; cursor: ${cursorStyle};`}
	onpointerdown={onPointerDown}
	onpointermove={onPointerMove}
	onpointerup={endDrag}
	onpointerleave={endDrag}
>
	<svg class="curved-loop-svg" viewBox="0 0 1440 120">
		<text
			bind:this={measureRef}
			xml:space="preserve"
			style="visibility: 'hidden'; opacity: 0; pointer-events: 'none';"
		>
			{text}
		</text>
		<defs>
			<path bind:this={pathRef} id={pathId} d={pathD} fill="none" stroke="transparent" />
		</defs>
		{#if ready}
			<text font-weight="bold" xml:space="preserve" class={className}>
				<textPath
					bind:this={textPathRef}
					href={`#${pathId}`}
					startOffset={offset + 'px'}
					xml:space="preserve"
				>
					{totalText}
				</textPath>
			</text>
		{/if}
	</svg>
</div>

<style>
	.curved-loop-jacket {
		min-height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		width: 100%;
	}

	.curved-loop-svg {
		user-select: none;
		width: 100%;
		aspect-ratio: 100 / 12;
		overflow: visible;
		display: block;
		font-size: 6rem;
		fill: #222;
		user-select: none;
		-moz-user-select: none;
		-webkit-user-select: none;
		font-weight: 700;
		text-transform: uppercase;
		line-height: 1;
	}
</style>
