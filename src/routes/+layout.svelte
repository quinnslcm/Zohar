<script>
	import { onMount } from 'svelte';

	let { children } = $props();
	let loading = $state(true);
	let progress = $state(0);
	let statusText = $state('INITIALIZING');
	let fadeOut = $state(false);

	const statuses = [
		'INITIALIZING',
		'ESTABLISHING SECURE CONNECTION',
		'ENCRYPTING CHANNEL',
		'VERIFYING PROTOCOLS',
		'SCANNING PERIMETER',
		'SYSTEM READY'
	];

	onMount(() => {
		let frame;
		const start = Date.now();
		const duration = 3500;

		function tick() {
			const elapsed = Date.now() - start;
			const t = Math.min(elapsed / duration, 1);

			// Ease-out progress
			progress = Math.round(t * t * (3 - 2 * t) * 100);

			// Cycle status text
			const idx = Math.min(Math.floor(t * statuses.length), statuses.length - 1);
			statusText = statuses[idx];

			if (t < 1) {
				frame = requestAnimationFrame(tick);
			} else {
				fadeOut = true;
				setTimeout(() => { loading = false; }, 500);
			}
		}

		frame = requestAnimationFrame(tick);
		return () => cancelAnimationFrame(frame);
	});

	function initGridOverlay(canvasEl) {
		const ctx = canvasEl.getContext("2d");
		let animId;
		let mx = -9999, my = -9999;
		const hexChars = "0123456789ABCDEF";
		const cellSize = 40;

		function resize() {
			canvasEl.width = window.innerWidth;
			canvasEl.height = window.innerHeight;
		}

		resize();
		window.addEventListener("resize", resize);

		window.addEventListener("mousemove", (e) => {
			mx = e.clientX;
			my = e.clientY;
		});

		function draw() {
			ctx.clearRect(0, 0, canvasEl.width, canvasEl.height);

			const radius = 300;

			// Grid lines near cursor
			ctx.lineWidth = 0.5;
			const startCol = Math.floor((mx - radius) / cellSize);
			const endCol = Math.ceil((mx + radius) / cellSize);
			const startRow = Math.floor((my - radius) / cellSize);
			const endRow = Math.ceil((my + radius) / cellSize);

			for (let col = startCol; col <= endCol; col++) {
				const x = col * cellSize;
				const dx = Math.abs(x - mx);
				if (dx > radius) continue;
				const fade = 1 - dx / radius;
				ctx.strokeStyle = `hsla(200, 50%, 45%, ${fade * fade * 0.06})`;
				ctx.beginPath();
				ctx.moveTo(x, Math.max(0, my - radius));
				ctx.lineTo(x, Math.min(canvasEl.height, my + radius));
				ctx.stroke();
			}

			for (let row = startRow; row <= endRow; row++) {
				const y = row * cellSize;
				const dy = Math.abs(y - my);
				if (dy > radius) continue;
				const fade = 1 - dy / radius;
				ctx.strokeStyle = `hsla(200, 50%, 45%, ${fade * fade * 0.06})`;
				ctx.beginPath();
				ctx.moveTo(Math.max(0, mx - radius), y);
				ctx.lineTo(Math.min(canvasEl.width, mx + radius), y);
				ctx.stroke();
			}

			// Crosshairs at intersections + hex values
			ctx.font = "7px monospace";
			for (let col = startCol; col <= endCol; col++) {
				for (let row = startRow; row <= endRow; row++) {
					const x = col * cellSize;
					const y = row * cellSize;
					const dx = x - mx;
					const dy = y - my;
					const dist = Math.sqrt(dx * dx + dy * dy);
					if (dist > radius) continue;

					const falloff = 1 - dist / radius;
					const alpha = falloff * falloff * 0.12;

					// Small crosshair
					ctx.strokeStyle = `hsla(200, 55%, 50%, ${alpha})`;
					ctx.lineWidth = 0.5;
					ctx.beginPath();
					ctx.moveTo(x - 3, y); ctx.lineTo(x + 3, y);
					ctx.moveTo(x, y - 3); ctx.lineTo(x, y + 3);
					ctx.stroke();

					// Hex value near intersection
					if (falloff > 0.3) {
						const h1 = hexChars[Math.floor(Math.random() * 16)];
						const h2 = hexChars[Math.floor(Math.random() * 16)];
						ctx.fillStyle = `hsla(200, 50%, 50%, ${alpha * 0.5})`;
						ctx.fillText(h1 + h2, x + 5, y - 3);
					}
				}
			}

			animId = requestAnimationFrame(draw);
		}

		draw();

		return {
			destroy() {
				cancelAnimationFrame(animId);
				window.removeEventListener("resize", resize);
			}
		};
	}
</script>

<svelte:head>
	<link rel="icon" type="image/png" sizes="64x64" href="/favicon-64.png" />
	<link rel="icon" type="image/png" sizes="32x32" href="/favicon.png" />
	<link rel="apple-touch-icon" href="/apple-touch-icon.png" />
	<meta name="theme-color" content="hsl(200, 20%, 5%)" />
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet" />
</svelte:head>

