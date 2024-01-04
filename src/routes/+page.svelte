<svelte:head>
    <title>Todo List App</title> 
</svelte:head>

<script lang="ts">
  type Todo = {
    text: string;
    done: boolean;
  };

  type Filter = "all" | "active" | "done";

  import { writable } from "svelte/store";

  let todoList = writable<Todo[]>([]);
  let filter = writable<String>("all");

  $: {
    if (typeof localStorage !== "undefined") {
      if (localStorage.getItem("todoList")) {
        todoList.set(JSON.parse(localStorage.getItem("todoList")!));
      }
    }
  }

  $: {
    if (typeof localStorage !== "undefined") {
      localStorage.setItem("todoList", JSON.stringify($todoList));
    }
  }

  const addTodo = (e: KeyboardEvent) => {
    if (e.key !== "Enter") return;
    const elm = e.target as HTMLInputElement;
    todoList.update((list) => [...list, { text: elm.value, done: false }]);
    elm.value = "";
  };

  const editTodo = (e: Event) => {
    const elm = e.target as HTMLInputElement;
    const index = +elm.dataset.index!;
    $todoList[index].text = elm.value;
  };

  const toggledTodo = (e: Event) => {
    const elm = e.target as HTMLInputElement;
    const index = +elm.dataset.index!;
    $todoList[index].done = !$todoList[index].done;
  };

  const changedFilter = (value: Filter) => {
    filter.set(value);
  };

  const deleteTodo = (index: number) => {
    todoList.update((list) => list.filter((_, i) => i !== index));
  };
</script>

<div class="center">
  <div class="todo-list">
    <div class="todo">
      <input type="text" placeholder="Enter todo ..." on:keydown={addTodo} />
    </div>

    <div class="filter">
      <p>Filter</p>
      <button on:click={() => changedFilter("all")}>All</button>
      <button on:click={() => changedFilter("active")}>Active</button>
      <button on:click={() => changedFilter("done")}>Done</button>
    </div>

    {#each $todoList as todo, i (todo)}
      {#if $filter === "all" || ($filter === "active" && !todo.done) || ($filter === "done" && todo.done)}
        <div class="todo">
          <button class="btn-delete" on:click={() => deleteTodo(i)}> x </button>
          <input type="text" class:done={todo.done} value={todo.text} on:input={editTodo} data-index={i} />
          <input type="checkbox" checked={todo.done} on:change={toggledTodo} data-index={i} />
        </div>
      {/if}
    {/each}
  </div>
</div>

<style>
  .center {
    height: 100dvh;
    display: flex;
    flex-direction: column;
    gap: 10px;
    justify-content: center;
    align-items: center;
  }

  .filter {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 10px;
    justify-content: start;
    width: 100%;
  }

  .todo-list {
    width: 100%;
    max-width: 400px;
    display: flex;
    flex-direction: column;
    gap: 10px;
    justify-content: center;
    align-items: center;
  }

  .todo {
    width: 100%;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .btn-delete {
    margin-right: 10px;
  }
  .btn-delete:hover {
    background: red;
    border-color: transparent;
    color: whitesmoke;
  }

  .done {
    background: green;
    border: 2px solid #fdfcfa;
  }

  input[type="text"] {
    width: 100%;
    padding: 1rem 0.5rem;
  }

  input[type="checkbox"] {
    position: absolute;
    right: 5%;
    top: 45%;
    translate: 0% -45%;
  }
</style>
