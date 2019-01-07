<template>
  <div>
    <input type="text" 
    class="todo-input" 
    placeholder="What needs to be done"
    v-model="newTodo"
    @keyup.enter="addTodo">
    <transition-group name="fade" enter-active-class="animated fadeInUp" leave-active-class="animated fadeOutDown">
      <todo-item v-for="todo in todosFiltered" 
      :key="todo.id" 
      :todo="todo" 
      :checkAll="!anyRemaining">
        <!-- <div class="todo-item-left">
          <input type="checkbox" v-model="todo.completed">
          <div v-if="!todo.editing" 
          @dblclick="editTodo(todo)" 
          class="todo-item-label"
          :class="{ completed : todo.completed }">
          {{ todo.title }}</div>
          <input v-else class="todo-item-edit" 
          type="text" v-model="todo.title" 
          @blur="doneEdit(todo)" 
          @keyup.enter="doneEdit(todo)" 
          @keyup.esc="cancelEdit(todo)" 
          v-focus>
        </div>
        <div class="remove-item" @click="removeTodo(index)">
          &times;
        </div> -->
      </todo-item> 
    </transition-group>
    <div class="extra-container">
        <todo-check-all :anyRemaining="anyRemaining"></todo-check-all>
        <todo-items-remaining :remaining="remaining"></todo-items-remaining>
    </div>

    <div class="extra-container"> 
      <todo-filtered></todo-filtered>
      <div>
        <transition name="fade">
          <todo-clear-completed :showClearCompletedButton="showClearCompletedButton"></todo-clear-completed>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
import TodoItem from './TodoItem'
import TodoItemsRemaining from './TodoItemsRemaining'
import TodoCheckAll from './TodoCheckAll'
import TodoFiltered from './TodoFiltered'
import TodoClearCompleted from './TodoClearCompleted'

export default {
  name: 'todo-list',
  components: {
    TodoItem,
    TodoItemsRemaining,
    TodoCheckAll,
    TodoFiltered,
    TodoClearCompleted,
  },
  data () {
    return {
      newTodo: '',
      idForTodo: 3,
      beforeEditCache: '',
      filter: 'all',
      todos: [
        {
          'id': 1,
          'title': 'Finish vue',
          'completed': false,
          'editing': false,
        },
        {
          'id': 2,
          'title': 'Take over world',
          'completed': false,
          'editing': false,
        }
      ],
    } 
  },
  created() {
    eventBus.$on('removeTodo', (id) => this.removeTodo(id))
    eventBus.$on('finishedEdit', (data) => this.finishedEdit(data))
    eventBus.$on('checkAllChanged', (checked) => this.checkAllTodos(checked))
    eventBus.$on('filterChanged', (filter) => this.filter = filter)
    eventBus.$on('clearCompletedTodos', () => this.clearCompleted())
  },
  beforeDestroy() {
    eventBus.$off('removeTodo', (id) => this.removeTodo(id))
    eventBus.$off('finishedEdit', (data) => this.finishedEdit(data))
    eventBus.$off('checkAllChanged', (checked) => this.checkAllTodos(checked))
    eventBus.$off('filterChanged', (filter) => this.filter = filter)
    eventBus.$off('clearCompletedTodos', () => this.clearCompleted())
  },
  computed: {
    remaining() {
      return this.todos.filter(todo => !todo.completed).length
    },
    anyRemaining() {
      return this.remaining != 0
    },
    todosFiltered() {
      if(this.filter === 'all') {
        return this.todos
      } else if(this.filter === 'active') {
        return this.todos.filter(todo => !todo.completed)
      } else if(this.filter === 'completed') {
        return this.todos.filter(todo => todo.completed)
      }
      return this.todos
    },
    showClearCompletedButton() {
      return this.todos.filter(todo => todo.completed).length > 0
    },
  },
  methods: {
    addTodo() {
      if(this.newTodo.trim().length == 0) {
        return
      }
      this.todos.push({
        id: this.idForTodo,
        title: this.newTodo,
        completed: false,
      })

      this.newTodo = ''
      this.idForTodo++
    },
    // editTodo(todo) {
    //   this.beforeEditCache = todo.title
    //   todo.editing = true
    // },
    // doneEdit(todo) {
    //   if(todo.title.trim() === '') {
    //     todo.title = this.beforeEditCache
    //   }
    //   todo.editing = false
    // },
    // cancelEdit(todo) {
    //   todo.title - this.beforeEditCache
    //   todo.editing = false
    // },
    removeTodo(id) {
      // this.todos.splice(index, 1)
      const index = this.todos.findIndex((item) => item.id == id)
      this.todos.splice(index, 1) 
    },
    checkAllTodos() {
      this.todos.forEach((todo) => todo.completed = event.target.checked)
    },
    clearCompleted() {
      this.todos = this.todos.filter(todo => !todo.completed)
    },
    finishedEdit(data) {
      // this.todos.splice(data.index, 1, data.todo)
      const index = this.todos.findIndex((item) => item.id == data.id)
      this.todos.splice(index, 1, data)
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
@import url(https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.0/animate.min.css);
  .todo-input {
    width: 100%;
    padding: 10px 18px;
    font-size: 18px;
    margin-bottom: 16px;

    &:focus {
      outline: 0; 
    }
  }

  .todo-item {
    margin-bottom: 12px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    animation-duration: 0.3;
  }

  .remove-item {
    cursor: pointer;
    margin-left: 14px;
    &:hover {
      color: black;
    }
  }

  .todo-item-left {
    display: flex;
    align-items: center;
  }

  .todo-item-label {
    padding: 10px;
    border: 1px solid #fff;
  }

  .todo-item-edit {
    font-size: 24px;
    color: #2c3e50;
    margin-left: 12px;
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    font-family: 'Avenir', Helvetica, sans-serif;

    &:focus {
      outline: none;
    }
  }

  .completed {
    text-decoration: line-through;
    color: gray;
  }

  .extra-container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 16px;
    border-top: 1px solid lightgray;
    padding-top: 14px;
    margin-bottom: 14px;
  }

  button {
    font-size: 14px;
    background-color:#fff;
    appearance: none;

    &:hover {
      background: lightgreen;
    }

    &:focus {
      outline: none;
    }
  }

  .active {
    background: lightgreen; 
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity .2s;
  }

  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
</style>
