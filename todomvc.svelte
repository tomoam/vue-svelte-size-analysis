<script>
	const ENTER_KEY = 13;
	const ESCAPE_KEY = 27;

	const STORAGE_KEY = 'todos-svelte';
	const filters = {
	  all: (todos) => todos,
	  active: (todos) => todos.filter((todo) => !todo.completed),
	  completed: (todos) => todos.filter((todo) => todo.completed)
	}

	let currentFilter = 'all';
	let items = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
	let editing = null;

	const updateView = () => {
		const route = window.location.hash.replace(/#\/?/, '')
  		if (filters[route]) {
		  currentFilter = route;
  		} else {
		  currentFilter = 'all';
  		}
	};

	window.addEventListener('hashchange', updateView);
	updateView();

	function clearCompleted() {
		items = filters.active(items);
	}

	function remove(index) {
		items.splice(index, 1)
		items = items;
	}

	function toggleAll(event) {
		items = items.map(item => ({
			id: item.id,
			title: item.title,
			completed: event.target.checked
		}));
	}

	function createNew(event) {
		if (event.which === ENTER_KEY) {
			items = items.concat({
				id: Date.now(),
				title: event.target.value,
				completed: false
			});
			event.target.value = '';
		}
	}

	function handleEdit(event) {
		if (event.which === ENTER_KEY) event.target.blur();
		else if (event.which === ESCAPE_KEY) editing = null;
	}

	function submit(event) {
		items[editing].title = event.target.value;
		editing = null;
	}

    $: filtered = filters[currentFilter](items);
	$: numActive = filters.active(items).length;
	$: numCompleted = filters.completed(items).length;

	$: localStorage.setItem(STORAGE_KEY, JSON.stringify(items));
</script>

<header class="header">
	<h1>todos</h1>
	<input
		class="new-todo"
		on:keydown={createNew}
		placeholder="What needs to be done?"
		autofocus
	>
</header>

<section class="main" style={items.length > 0 ? "" : "display: none;"}>
	<input id="toggle-all" class="toggle-all" type="checkbox" on:change={toggleAll} checked="{numCompleted === items.length}">
	<label for="toggle-all">Mark all as complete</label>

	<ul class="todo-list">
		{#each filtered as item, index (item.id)}
			<li class="{item.completed ? 'completed' : ''} {editing === index ? 'editing' : ''}">
				<div class="view">
					<input class="toggle" type="checkbox" bind:checked={item.completed}>
					<label on:dblclick="{() => editing = index}">{item.title}</label>
					<button on:click="{() => remove(index)}" class="destroy"></button>
				</div>

				{#if editing === index}
					<input
						value='{item.title}'
						class="edit"
						on:keydown={handleEdit}
						on:blur={submit}
						autofocus
					>
				{/if}
			</li>
		{/each}
	</ul>

	<footer class="footer">
		<span class="todo-count">
			<strong>{numActive}</strong> {numActive === 1 ? 'item' : 'items'} left
		</span>

		<ul class="filters">
			<li><a class="{currentFilter === 'all' ? 'selected' : ''}" href="#/">All</a></li>
			<li><a class="{currentFilter === 'active' ? 'selected' : ''}" href="#/active">Active</a></li>
			<li><a class="{currentFilter === 'completed' ? 'selected' : ''}" href="#/completed">Completed</a></li>
		</ul>

		<button class="clear-completed" on:click={clearCompleted} style={numCompleted ? "" : "display:none;"}>
			Clear completed
		</button>
	</footer>
</section>