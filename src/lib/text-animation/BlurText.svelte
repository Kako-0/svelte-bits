<script module lang="ts">
	type BlurTextProps = {
		text?: string;
		delay?: number;
		className?: string;
		animateBy?: 'words' | 'letters';
		direction?: 'top' | 'bottom';
		threshold?: number;
		rootMargin?: string;
		animationFrom?: Record<string, string | number>;
		animationTo?: Array<Record<string, string | number>>;
		easing?: (t: number) => number;
		onAnimationComplete?: () => void;
		stepDuration?: number;
	};
</script>

<script lang="ts">
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';
	import { SplitText as GSAPSplitText } from 'gsap/SplitText';

	gsap.registerPlugin(ScrollTrigger, GSAPSplitText);

	const buildKeyframes = (
		from: Record<string, string | number>,
		steps: Array<Record<string, string | number>>
	): Record<string, Array<string | number>> => {
		const keys = new Set<string>([...Object.keys(from), ...steps.flatMap((s) => Object.keys(s))]);

		const keyframes: Record<string, Array<string | number>> = {};
		keys.forEach((k) => {
			keyframes[k] = [from[k], ...steps.map((s) => s[k])];
		});
		return keyframes;
	};

	let {
		text = '',
		delay = 200,
		className = '',
		animateBy = 'words',
		direction = 'top',
		threshold = 0.1,
		rootMargin = '0px',
		animationFrom,
		animationTo,
		easing = (t) => t,
		onAnimationComplete,
		stepDuration = 0.35
	}: BlurTextProps = $props();

	const elements = animateBy === 'words' ? text.split(' ') : text.split('');
	let inView = $state<boolean>(false);
	let ref: HTMLDivElement | null = null;
	let spans: HTMLSpanElement[] = [];

	$effect(() => {
		if (!ref) return;

		const observer = new IntersectionObserver(
			([entry]) => {
				if (entry.isIntersecting) {
					inView = true;
					observer.unobserve(ref as Element);
					animateIn();
				}
			},
			{ threshold, rootMargin }
		);
		observer.observe(ref);
		return () => observer.disconnect();
	});

	let defaultFrom = $derived.by(() =>
		direction === 'top'
			? { filter: 'blur(10px)', opacity: 0, y: -50 }
			: { filter: 'blur(10px)', opacity: 0, y: 50 }
	);

	let defaultTo = $derived.by(() => [
		{
			filter: 'blur(5px)',
			opacity: 0.5,
			y: direction === 'top' ? 5 : -5
		},
		{ filter: 'blur(0px)', opacity: 1, y: 0 }
	]);

	let fromSnapshot = $derived(animationFrom ?? defaultFrom);
	let toSnapshots = $derived(animationTo ?? defaultTo);

	let stepCount = $derived(toSnapshots.length + 1);
	let totalDuration = $derived(stepDuration * (stepCount - 1));
	let times = $derived(
		Array.from({ length: stepCount }, (_, i) => (stepCount === 1 ? 0 : i / (stepCount - 1)))
	);

	function animateIn() {
		spans.forEach((el) => {
			gsap.set(el, fromSnapshot);
		});

		const tl = gsap.timeline({
			onComplete: onAnimationComplete
		});

		elements.forEach((_, index) => {
			const el = spans[index];

			const keyframes = [fromSnapshot, ...toSnapshots];
			const timesWithDuration = times.map((t) => t * totalDuration);

			keyframes.forEach((frame, i) => {
				tl.to(
					el,
					{
						...frame,
						duration: i === 0 ? 0 : stepDuration,
						ease: easing
					},
					(index * delay) / 1000 + timesWithDuration[i] // delay por índice + tempo relativo
				);
			});
		});
	}
</script>

<p bind:this={ref} class={`${className}`} style="display:flex; flex-wrap: wrap">
	{#each elements as segment, index}
		<span
			bind:this={spans[index]}
			style="display: inline-block; will-change: transform, filter, opacity;"
		>
			{segment === ' ' ? '\u00A0' : segment}
			{animateBy === 'words' && index < elements.length - 1 ? '\u00A0' : ''}
		</span>
	{/each}
</p>
