<script>
	import { page } from '$app/stores';
</script>
<nav>
	<ul>
		<li class={$page?.url?.pathname === '/' ? 'active' : ''}><a href="/">Home</a></li>
		<li class={$page?.url?.pathname === '/task' ? 'active' : ''}><a href="/task">Tasks</a></li>
		<li class={$page?.url?.pathname === '/contact' ? 'active' : ''}>
			<a href="/contact">Contact</a>
		</li>
	</ul>
</nav>

<main>
	<slot />
</main>