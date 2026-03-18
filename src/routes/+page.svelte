<script lang="ts">
import { onMount } from 'svelte';

onMount(() => {
	const saved = localStorage.getItem('tasks');
	if (saved) {
		tasks = JSON.parse(saved);
	}
});

$effect(() => {
	localStorage.setItem('tasks', JSON.stringify(tasks));
});

function formatDate(date: string) {
	return new Date(date).toLocaleString();
}
	type Task = {
		id: number;
		text: string;
		completed: boolean;
        dueDate?: string;
        comment?: string;
	};

    let editingId = $state<number | null>(null);
    let editText = $state('');
    let editDate = $state('');
    let editComment = $state('');

    function startEdit(task: Task) {
	editingId = task.id;
	editText = task.text;
	editDate = task.dueDate || '';
	editComment = task.comment || '';
}
function saveEdit(id: number) {
	tasks = tasks.map(task =>
		task.id === id
			? {
					...task,
					text: editText,
					dueDate: editDate || undefined,
					comment: editComment || undefined
			  }
			: task
	);

	cancelEdit();
}

function cancelEdit() {
	editingId = null;
	editText = '';
	editDate = '';
	editComment = '';
}

	let tasks = $state<Task[]>([]);
	let newTask = $state('');
    let dueDate = $state('');
    let comment = $state('');
    let filter = $state<'all' | 'active' | 'completed'>('all');
	function addTask() {
		if (!newTask.trim()) return;

		tasks = [
			...tasks,
			{
				id: Date.now(),
				text: newTask,
				completed: false,
                dueDate: dueDate || undefined,
                comment: comment || undefined
			}
		];

		newTask = '';
        dueDate = '';
        comment = '';
	}

	function toggleTask(id: number) {
		tasks = tasks.map(task =>
			task.id === id
				? { ...task, completed: !task.completed }
				: task
		);
	}

	function deleteTask(id: number) {
		tasks = tasks.filter(task => task.id !== id);
	}
   const filteredTasks = $derived.by((): Task[] => {
	if (filter === 'active') return tasks.filter(t => !t.completed);
	if (filter === 'completed') return tasks.filter(t => t.completed);
	return tasks;
});
</script>

<div class="container">
	<h1>📝 Task Manager</h1>

	<div class="input-box">
	<input
		placeholder="Add a task..."
		bind:value={newTask}
		onkeydown={ (e) => {
          if (e.key === 'Enter' && !e.shiftKey){
            e.preventDefault();
            addTask();
          }   
        } }
	/>

	<input
		type="datetime-local"
		bind:value={dueDate}
	/>
</div>

<textarea
	placeholder="Add a comment..."
	bind:value={comment}
></textarea>

<div class="actions">
   <button onclick={addTask}>Add</button> 
</div>

<div class="filters">
	<button
		class:active={filter === 'all'}
		onclick={() => filter = 'all'}
	>All</button>

	<button
		class:active={filter === 'active'}
		onclick={() => filter = 'active'}
	>Active</button>

	<button
		class:active={filter === 'completed'}
		onclick={() => filter = 'completed'}
	>Completed</button>
</div>

	{#if filteredTasks.length === 0}
		<p class="task-count">
            {tasks.filter(t => !t.completed).length} tasks found
        </p>
	{/if}
<p class="task-count">
	{#if filter === 'completed'}
		{tasks.filter(t => t.completed).length} completed
	{:else if filter === 'active'}
		{tasks.filter(t => !t.completed).length} active
	{:else}
		{tasks.filter(t => !t.completed).length} tasks left
	{/if}
</p>
<ul>
	
		{#each filteredTasks as task (task.id)}
	<li
		class:done={task.completed }
		class:overdue={task.dueDate && new Date(task.dueDate) < new Date()}
	>
		{#if editingId === task.id}
			<!-- ✏️ EDIT MODE -->
			<div class="task-content">
				<input 
                bind:value={editText}
                onkeydown={(e) => e.key === 'Enter' && saveEdit(task.id)}
                />

				<input type="datetime-local" bind:value={editDate} />

				<textarea bind:value={editComment}></textarea>

				<div class="edit-actions">
					<button onclick={() => saveEdit(task.id)}>💾</button>
					<button onclick={cancelEdit}>❌</button>
				</div>
			</div>
		{:else}
			<!-- 👀 VIEW MODE -->
			<input
				type="checkbox"
				checked={task.completed}
				onchange={() => toggleTask(task.id)}
			/>

			<div class="task-content">
				<span>{task.text}</span>

				{#if task.dueDate}
					<small class="date">
						📅 {formatDate(task.dueDate)}
					</small>
				{/if}

				{#if task.comment}
					<p class="comment">💬 {task.comment}</p>
				{/if}
			</div>

			<button onclick={() => startEdit(task)}>✏️</button>
			<button onclick={() => deleteTask(task.id)}>❌</button>
		{/if}
	</li>
{/each}
</ul>

</div>

<style>
	.container {
		max-width: 400px;
		margin: auto;
		text-align: center;
		font-family: Arial;
	}

	.input-box {
		display: flex;
		gap: 10px;
		margin-bottom: 20px;
	}

	input {
		flex: 1;
		padding: 10px;
		border-radius: 8px;
		border: 1px solid #ccc;
	}

	button {
		padding: 10px;
		border: none;
		border-radius: 8px;
		background: #4cafef;
		color: white;
		cursor: pointer;
	}

	ul {
		list-style: none;
		padding: 0;
	}

	li {
		display: flex;
		align-items: center;
        gap:10px;
		background: #f5f5f5;
		padding: 12px;
		margin-bottom: 10px;
		border-radius: 10px;
        transition: transform 0.2s ease;
	}

    li.done {
	    opacity: 0.6;
    }

	li.done span {
		text-decoration: line-through;
		color: gray;
	}

    li:hover {
        transform:translateY(-2px);
    }

    li button {
	background: transparent;
	color: red;
	font-size: 16px;
}

span {
	flex: 1;
	text-align: left;
}

textarea {
	width: 100%;
	padding: 10px;
	margin-bottom: 10px;
	border-radius: 8px;
	border: 1px solid #ccc;
}

.task-content {
	flex: 1;
	text-align: left;
	display: flex;
	flex-direction: column;
	gap: 4px;
}

.date {
	font-size: 12px;
	color: #555;
}

.comment {
	font-size: 13px;
	color: #333;
	margin: 0;
}

.overdue {
	border-left: 4px solid red;
    background: #ffe5e5;
}

.actions {
	margin-bottom: 20px;
}

.overdue .date {
    color: red;
    font-weight: bold;
}

.task-count {
	font-size: 14px;
	color: #555;
	margin-bottom: 10px;
}

p {
	color: #888;
	font-style: italic;
}

.filters {
	display: flex;
	gap: 10px;
	margin-bottom: 10px;
}

.filters button {
	flex: 1;
	background: #eee;
	color: #333;
    transition: all 0.2s ease;
}

.filters button:hover {
	background: #ddd;
}

.filters button.active {
	background: #4cafef;
	color: white;
}

button:hover {
	opacity: 0.9;
}

.edit-actions {
	display: flex;
	gap: 5px;
	margin-top: 5px;
}

.edit-actions button {
	background: #ddd;
	color: black;
}
</style>