<script>
    // @ts-nocheck

    // @ts-ignore
    import axios from "axios";
    import { onMount } from "svelte";

    const apiPrefix = "http://localhost:4000";
    const filters = ["All", "ToDo", "Done"];
    let todos = [];
    let text = "";
    let selectedFilter = "All";
    $: filteredTodos = filterTodos(todos, selectedFilter);
    onMount(async () => {
        todos = await getTodos();
    });

  

    function handleDelete(todo) {
        axios.delete(`${apiPrefix}/api/all/` + todo._id).then((response) => {
            todos = response.data;
        });
        todo = "";
    }
    
    function handlePost() {
        axios
            .post(`${apiPrefix}/api/all`, { text: text })
            // @ts-ignore
            .then((response) => {
                todos = response.data;
            });
        text = "";
    }

    function getTodos() {
        axios.get(`${apiPrefix}/api/all`).then((response) => {
            todos = response.data;
        });
    }

    async function changeTodo(id, state) {
        await axios
            .patch(`${apiPrefix}/api/all/` + id, { done: state })
            .then(getTodos);
    }

    function setFilter(newFilter) {
        selectedFilter = newFilter;
    }
    function filterTodos(todos, filter) {
        if (filter === "ToDo") {
            return todos.filter((text) => !text.done);
        } else if (filter === "Done") {
            return todos.filter((text) => text.done);
        }
        return todos;
    }
</script>

<div class="content">
    <div class="row w-[40rem] flex mx-auto mt-8">
        {#each filters as filter}
            <button
                class="mx-auto w-[100%] border border-solid"
                on:click={() => setFilter(filter)}
                class:selected={selectedFilter === filter}
            >
                {filter}
            </button>
        {/each}
    </div>

    {#each filteredTodos as text (text._id)}
        <div class="w-[40rem] flex mx-auto">
            <div class="mx-auto mt-6 grid grid-cols-3">
                <input
                    type="checkbox"
                    checked={text.done}
                    on:change={async () =>
                        await changeTodo(text._id, !text.done)}
                />
                <span>
                    {text.text}
                </span>
                <button class="ml-6 my-auto" on:click={handleDelete(text)}>
                    <svg
                        xmlns="http://www.w3.org/2000/svg"
                        class="h-6 w-6"
                        fill="none"
                        viewBox="0 0 24 24"
                        stroke="currentColor"
                        stroke-width="2"
                    >
                        <path
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
                        />
                    </svg>
                </button>
            </div>
        </div>
    {/each}

    <div class="w-[40rem] flex mx-auto">
        <div class="mx-auto mt-16">
            <div class="text-xl text-center mb-4">
                Aby dodać wpisz i kliknij przycisk
            </div>
            <input
                type="text"
                class="border-b border-b-black"
                bind:value={text}
            />
            <button class="ml-6 border w-[10rem] h-8" on:click={handlePost}>
                Dodaj
            </button>
        </div>
    </div>
</div>
