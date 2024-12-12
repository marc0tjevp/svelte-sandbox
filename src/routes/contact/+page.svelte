<script lang="ts">
	import { onMount } from 'svelte';

	// Define the Task type based on Flowable's API response
	interface Task {
		id: string;
		name: string;
		assignee: string | null;
		status: string | null;
	}

	let tasks: Task[] = [];
	let error: string | null = null;

	// API Configuration
	const AUTH_TOKEN = btoa('admin:test'); // Replace with actual credentials

	onMount(async () => {
		try {
			// Use the proxied API path
			const response = await fetch('/api/runtime/tasks', {
				method: 'GET',
				headers: {
					'Content-Type': 'application/json',
					Authorization: `Basic ${AUTH_TOKEN}`,
				},
			});

			if (!response.ok) {
				throw new Error(`Error: ${response.status} ${response.statusText}`);
			}

			// Type the response properly
			const data: { data: Task[] } = await response.json();
			tasks = data.data;
		} catch (err: unknown) {
			// Ensure error handling is typed
			error = err instanceof Error ? err.message : 'Unknown error occurred';
		}
	});
</script>

<div>
	{#if error}
		<p>Error: {error}</p>
	{:else if tasks.length === 0}
		<p>No tasks available.</p>
	{:else}
		<ul>
			{#each tasks as task}
				<li>
					<strong>{task.name}</strong> - {task.assignee ? task.assignee : 'Unassigned'}
				</li>
			{/each}
		</ul>
	{/if}
</div>
