<svelte:head>
	<title>Outlook Safelink Parser 📧</title>
	<meta name="description" content="Find out where the Outlook Safelink would take you" />
</svelte:head>

<script>
	import '../app.css';
	
	let { children } = $props();
</script>

{@render children()}
