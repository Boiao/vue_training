<script>
export default {
  data() {
    return {
      todos: [],
      hideCompleted: false,
      name: "",
      input_content: "",
      input_category: null,
    };
  },

  computed: {
    todos_asc() {
      let sorted = this.todos.sort((a, b) => {
        return b.timestamp - a.timestamp;
      });
      return this.hideCompleted ? sorted.filter((todo) => !todo.done) : sorted;
    },
  },

  methods: {
    addTodo() {
      if (this.input_content.trim() === "" || this.input_category === null)
        return;

      this.todos.push({
        content: this.input_content,
        category: this.input_category,
        done: false,
        timestamp: Date.now(),
      });
      this.input_content = "";
      this.input_category = null;
    },

    removeTodo(todo) {
      this.todos = this.todos.filter((t) => t !== todo);
    },

    toggleCheckbox() {
      this.hideCompleted = !this.hideCompleted;
    },
  },

  mounted() {
    this.name = localStorage.getItem("name") || "";
    this.todos = JSON.parse(localStorage.getItem("todos")) || [];
  },

  watch: {
    todos: {
      handler(newVal) {
        localStorage.setItem("todos", JSON.stringify(newVal));
      },
      deep: true,
    },

    name(newVal) {
      localStorage.setItem("name", newVal);
    },
  },
};
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        What's up, <input type="text" placeholder="Your Name" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>NEW TODO</h3>

      <form @submit.prevent="addTodo">
        <h4>What's on your todo list?</h4>
        <input
          type="text"
          placeholder="e.g deliver project"
          v-model="input_content"
        />
        <h4>What category is this todo?</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              value="business"
              v-model="input_category"
            />
            <span class="bubble business"></span>
            <div>Business</div>
          </label>

          <label>
            <input
              type="radio"
              name="category"
              value="personal"
              v-model="input_category"
            />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>
        </div>
        <input type="submit" value="Add todo" />
      </form>
    </section>
    <section class="todo-list">
      <div class="title">
        <h3>TODO LIST</h3>
        <div class="hideswitch">
          <a>Hide Completed</a>
          <label class="switch">
            <input type="checkbox" @click="toggleCheckbox" />
            <div class="slider round"></div>
          </label>
        </div>
      </div>

      <div class="list">
        <div
          v-for="todo in todos_asc"
          :class="`todo-item ${todo.done && 'done'}`"
          :key="todo.content"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>
          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>
          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>
