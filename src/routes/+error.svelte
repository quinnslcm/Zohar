<script>
    import { page } from '$app/stores';
</script>

<svelte:head>
    <title>404 | Zohar Global Security</title>
</svelte:head>

<div class="error-page">
    <div class="error-inner">
        <span class="error-code">{$page.status}</span>
        <h1>Area Restricted</h1>
        <p>The page you're looking for doesn't exist or has been moved.</p>
        <a href="/" class="error-link">&larr; Return to Secure Perimeter</a>
    </div>
</div>

<style>
    .error-page {
        min-height: 100vh;
        display: flex;
        align-items: center;
        justify-content: center;
        padding: var(--spacing-md);
    }

    .error-inner {
        text-align: center;
    }

    .error-code {
        display: block;
        font-size: 6rem;
        font-weight: 800;
        letter-spacing: -0.04em;
        color: hsla(200, 50%, 50%, 0.15);
        line-height: 1;
        margin-bottom: var(--spacing-sm);
    }

    h1 {
        font-size: var(--font-size-h2);
        font-weight: 700;
        color: var(--c-heading);
        margin-bottom: var(--spacing-sm);
    }

    p {
        color: var(--c-text-muted);
        margin-bottom: var(--spacing-xl);
    }

    .error-link {
        font-size: 13px;
        font-weight: 600;
        text-transform: uppercase;
        letter-spacing: 0.1em;
        color: var(--c-accent);
        text-decoration: none;
    }
</style>