{#if loading}
<div class="loader" class:fade-out={fadeOut}>
	<div class="loader-inner">
		<!-- Corner brackets -->
		<div class="loader-brackets">
			<span class="br br-tl"></span>
			<span class="br br-tr"></span>
			<span class="br br-bl"></span>
			<span class="br br-br"></span>
		</div>

		<!-- Logo -->
		<img src="/logo.png" alt="" class="loader-logo" />

		<!-- Brand -->
		<p class="loader-brand">ZOHAR GLOBAL SECURITY</p>

		<!-- Progress bar -->
		<div class="loader-bar-track">
			<div class="loader-bar-fill" style="width: {progress}%"></div>
		</div>

		<!-- Status line -->
		<div class="loader-status">
			<span class="loader-status-text">{statusText}</span>
			<span class="loader-pct">{progress}%</span>
		</div>

		<!-- Data readout -->
		<div class="loader-data">
			<div class="loader-data-row">
				<span class="loader-data-label">CONN</span>
				<span class="loader-data-val">AES-256-GCM</span>
			</div>
			<div class="loader-data-row">
				<span class="loader-data-label">NODE</span>
				<span class="loader-data-val">US-EAST-1 // ACTIVE</span>
			</div>
			<div class="loader-data-row">
				<span class="loader-data-label">CERT</span>
				<span class="loader-data-val">SHA-384 // VERIFIED</span>
			</div>
		</div>
	</div>

	<!-- Grid bg -->
	<div class="loader-grid"></div>
</div>
{/if}

<canvas use:initGridOverlay class="grid-overlay" aria-hidden="true"></canvas>
{@render children()}

<style>
	.grid-overlay {
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		pointer-events: none;
		z-index: 9999;
	}

	@media (max-width: 768px) {
		.grid-overlay {
			display: none;
		}
	}

	/* ── Loading Screen ──────────────────────── */
	.loader {
		position: fixed;
		inset: 0;
		z-index: 99999;
		background: hsl(200, 20%, 5%);
		display: flex;
		align-items: center;
		justify-content: center;
		transition: opacity 0.5s ease;
	}

	.loader.fade-out {
		opacity: 0;
	}

	.loader-inner {
		position: relative;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 20px;
		padding: 60px;
	}

	/* Corner brackets — lock-on animation */
	.loader-brackets {
		position: absolute;
		inset: -20px;
		animation: brackets-lock 3s ease-out forwards;
	}

	@keyframes brackets-lock {
		0% { inset: -40px; opacity: 0; }
		15% { opacity: 1; }
		100% { inset: 0px; }
	}

	.br {
		position: absolute;
		width: 20px;
		height: 20px;
		border-color: hsla(200, 50%, 50%, 0.3);
		border-style: solid;
		transition: border-color 0.3s;
	}

	.loader.fade-out .br {
		border-color: hsla(200, 50%, 50%, 0.6);
	}

	.br-tl { top: 0; left: 0; border-width: 1px 0 0 1px; }
	.br-tr { top: 0; right: 0; border-width: 1px 1px 0 0; }
	.br-bl { bottom: 0; left: 0; border-width: 0 0 1px 1px; }
	.br-br { bottom: 0; right: 0; border-width: 0 1px 1px 0; }

	.loader-logo {
		width: 48px;
		height: auto;
		animation: pulse-glow 1.5s ease-in-out infinite;
	}

	@keyframes pulse-glow {
		0%, 100% { opacity: 0.6; filter: brightness(1); }
		50% { opacity: 1; filter: brightness(1.3); }
	}

	.loader-brand {
		font-family: 'Inter', sans-serif;
		font-size: 11px;
		font-weight: 500;
		letter-spacing: 0.25em;
		color: hsl(200, 50%, 50%);
		text-align: center;
	}

	.loader-bar-track {
		width: 200px;
		height: 2px;
		background: hsla(200, 20%, 20%, 0.5);
		overflow: hidden;
	}

	.loader-bar-fill {
		height: 100%;
		background: hsl(200, 50%, 50%);
		transition: width 0.1s linear;
	}

	.loader-status {
		display: flex;
		justify-content: space-between;
		width: 200px;
		font-family: monospace;
		font-size: 9px;
		color: hsla(200, 40%, 50%, 0.6);
		letter-spacing: 0.05em;
	}

	.loader-pct {
		color: hsl(200, 50%, 50%);
	}

	.loader-data {
		margin-top: 12px;
		display: flex;
		flex-direction: column;
		gap: 6px;
		width: 200px;
	}

	.loader-data-row {
		display: flex;
		justify-content: space-between;
		font-family: monospace;
		font-size: 8px;
		letter-spacing: 0.05em;
		animation: data-fade 3.5s ease forwards;
		opacity: 0;
	}

	.loader-data-row:nth-child(1) { animation-delay: 0.8s; }
	.loader-data-row:nth-child(2) { animation-delay: 1.4s; }
	.loader-data-row:nth-child(3) { animation-delay: 2.0s; }

	@keyframes data-fade {
		0% { opacity: 0; transform: translateY(4px); }
		15% { opacity: 1; transform: translateY(0); }
		100% { opacity: 1; transform: translateY(0); }
	}

	.loader-data-label {
		color: hsla(200, 40%, 45%, 0.5);
	}

	.loader-data-val {
		color: hsla(200, 50%, 55%, 0.4);
	}

	.loader-grid {
		position: absolute;
		inset: 0;
		opacity: 0.03;
		background-image:
			linear-gradient(hsla(200, 50%, 50%, 0.5) 1px, transparent 1px),
			linear-gradient(90deg, hsla(200, 50%, 50%, 0.5) 1px, transparent 1px);
		background-size: 40px 40px;
		pointer-events: none;
	}

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
