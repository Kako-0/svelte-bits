<script module lang="ts">
	interface SplitTextProps {
		text: string;
		className?: string;
		delay?: number;
		duration?: number;
		ease?: string | ((t: number) => number);
		splitType?: 'chars' | 'words' | 'lines' | 'words, chars';
		from?: gsap.TweenVars;
		to?: gsap.TweenVars;
		threshold?: number;
		rootMargin?: string;
		textAlign?: 'left' | 'right' | 'center' | 'justify' | 'initial' | 'inherit';
		onLetterAnimationComplete?: () => void;
	}
</script>

<script lang="ts">
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';
	import { SplitText as GSAPSplitText } from 'gsap/SplitText';

	gsap.registerPlugin(ScrollTrigger, GSAPSplitText);

	let {
		text,
		className = '',
		delay = 100,
		duration = 0.6,
		ease = 'power3.out',
		splitType = 'chars',
		from = { opacity: 0, y: 40 },
		to = { opacity: 1, y: 0 },
		threshold = 0.1,
		rootMargin = '-100px',
		textAlign = 'center',
		onLetterAnimationComplete
	}: SplitTextProps = $props();

	let ref: HTMLParagraphElement | null = null;
	let animationCompletedRef: boolean = false;
	let scrollTriggerRef: ScrollTrigger | null = null;

	$effect(() => {
		if (typeof window === 'undefined' || !ref || !text) return;

		const el = ref;

		animationCompletedRef = false;

		const absoluteLines = splitType === 'lines';
		if (absoluteLines) el.style.position = 'relative';

		let splitter: GSAPSplitText;
		try {
			splitter = new GSAPSplitText(el, {
				type: splitType,
				absolute: absoluteLines,
				linesClass: 'split-line'
			});
		} catch (error) {
			console.error('Failed to create SplitText:', error);
			return;
		}

		let targets: Element[];
		switch (splitType) {
			case 'lines':
				targets = splitter.lines;
				break;
			case 'words':
				targets = splitter.words;
				break;
			case 'chars':
				targets = splitter.chars;
				break;
			default:
				targets = splitter.chars;
		}

		if (!targets || targets.length === 0) {
			console.warn('No targets found for SplitText animation');
			splitter.revert();
			return;
		}

		targets.forEach((t) => {
			(t as HTMLElement).style.willChange = 'transform, opacity';
		});

		const startPct = (1 - threshold) * 100;
		const marginMatch = /^(-?\d+(?:\.\d+)?)(px|em|rem|%)?$/.exec(rootMargin);
		const marginValue = marginMatch ? parseFloat(marginMatch[1]) : 0;
		const marginUnit = marginMatch ? marginMatch[2] || 'px' : 'px';
		const sign =
			marginValue < 0 ? `-=${Math.abs(marginValue)}${marginUnit}` : `+=${marginValue}${marginUnit}`;
		const start = `top ${startPct}%${sign}`;

		const tl = gsap.timeline({
			scrollTrigger: {
				trigger: el,
				start,
				toggleActions: 'play none none none',
				once: true,
				onToggle: (self) => {
					scrollTriggerRef = self;
				}
			},
			smoothChildTiming: true,
			onComplete: () => {
				animationCompletedRef = true;
				gsap.set(targets, {
					...to,
					clearProps: 'willChange',
					immediateRender: true
				});
				onLetterAnimationComplete?.();
			}
		});

		tl.set(targets, { ...from, immediateRender: false, force3D: true });
		tl.to(targets, {
			...to,
			duration,
			ease,
			stagger: delay / 1000,
			force3D: true
		});

		return () => {
			tl.kill();
			if (scrollTriggerRef) {
				scrollTriggerRef.kill();
				scrollTriggerRef = null;
			}
			gsap.killTweensOf(targets);
			if (splitter) {
				splitter.revert();
			}
		};
	});
</script>

<p
	bind:this={ref}
	class={`split-parent ${className}`}
	style={`text-align: ${textAlign}; overflow: hidden; display: inline-block; white-space: normal; word-wrap: break-word;`}
>
	{text}
</p>
