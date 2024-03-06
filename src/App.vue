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
  const nowDate = ref(new Date().toLocaleString('th-TH'));

  const todosList = computed(()=> todos.value.sort((a, b) =>{
    console.log(a.dueDatetimestamp - b.dueDatetimestamp)
    return a.dueDatetimestamp - b.dueDatetimestamp
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




    const addTodo = () => {
    if (input_content.value.trim() === '' || input_category.value === null){
      return
    }
    const localTimeZoneOffset = new Date().getTimezoneOffset();
    const currentDateTime = new Date();
    const offsetDateTime = new Date(currentDateTime.getTime() - localTimeZoneOffset * 60000);
    const formattedDateTime = offsetDateTime.toISOString().slice(0, 16);

    const currentDate = new Date();
    const dueDateTime = new Date(input_date.value);
    const timeDifference = dueDateTime - currentDate;

      // Convert the time difference to minutes
    const totalMinutes = Math.floor(timeDifference / (1000 * 60));
      
      // Calculate days, hours, and remaining minutes
    const days = Math.floor(totalMinutes / (24 * 60));
    const hours = Math.floor((totalMinutes % (24 * 60)) / 60);
    const minutes = totalMinutes % 60;


      if (days < 0) {
        return null;
      }

      if (hours < 0) {
        return null;
      }

      if (minutes < 0) {
        return null;
      }

    
    todos.value.push({
      content: input_content.value,
      category: input_category.value,
      done:false ,
      createAt: formattedDateTime,
      dueDatetimestamp: new Date(input_date.value),
      dueDate: input_date.value,
      days: days,
      hours: hours,
      minutes: minutes,
    })
    // Reset the form values
    input_content.value = '';
    input_category.value = null;
    input_date.value = null;
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

  const showDone = ref(false);
  const toggleShowDone = () => {
    showDone.value = !showDone.value;
  };

  const filteredTodos = computed(() => {
    if (showDone.value) {
      return todosList.value;
    } else {
      return todosList.value.filter(todo => !todo.done);
    }
  });
  const updateTodoList = () => {
      todos.value.forEach((todo) => {
        const currentDate = new Date();
        const dueDateTime = new Date(todo.dueDate);
        const timeDifference = dueDateTime - currentDate;

          // Convert the time difference to minutes
        const totalMinutes = Math.floor(timeDifference / (1000 * 60));
          
          // Calculate days, hours, and remaining minutes
        todo.days = Math.floor(totalMinutes / (24 * 60));
        todo.hours = Math.floor((totalMinutes % (24 * 60)) / 60);
        todo.minutes = totalMinutes % 60;

        //console.log(calculateTimeDifference(input_date.value))

  })// Schedule the next update
      requestAnimationFrame(updateTodoList);}

  onMounted(() => {
      userName.value = localStorage.getItem('userName') || '';
      categories.value = localStorage.getItem('categories') ? localStorage.getItem('categories').split(',') : [];
      categories.value.sort();
      todos.value = JSON.parse(localStorage.getItem('todos')) || []
      updateTodoList()
      setInterval(() => {
    nowDate.value = new Date().toLocaleString('th-TH');
    }, 1000);

    })


</script>

<template>
  <main class="app">
    <section class="greeting">
      <h1 class="mx-auto flex max-w-7xl items-center justify-center p-6 lg:px-8" aria-label="Global">
        {{isNameInput ? "What's up!" : greeting}}
        <input class="text-center max-w-fit p-0 m-0"  type="text" placeholder="Your Name" 
        v-model="userName" />
      </h1>
    </section>
    <section class="create-todo">



      <form @submit.prevent="addTodo">
        <section class="mx-auto flex preview max-w-5xl justify-between ">
          <div data-theme="dim" class="bg-white chat chat-start">
            <div class="chat-bubble w-auto text-white">What on your day!</div>
          </div> 
          
            <div data-theme="dim" class="chat chat-end bg-white mt-2">
            <div class="chat-image avatar">
              <div class="w-10 rounded-full">
                <img alt="Tailwind CSS chat bubble component" src="https://daisyui.com/images/stock/photo-1534528741775-53994a69daeb.jpg" />
              </div>
            </div>

            <div class="chat-header text-black">
              {{ userName }}
              <time class="text-xs text-black opacity-80"> {{ nowDate }} </time>
            </div>
          
            <input class=" chat-bubble text-center  text-white relative focus:outline-none max-w-md h-auto"  oninput="this.size = this.value.length" type="text" placeholder="text..." v-model="input_content" />
          </div>
          
        </section>
        
        <div class="mx-auto max-w-md box text-center mt-5">

              <select id="category-text" name="category-text" v-model="input_category" class="text-center inset-y-0 mt-10 rounded-md border-0 bg-transparent py-3 pl-10 pr-8 text-gray-500 focus:ring-2 focus:ring-black ">
                <option v-if="!input_category" :value="null" disabled selected hidden>Category</option>
                <option class="items-center" v-for="category in categories" :key="category" :value="category"> {{ category }} </option>
              </select> 
              <button data-theme="dim" v-if="!newCategoryActive" @click="addNewCategory" class="relative btn  font-light absolute mt-10 mx-2 right-0 px-4 py-1.5 text-white rounded-md">New Category</button>
        </div>

  
       
        <div class="m-auto p-auto max-w-lg mt-10 text-center" >
          <h4>Date Due</h4>
          <input type="datetime-local" name="date" v-model="input_date">
          <br>
          <button data-theme="dim" type="submit" class=" mt-2 text-center btn btn-active btn-neutral text-white">Add task</button>
        </div>
        
      </form>
    </section>

    <div class="todo-list flex justify-center gap-x-10 items-center text-center  mb-3">
      <p class="ml-16 pl-12"></p>
      <p class="ml-16 pl-12"></p>
      <p class="ml-16 pl-12"></p>
      <p class="ml-16 pl-12"></p>
      <h3>TODO list</h3>

      <button  data-theme="dim" class="btn btn-active btn-neutral text-white" @click="toggleShowDone">Show {{ showDone ? 'All' : 'Done' }} Todos</button>

    </div>
 

    <div data-theme="dim" class="bg-white text-black overflow-x-none flex preview border-base-300 rounded-box shadow flex min-h-[6rem] min-w-[18rem] max-w-4xl flex-wrap items-center justify-center gap-2 overflow-x-hidden p-4 [border-width:var(--tab-border)]  m-auto undefined">
      <table class="table  text-black  text-base divide-y divide-gray-100 table-md "  >
        <!-- head -->
        <thead class="text-black text-base font-normal text-center">
          <tr >
            <th class="font-normal">Status</th>
            <th class="font-normal ">Task</th>
            <th class="font-normal">Due Date</th>
            <th class="font-normal">Time Remain</th>
            <th class="font-normal"></th>
            <th></th>
          </tr>
        </thead>
        <tbody v-for='todo in filteredTodos' :key="todo.dueDatetimestamp" :class="`todo-item ${todo.done && 'done'}` ">
          <!-- row 1 -->
          <tr :class="{ 'todo-content': true, 'time-out': todo.days <= 0 && todo.hours <= 0 && todo.minutes <= 0 }" class="text-center">
            <td >
              <label>
                <input type="checkbox" v-model="todo.done" class="checkbox shadow ">
              </label>
            </td>
            <td >
              <input type="text" v-model="todo.content" :readonly="!todo.isEditing" class="text-center bg-white"/>
            </td>
            <td>
              <input type="datetime-local" v-model="todo.dueDate" :readonly="!todo.isEditing" class="ml-3 text-center bg-white"/>
            </td>
            <td class="join gap-x-2">
              <p v-if="todo.days > 0">{{ todo.days }} วัน </p>
              <p v-if="todo.hours > 0">{{ todo.hours }} ชั่วโมง </p>
              <p v-if="todo.minutes > 0">{{ todo.minutes }} นาที</p>
              <p v-if="todo.days <= 0 && todo.hours <= 0 && todo.minutes <= 0">Time Off</p>
            </td>
            <th></th>
            <th>
              <div class="flex">
                <button v-if="!todo.isEditing" class="edit btn btn-xs text-white font-light" @click="enableEdit(todo)">Edit</button>
                <button v-if="todo.isEditing" class="save mx-2 btn btn-xs text-white font-light" @click="saveEdit(todo)">Save</button>
                <button v-if="todo.isEditing" class="cancel btn btn-xs text-white font-light" @click="cancelEdit(todo)">Cancel</button>
                <button class="delete mx-2 btn btn-xs btn-outline btn-secondary font-light " @click="removeTodo(todo)">Delete</button>
              </div>
              
            </th>
            
          </tr>
          
        </tbody>
        <!-- foot -->
        <tfoot>
          
        </tfoot>
        
      </table>
    </div>


  </main>
</template>

<style>

  .time-out td {
    opacity: 0.5;
  }
  .greeting{
    font-size: 20px;
  }
  *{
    transition: all 1s ease-in-out;
  }
</style>