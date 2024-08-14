<script lang="ts">
	// @ts-nocheck
	import { fade, fly } from 'svelte/transition';
	import { flip } from 'svelte/animate';
	import { Trash2 } from 'lucide-svelte';
	import { compile } from 'svelte/compiler';

	import { afterUpdate, createEventDispatcher } from 'svelte';
	const dispatch = createEventDispatcher();

	export let todoItems = [];
	export let isLoading = false;
	export let loadingError = null;
	export let disabledItems = [];

	let prevTodoItems = todoItems;
	let autoScroll = false;
	let transitionCounter = 300;

	$: {
		autoScroll = todoItems.length > prevTodoItems.length;
		prevTodoItems = todoItems;
	}

	function handleToggleTodo(id, value) {
		dispatch('toggletodo', {
			id,
			value
		});
	}

	function handleDeleteTodo(id) {
		dispatch('deleTodo', {
			id
		});
	}

	afterUpdate(() => {
		dispatch('afterUpdate', {
			autoScroll
		});
		console.log(todoItems);
	});
</script>

{#if loadingError}
	<p
		in:fly|local={{ x: 100 }}
		out:fade|local={{ duration: 100 }}
		class="todo-element__not-founded-label"
	>
		{loadingError}
	</p>
{:else if isLoading}
	<p
		in:fly|local={{ x: 100 }}
		out:fade|local={{ duration: 100 }}
		class="todo-element__not-founded-label"
	>
		Loading...
	</p>
{:else if todoItems.length > 0}
	<ul class="list-none p-0">
		{#each todoItems as todoItem (todoItem.id)}
			<li class="p-0" out:fly|local={{ x: -100 }} animate:flip={{ duration: 400 }}>
				<div class="card bg-base-300 p-3 rounded-box flex flex-row justify-between items-center">
					<div class="flex flex-row justify-between items-center">
						<input
							class="checkbox"
							type="checkbox"
							checked={todoItem.completed}
							disabled={disabledItems.includes(todoItem.id)}
							on:input={(event) => {
								event.currentTarget.checked = todoItem.completed;
								handleToggleTodo(todoItem.id, !todoItem.completed);
							}}
						/>
						<span class="ml-5 mr-5">{todoItem.title}</span>
					</div>
					<button
						class="btn p-2 btn-warning"
						on:click={() => {
							console.log(todoItem.id);
							handleDeleteTodo(todoItem.id);
						}}
						disabled={disabledItems.includes(todoItem.id)}><Trash2 /></button
					>
				</div>
			</li>
		{/each}
	</ul>
{:else}
	<p class="todo-element__not-founded-label">Elements not founded!</p>
{/if}
