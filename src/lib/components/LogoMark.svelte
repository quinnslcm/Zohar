<!-- SVG paths sourced from src/lib/assets/logomark.svg — keep in sync -->
<script>
	/** @type {{ size?: number, color?: string, class?: string }} */
	let { size = 40, color = 'hsl(200, 50%, 50%)', class: className = '' } = $props();
</script>

<svg
	width={size}
	height={size}
	viewBox="0 0 800 800"
	xmlns="http://www.w3.org/2000/svg"
	class={className}
	aria-hidden="true"
>
	<g fill={color}>
		<path d="M0,180v422.4c0,57.36,39.04,107.36,94.68,121.27l305.32,76.33,305.32-76.33c55.64-13.91,94.68-63.91,94.68-121.27V180H0ZM650,517.5v82.5H141.8c-1.67,0-2.44-2.33-1.16-3.53l290.39-242.97c1.38-1.22.52-3.5-1.32-3.5h-229.71v-70h415.51c4.2,0,6.1,5.87,2.86,8.85l-249.5,225.17c-1.36,1.23-.49,3.48,1.34,3.48h279.8Z"/>
		<polygon points="0 0 0 120 800 120 800 0 600 50 400 0 200 50 0 0"/>
	</g>
</svg>
