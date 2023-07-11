<template>
  <div>
    <h1>Attendance</h1>

    <!-- Task Input -->
    <input v-model="taskInput" placeholder="Add a Name" />
    <input v-model="timeInput" placeholder="Add a Time" />
    <button @click="addTask">Add</button>

    <!-- Task List -->
    <ul>
      <li v-for="task in tasks" :key="task.id">
        <div>
          <span>{{ task.name }}</span>
          <span>{{ task.time }}</span>
        </div>
        <div v-if="task.editing">
          <input v-model="task.updatedName" placeholder="New Name" />
          <input v-model="task.updatedTime" placeholder="New Time" />
          <button @click="updateTask(task)">Update</button>
          <button @click="cancelEdit(task)" class="button-delete">Cancel</button>
        </div>
        <div v-else>
          <button @click="startEdit(task)">Edit</button>
          <button @click="deleteTask(task.id)" class="button-delete">Delete</button>
        </div>
      </li>
    </ul>
  </div>
</template>


<script>
import axios from 'axios';

export default {
  data() {
    return {
      taskInput: '',   // store user name input
      timeInput: '',   // store user time input
      tasks: []        // hold tasks fetched from API
    };
  },

  methods: {
    async fetchTasks() {
  try {
    const response = await axios.get('http://localhost:8000/api/v1/tasks');
    this.tasks = response.data.data.map(task => ({
      ...task,
      editing: false,
      updatedName: '',
      updatedTime: ''
    }));
    console.log(this.tasks);
  } catch (error) {
    console.error(error);
  }
},

async addTask() {
  if (this.taskInput && this.timeInput) {
    try {
      const response = await axios.post('http://localhost:8000/api/v1/tasks', {
        name: this.taskInput,
        time: this.timeInput,
        updatedName: '',
        updatedTime: '',
        editing: false
      });

      const newTask = response.data.data;
      this.tasks.push(newTask);

      this.taskInput = '';
      this.timeInput = '';
    } catch (error) {
      console.error(error);
    }
  }
},

    startEdit(task) {      // edit function
      task.editing = true;
      task.updatedName = task.name;
      task.updatedTime = task.time;
    },
    cancelEdit(task) {    // cancel function
      task.editing = false;
      task.updatedName = '';
      task.updatedTime = '';
    },
    async updateTask(task) {
  try {
    const { updatedName, updatedTime } = task;
    const response = await axios.put(
      `http://localhost:8000/api/v1/tasks/${task.id}`,
      {
        name: updatedName || task.name,
        time: updatedTime || task.time
      }
    );
    const updatedTask = response.data.data;
    const taskIndex = this.tasks.findIndex(t => t.id === updatedTask.id);
    if (taskIndex !== -1) {
      this.tasks.splice(taskIndex, 1, updatedTask);
    }
    task.editing = false;
    task.updatedName = '';
    task.updatedTime = '';
  } catch (error) {
    console.error(error);
  }
},

    async deleteTask(taskId) {    // delete function
      try {
        await axios.delete(
          `http://localhost:8000/api/v1/tasks/${taskId}`
        );
        this.tasks = this.tasks.filter((task) => task.id !== taskId);
      } catch (error) {
        console.error(error);
      }
    },
  },

  created() {
    this.fetchTasks();
  },
}
</script>




<style scoped>      
.container {    
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background-color: #f3f3f3;
  margin: auto;
  text-align: center;
}

h1 {
  margin-bottom: 20px;
}

input,
button {
  margin: 10px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  margin-bottom: 10px;
}

button {
  cursor: pointer;
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
}

.button-delete {
  background-color: #ff0000; /* Red color */
}

.edit-container {
  display: flex;
  align-items: center;
  justify-content: center;
}

.edit-container input {
  margin-right: 10px;
}

.edit-container button {
  margin-right: 5px;
}
</style>