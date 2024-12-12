<script lang="ts">
	import { onMount } from 'svelte';

	// Task field structure
	type TaskField = {
		label: string;
		value: string | number;
	};

	// Task structure
	type Task = {
		id: number;
		title: string;
		description: string;
		assignedTo: string;
		creationDate: string;
		fields: TaskField[];
	};

	let tasks: Task[] = []; // Tasks fetched from the API
	let filteredTasks: Task[] = []; // Filtered list of tasks
	let filterText: string = ''; // Text for filtering tasks
	let selectedTaskId: number | null = null; // Selected task ID
	let error: string | null = null; // Error message (if any)
	let selectedTaskFields: TaskField[] = []; // Form fields for the selected task

	// Fetch tasks from the Flowable API
	onMount(async () => {
		try {
			const response = await fetch('/api/runtime/tasks', {
				method: 'GET',
				headers: {
					'Content-Type': 'application/json',
					Authorization: `Basic ${btoa('admin:test')}` // Replace with actual credentials
				}
			});

			if (!response.ok) {
				throw new Error(`Error: ${response.status} ${response.statusText}`);
			}

			// Map the Flowable API response to Task type
			const data = await response.json();
			tasks = data.data.map((task: any) => ({
				id: task.id,
				title: task.name || 'Untitled Task',
				description: task.description || 'No description provided',
				assignedTo: task.assignee || 'Unassigned',
				creationDate: new Date(task.createTime).toLocaleDateString(),
				fields: [] // Fields are fetched dynamically
			}));

			filteredTasks = tasks; // Initialize filtered tasks
		} catch (err: unknown) {
			error = err instanceof Error ? err.message : 'Failed to load tasks.';
		}
	});

	// Update filtered tasks whenever filterText changes
	$: {
		filteredTasks = tasks.filter((task) =>
			task.title.toLowerCase().includes(filterText.toLowerCase())
		);
		if (filterText) {
			selectedTaskId = null; // Reset selected task when filtering
		}
	}

	// Handle task selection and fetch form fields
	async function selectTask(taskId: number): Promise<void> {
		selectedTaskId = taskId;

		// Fetch the form fields for the selected task
		try {
			const response = await fetch(`/api/runtime/tasks/${taskId}/form`, {
				method: 'GET',
				headers: {
					'Content-Type': 'application/json',
					Authorization: `Basic ${btoa('admin:test')}`
				}
			});

			if (!response.ok) {
				if (response.status === 404) {
					// No form available for the task
					selectedTaskFields = [];
					return;
				}
				throw new Error(`Error fetching form: ${response.status} ${response.statusText}`);
			}

			// Map the form fields to TaskField type
			const formData = await response.json();
			selectedTaskFields = formData.fields.map((field: any) => ({
				label: field.name || 'Unnamed Field',
				value: field.value || ''
			}));
		} catch (err: unknown) {
			selectedTaskFields = []; // Clear fields on error or no form
		}
	}

	// Handle task completion
	function completeTask(taskId: number | null): void {
		alert(`Task ${taskId} marked as complete!`);
	}

	// Handle creating a new task
	function createNewTask(): void {
		alert('Create a new task!');
	}

	// Get selected task data
	function getSelectedTask() {
		return tasks.find((task) => task.id === selectedTaskId);
	}
</script>

<div class="layout">
	{#if error}
		<p class="error">{error}</p>
	{/if}

	<!-- Sidebar with task list -->
	<div class="sidebar card">
		<input
			type="text"
			class="task-filter"
			placeholder="Filter tasks by name..."
			bind:value={filterText}
		/>

		<div class="sidebar-panel">
			<button class="btn btn-normal" on:click={createNewTask}>New Task</button>
		</div>

		{#if !tasks.length}
			<p>No tasks available.</p>
		{:else}
			<ul>
				{#each filteredTasks as task}
					<li class:selected={task.id === selectedTaskId}>
						<button on:click={() => selectTask(task.id)}>
							<h4>{task.title}</h4>
							<p>{task.description}</p>
							<p>Assigned to {task.assignedTo}</p>
							<p>{task.creationDate}</p>
						</button>
					</li>
				{/each}
			</ul>
		{/if}
	</div>

	<!-- Main panel with task details -->
	<div class="main card">
		{#if selectedTaskId !== null}
			<div class="header">
				<div class="header-top">
					<h1>{getSelectedTask()?.title}</h1>
					<button class="btn btn-small" on:click={() => completeTask(selectedTaskId)}>✔</button>
				</div>
				<div class="details">
					<p><strong>Description:</strong> {getSelectedTask()?.description}</p>
					<p><strong>Assigned To:</strong> {getSelectedTask()?.assignedTo}</p>
					<p><strong>Created On:</strong> {getSelectedTask()?.creationDate}</p>
				</div>
			</div>

			<div class="form">
				{#if selectedTaskFields.length > 0}
					{#each selectedTaskFields as field, index}
						<div class="form-group">
							<label for="field-{index}">{field.label}</label>
							<input
								type="text"
								id="field-{index}"
								bind:value={field.value}
								placeholder={`Enter ${field.label.toLowerCase()}`}
							/>
						</div>
					{/each}
				{:else}
					<div class="centered-container">
						<div class="centered-block">
							<p>No fields available for this task.</p>
						</div>
					</div>
				{/if}
			</div>
		{:else}
			<div class="centered-container">
				<div class="centered-block">
					<p>Select a task to see details.</p>
				</div>
			</div>
		{/if}
	</div>
</div>
