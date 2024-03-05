<script setup>
  import { ref, onMounted, computed, watch} from 'vue'
  const todos = ref([])
  const userName = ref('')
  const isNameInput = ref(false);
  const greeting = 'Hi🖐🏻What your Name?'
  const categories = ref(['Personal'])

  const input_content = ref('')
  const input_category = ref(null)
  const input_date = ref(null)

  const todosList = computed(()=> todos.value.sort((a, b) =>{
    console.log(a.timestamp - b.timestamp)
    return a.timestamp - b.timestamp
  }))
  const addNewCategory = () => {
    const newCategory = prompt('Enter new category:');
    if (newCategory && !categories.value.includes(newCategory)) {
      // เพิ่ม category ใหม่ลงใน array
      categories.value.push(newCategory);
      // อัปเดต localStorage
      localStorage.setItem('categories', categories.value);
      }
    }


    const calculateTimeDifference = (dueDate) => {
    const currentDate = new Date();
    const dueDateTime = new Date(dueDate);
    const timeDifference = dueDateTime - currentDate;

    // Convert the time difference to minutes
    const totalMinutes = Math.floor(timeDifference / (1000 * 60));
    
    // Calculate days, hours, and remaining minutes
    const days = Math.floor(totalMinutes / (24 * 60));
    const hours = Math.floor((totalMinutes % (24 * 60)) / 60);
    const minutes = totalMinutes % 60;

    // Create a formatted string
    let formattedTime = '';

    if (days > 0) {
      formattedTime += `${days} วัน `;
    }

    if (hours > 0) {
      formattedTime += `${hours} ชั่วโมง `;
    }

    if (minutes > 0) {
      formattedTime += `${minutes} นาที`;
    }

    return formattedTime.trim();
};

    const addTodo = () => {
    if (input_content.value.trim() === '' || input_category.value === null){
      return
    }
    const localTimeZoneOffset = new Date().getTimezoneOffset();
    const currentDateTime = new Date();
    const offsetDateTime = new Date(currentDateTime.getTime() - localTimeZoneOffset * 60000);
    const formattedDateTime = offsetDateTime.toISOString().slice(0, 16);
    
    todos.value.push({
      content: input_content.value,
      category: input_category.value,
      done:false ,
      createAt: formattedDateTime,
      timestamp: new Date().getTime(), //UTC timestamp
      dueDate: input_date.value,
      timeRemaining: calculateTimeDifference(input_date.value),
    })
    console.log('add todo')
  }

  const removeTodo = todo =>{
    todos.value = todos.value.filter(t => t!== todo)
  }
  const enableEdit = (todo) => {
    todo.isEditing = true;
    todo.originalContent = todo.content;
  };

  const saveEdit = (todo) => {
    todo.isEditing = false;
    // Optionally, you can perform additional validation or updates here
  };

  const cancelEdit = (todo) => {
    todo.isEditing = false;
    todo.content = todo.originalContent;
  };

  watch(todos,newVal =>{
    localStorage.setItem('todos', JSON.stringify(newVal))
  },{deep:true})

  watch(userName, (newVal) => {
    localStorage.setItem('userName', newVal)
    isNameInput.value = userName.value !== ''; //if userName value empty state = true
  })

  const updateTodoList = () => {
      todos.value.forEach((todo) => {
        todo.timeRemaining = calculateTimeDifference(todo.dueDate);
  })// Schedule the next update
      requestAnimationFrame(updateTodoList);}

  onMounted(() => {
      userName.value = localStorage.getItem('userName') || '';
      categories.value = localStorage.getItem('categories') ? localStorage.getItem('categories').split(',') : [];
      categories.value.sort();
      todos.value = JSON.parse(localStorage.getItem('todos')) || []
      updateTodoList()
    })

</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        {{isNameInput ? "What's up" : greeting}}<input type="text" placeholder="Your Name" 
        v-model="userName" />
      </h2>
    </section>
    <section class="create-todo">
      <h3>Create Task</h3>
      <form @submit.prevent="addTodo">
        <h4>What your Task?</h4>
        <input type="text" placeholder="text..." v-model="input_content" />
        <h4>Set Category</h4>
        <div class="options">
          <label>
            <select name="category" v-model="input_category">
              <option v-for="category in categories" :key="category" :value="category">
                {{ category }}
              </option>
            </select>
          </label>
          <button type="button" @click="addNewCategory">New Category</button>
          {{ input_category }}
        </div>
        <h4>Date Due</h4>
        <input type="datetime-local" name="date" v-model="input_date">
        {{ input_date }}
        <input type="submit" value="Add task">
      </form>
    </section>
    <section class="todo-list">
      <h3>TODO list</h3>
      <span>Todo | Duedate |Time remain</span>
      <div class="list">
        <div v-for='todo in todosList' :class="`todo-item ${todo.done && 'done'}`">
          <label>
            <input type="checkbox" name="state" v-model="todo.done">
            <!-- <span :class="`bubble ${todo.category}`"></span> -->
          </label>
          <div class="todo-content">
            <input type="text" v-model="todo.content" :readonly="!todo.isEditing" />
            <input type="datetime-local" v-model="todo.dueDate" :readonly="!todo.isEditing" />
            {{ todo.timeRemaining }}
          </div>
          <div class="action">
            <button v-if="!todo.isEditing" class="edit" @click="enableEdit(todo)">Edit</button>
            <button v-if="todo.isEditing" class="save" @click="saveEdit(todo)">Save</button>
            <button v-if="todo.isEditing" class="cancel" @click="cancelEdit(todo)">Cancel</button>
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

