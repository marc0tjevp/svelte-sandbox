<script lang="ts">
	type TaskField = {
		label: string;
		value: string | number;
	};

	type Task = {
		id: number;
		title: string;
		description: string;
		assignedTo: string;
		creationDate: string;
		fields: TaskField[];
	};

	let tasks: Task[] = [
		{
			id: 1,
			title: 'Fill in a Leave Application Form',
			description: 'Form submitted by Sterre',
			assignedTo: 'Test Admin',
			creationDate: '2024-12-10',
			fields: [
				{ label: 'Name', value: 'John Doe' },
				{ label: 'Amount Of Days', value: 5 }
			]
		},
		{
			id: 2,
			title: 'Review Project Plan',
			description: 'Plan submitted by Marco',
			assignedTo: 'Team Lead',
			creationDate: '2024-12-09',
			fields: [
				{ label: 'Reviewer Name', value: 'Marco' },
				{ label: 'Project Priority', value: 'High' }
			]
		},
		{
			id: 3,
			title: 'Empty Task',
			description: '',
			assignedTo: 'Admin',
			creationDate: '2024-12-09',
			fields: []
		}
	];

	let filterText: string = '';
	let filteredTasks = tasks;

	$: {
		filteredTasks = tasks.filter((task) =>
			task.title.toLowerCase().includes(filterText.toLowerCase())
		);
		if (filterText) {
			selectedTaskId = null;
		}
	}

	// Track the selected task
	let selectedTaskId: number | null = null;

	// Handle task selection
	function selectTask(taskId: number): void {
		selectedTaskId = taskId;
	}

	// Handle task complete (stub)
	function completeTask(taskId: number): void {
		alert(`Task ${taskId} marked as complete!`);
	}

    function createNewTask(): void {
		alert('Create a new task!');
	}

	// Get the selected task details
	$: selectedTask = tasks.find((task) => task.id === selectedTaskId) || null;
</script>

<div class="layout">
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
	</div>

	<div class="main card">
		{#if selectedTask}
			<div class="header">
				<div class="header-top">
					<h1>{selectedTask.title}</h1>
                    <button class="btn btn-small" on:click={() => completeTask(selectedTask.id)}>✔</button>
				</div>
				<div class="details">
					<p><strong>Description:</strong> {selectedTask.description}</p>
					<p><strong>Assigned To:</strong> {selectedTask.assignedTo}</p>
					<p><strong>Created On:</strong> {selectedTask.creationDate}</p>
				</div>
			</div>

			<div class="form">
				<form>
					{#each selectedTask.fields as field, index}
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
				</form>
			</div>
		{:else}
			<p>Select a task to see details.</p>
		{/if}
	</div>
</div>
