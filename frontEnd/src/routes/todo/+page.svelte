<script>
	// @ts-nocheck

	import TodoListSection from './TodoListSection.svelte';
	import { v4 as uuid } from 'uuid';
	import { afterNavigate } from '$app/navigation';

	import Header from './Header.svelte';

	let todoInput, todoItemsWrapper, toDoList;
	let todoItems = [];
	let loadingError, isLoading;
	let isAdding = false;
	let disabledItems = [];
	let isTodoItemsLoaded = false;

	let prevTodoItems = todoItems;
	let autoScroll = false;
	let transitionCounter = 300;

	async function loadTodos() {
		isLoading = true;
		await fetch('https://jsonplaceholder.typicode.com/todos?_limit=10')
			.then((response) => {
				if (response.ok) {
					return response.json();
				} else {
					loadingError = 'Error loading todos from server';
					console.log('Loading error');
				}
			})
			.then((todoItemsFromAPI) => (todoItems = todoItemsFromAPI));
		isLoading = false;

		isTodoItemsLoaded = true;
	}

	afterNavigate((navigation) => {
		console.log(navigation);
		loadTodos();
	});

	function addTodo() {
		if (!todoInput) return;

		isAdding = true;
		fetch('https://jsonplaceholder.typicode.com/todos', {
			method: 'POST',
			headers: { 'Content-type': 'application/json' },
			body: JSON.stringify({
				id: uuid(),
				title: todoInput,
				completed: false
			})
		})
			.then((response) => response.json())
			.then((element) => {
				todoItems = [{ ...element, id: uuid() }, ...todoItems];
				console.log(todoItems);
			});

		todoInput = '';
	}

	function handleToggleTodo(event) {
		const id = event.detail.id;
		console.log(event);

		if (disabledItems.includes(id)) return;
		disabledItems = [...disabledItems, id];

		fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
			method: 'PATCH',
			headers: {
				'Content-type': 'application/json; charset=UTF-8'
			},
			body: JSON.stringify({
				completed: event.detail.value
			})
		}).then(async (response) => {
			if (response.ok) {
				const updatedTodo = await response.json();

				if (!('id' in updatedTodo)) {
					todoItems = todoItems.map((todo) => {
						if (todo.id === event.detail.id) {
							return { ...todo, completed: event.detail.value };
						}
						return { ...todo };
					});
				} else {
					todoItems = todoItems.map((todo) => {
						if (todo.id === event.detail.id) {
							return updatedTodo;
						}
						return { ...todo };
					});
				}
			}

			disabledItems = disabledItems.filter((element) => {
				element !== id;
			});
		});
	}

	async function handleDeleteTodo(event) {
		const id = event.detail.id;

		if (disabledItems.includes(id)) return;
		disabledItems = [...disabledItems, id];

		await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
			method: 'DELETE'
		}).then((response) => {
			if (response.ok) {
				todoItems = todoItems.filter((element) => element.id !== event.detail.id);
			}
		});

		disabledItems = disabledItems.filter((element) => {
			element !== id;
		});
	}

	function todoListAfterUpdate(event) {
		console.log(event.detail);
		if (event.detail.autoScroll && isAdding) {
			todoItemsWrapper.scrollTo(0, 0);
		}
		isAdding = false;
	}
</script>

<Header></Header>
<main class="prose m-auto h-screen">
	<div bind:this={todoItemsWrapper} class="todoList-wrapper">
		<TodoListSection
			bind:this={toDoList}
			{disabledItems}
			{isLoading}
			{loadingError}
			{todoItems}
			on:toggletodo={handleToggleTodo}
			on:afterUpdate={todoListAfterUpdate}
			on:deleTodo={handleDeleteTodo}
		/>
	</div>
	<form on:submit|preventDefault={addTodo}>
		<input
			type="text"
			bind:value={todoInput}
			placeholder="ToDo title"
			class="mt-10 input input-bordered w-full"
		/>
		<button class="mt-5 btn btn-primary w-full bg-base-content" type="submit" disabled={!todoInput}>
			{#if isAdding}<span class="loading loading-spinner"></span>
			{:else}
				Add ToDo
			{/if}
		</button>
	</form>
	<a href="/">Home</a>
</main>

<style>
	main {
		max-width: 600px;
		padding: 0 10px;
	}

	.todoList-wrapper {
		overflow: auto;
		height: calc(50vh);
		/* border: 1px solid black; */
		border-radius: 20px;
		/* background-color: white; */
	}
</style>
