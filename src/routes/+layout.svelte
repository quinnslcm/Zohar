<script>
	import favicon from '$lib/assets/logomark.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href="/logo.png" />
	<meta name="theme-color" content="hsl(200, 20%, 5%)" />
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet" />
</svelte:head>

{@render children()}

<style>
	:global(:root) {
		/* Colors — 60-30-10-3-1 Palette */
		--c-base: hsl(200, 20%, 5%);
		--c-heading: hsl(200, 10%, 90%);
		--c-text: hsl(200, 10%, 85%);
		--c-accent: hsl(200, 50%, 50%);
		--c-surface: hsl(200, 10%, 18%);
		--c-mute: hsl(200, 6%, 28%);
		--c-text-muted: hsl(200, 8%, 55%);
		--c-accent-hover: hsl(200, 61%, 57%);

		/* Font Sizes — dramatic scale */
		--font-size-base: 17px;
		--font-size-h1: clamp(2.5rem, 5vw, 4.5rem);
		--font-size-h2: clamp(2rem, 4vw, 3rem);
		--font-size-h3: 22px;
		--font-size-small: 14px;

		/* Spacing — generous whitespace */
		--spacing-xs: 8px;
		--spacing-sm: 16px;
		--spacing-md: 24px;
		--spacing-lg: 32px;
		--spacing-xl: 40px;
		--spacing-2xl: 48px;
		--spacing-3xl: 96px;
		--spacing-4xl: 128px;
		--spacing-5xl: 160px;

		/* Layout — wider */
		--max-width-container: 1080px;
		--max-width-prose: 720px;

		/* Borders & Radii — flatter */
		--border-width: 1px;
		--border-width-thick: 2px;
		--radius-card: 2px;
		--radius-sm: 1px;

		/* Line Heights */
		--line-height-base: 1.7;
		--line-height-relaxed: 1.8;
		--line-height-tight: 1.05;
	}

	:global(*),
	:global(*::before),
	:global(*::after) {
		box-sizing: border-box;
		margin: 0;
		padding: 0;
	}

	:global(html) {
		scroll-behavior: smooth;
		-webkit-text-size-adjust: 100%;
	}

	:global(body) {
		font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
		background-color: var(--c-base);
		color: var(--c-text);
		line-height: var(--line-height-base);
		font-size: var(--font-size-base);
		letter-spacing: -0.01em;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}

	:global(a) {
		color: var(--c-accent);
		text-decoration: none;
		transition: opacity 0.2s ease;
	}

	:global(a:hover) {
		opacity: 0.7;
	}

	:global(:focus-visible) {
		outline: var(--border-width-thick) solid var(--c-accent);
		outline-offset: 3px;
	}

	:global(img) {
		max-width: 100%;
		display: block;
	}

	:global(.bg-light) {
		--c-base: hsl(200, 15%, 85%);
		--c-text: hsl(200, 15%, 15%);
		--c-heading: hsl(200, 15%, 10%);
		--c-mute: hsl(200, 10%, 72%);
		--c-surface: hsl(200, 15%, 80%);
		--c-accent: hsl(200, 80%, 30%);
		--c-accent-hover: hsl(200, 61%, 45%);
		--c-text-muted: hsl(200, 10%, 45%);
		background-color: var(--c-base);
		color: var(--c-text);
	}

</style>
