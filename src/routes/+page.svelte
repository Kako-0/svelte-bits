<script lang="ts">
	import BlurText from '$lib/text-animation/BlurText.svelte';
	import CircularText from '$lib/text-animation/CircularText.svelte';
	import CurvedLoop from '$lib/text-animation/CurvedLoop.svelte';
	import SplitText from '$lib/text-animation/SplitText.svelte';
</script>

<h1 class="split-text">Welcome to SvelteKit</h1>

<SplitText
	text="Welcome to SvelteKit"
	className="big-text"
	delay={250}
	duration={0.6}
	ease="power3.out"
	splitType="chars"
	from={{ opacity: 0, y: 40 }}
	to={{ opacity: 1, y: 0 }}
	threshold={0.1}
	rootMargin="-100px"
	textAlign="center"
/>

<BlurText
	text="Isn't this so cool?!"
	delay={150}
	animateBy="words"
	direction="top"
	className="big-text"
/>

<CircularText text="SVELTE*BITS*COMPONENTS*" onHover="slowDown" spinDuration={20} />

<CurvedLoop
	marqueeText="Be ✦ Creative ✦ With ✦ Svelte ✦ Bits ✦"
	speed={3}
	curveAmount={500}
	direction="right"
	interactive={true}
	className="custom-text-style"
/>

<style>
	:global(body) {
		background-color: #e0e0e0;
	}

	:global(.big-text) {
		text-align: center;
		font-weight: 600;
		font-size: 4rem;
	}
</style>
