<script>
	import { initializeApp } from 'firebase/app';
	import { getFirestore, collection, onSnapshot, doc, addDoc, updateDoc, deleteDoc } from 'firebase/firestore';
	import { firebaseConfig } from '$lib/firebaseConfig';

	const app = initializeApp(firebaseConfig);
	const db = getFirestore(app);
	const todosCollection = collection(db, 'todos');

	let todos = [];

	const unsubscribe = onSnapshot(todosCollection, (querySnapshot) => {
        let dbTodos = [];

		querySnapshot.forEach((doc) => {
			const todo = { id: doc.id, ...doc.data() };

            // We can use the push() array function instead of assigning to the array like so
            // dbTodos = [...dbTodos, todo]
            // as we don't need reacttivity here.
			dbTodos.push(todo);
		});

        todos = dbTodos;
		console.table(todos);
	});

	let task = '';
	let error = '';

	const addTodo = async () => {
		if (task === '') {
			error = 'Task is empty';
			return;
		}

		const todo = { task: task, isComplete: false, createdAt: new Date() };
		await addDoc(collection(db, "todos"), todo);

		task = '';
        error = '';
	};

	const toggleTodoStatus = async (todo) => {
		await updateDoc(doc(db, "todos", todo.id), { isComplete: !todo.isComplete });
	};

	const deleteTodo = async (id) => {
		// let deleteTodo = todos[index];
		// todos = todos.filter((todo) => todo !== deleteTodo);

        await deleteDoc(doc(db, "todos", id));
	};

	const keyPressed = (event) => {
		if (event.key === 'Enter') {
			addTodo();
		}
	};
</script>

<input type="text" placeholder="Add a task" bind:value={task} />
<button on:click={addTodo}>Add</button>

<ol>
	{#each todos as todo, index}
		<li class:complete={todo.isComplete}>
			<span>{todo.task}</span>
			<span>
				<button on:click={() => toggleTodoStatus(todo)}>✅︎</button>
				<button on:click={() => deleteTodo(todo.id)}>❌</button>
			</span>
		</li>
	{:else}
		<p>No todos found</p>
	{/each}

	<p class="error">{error}</p>
</ol>

<svelte:window on:keydown={keyPressed} />

<style>
	.complete {
		text-decoration: line-through;
	}

	.error {
		color: red;
	}
</style>
