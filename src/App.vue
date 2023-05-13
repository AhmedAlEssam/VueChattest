<script>
import axios from 'axios';
export default {
  data() {
    return {
      columns: [],
      newTaskTitle: '',
      newColumnTitle: '',
    };
  },
  created() {
    this.getColumns();
  },
  methods: {
    getColumns() {
      axios.get('http://localhost:3000/api/columns').then(response => { this.columns = response.data; }).catch(error => { console.error(error); });
    },

    addColumn() {
      axios.post('http://localhost:3000/api/addColumn', {  title: this.newColumnTitle })
        .then(response => { 
          if (response.data) {
            this.columns.push(response.data);
            this.newColumnTitle = '';
          }
        }).catch(error => {
          console.error(error);
        });
    },

    addTask(columnId) {
      axios.post('http://localhost:3000/api/task', { columnId, title: this.newTaskTitle })
        .then(response => {
          const task = response.data;
          const column = this.columns.find(c => c._id === columnId);
          column.tasks.push(task);
          this.newTaskTitle = '';
        }).catch(error => {
          console.error(error);
        });
    },
    deleteTask(taksId,columnId) {
      axios.post('http://localhost:3000/api/deleteTask', { taksId ,columnId})
        .then(response => { 
          this.columns= response.data;
          
        }).catch(error => {
          console.error(error);
        });
    },


    dragStartHandler(taskId, sourceColumnId) { 
      event.dataTransfer.setData('text/plain', JSON.stringify({ taskId, sourceColumnId }));
    },
    dropHandler(columnId) { 
      const { taskId, sourceColumnId } = JSON.parse(event.dataTransfer.getData('text/plain'));
      axios.patch(`http://localhost:3000/api/drag/${taskId}`, { columnId ,sourceColumnId,taskId}).then(response => {
        const sourceColumn = this.columns.find(c => c._id === sourceColumnId);
        const taskIndex = sourceColumn.tasks.findIndex(t => t._id === taskId);
        sourceColumn.tasks.splice(taskIndex, 1);
        const targetColumn = this.columns.find(c => c._id === columnId);
        targetColumn.tasks.push(response.data);
      })
        .catch(error => {
          console.error(error);
        });
    },
  },
};
</script>

<template>
  <div class="board">
    <div class="columns">
      <div v-for="(column , index) in columns" :key="column._id" class="column min-w-[300px]" @drop="dropHandler(column._id)" @dragover.prevent>
        <h3>{{ column.title }}</h3>
        <div class="tasks">
          <div v-for="task in column.tasks" :key="task._id" class="task" draggable="true"
            @dragstart="dragStartHandler(task._id, column._id)">{{ task.title }} 
            <span @click="deleteTask(task._id,column._id)" class="text-white opacity-0 hover:opacity-100" ><svg xmlns="http://www.w3.org/2000/svg" version="1.0" width="50.000000pt" height="50.000000pt" viewBox="0 0 50.000000 50.000000" preserveAspectRatio="xMidYMid meet">
              <g transform="translate(0.000000,50.000000) scale(0.100000,-0.100000)" fill="#012030" stroke="none">
              <path fill="#D2042D" d="M155 456 c-60 -28 -87 -56 -114 -116 -36 -79 -19 -183 42 -249 33 -36 115 -71 167 -71 52 0 134 35 167 71 34 37 63 110 63 159 0 52 -35 134 -71 167 -37 34 -110 63 -159 63 -27 0 -65 -10 -95 -24z m180 -15 c128 -58 164 -223 72 -328 -101 -115 -283 -88 -348 52 -79 171 104 354 276 276z"/>
              <path fill="#D2042D" d="M186 328 c4 -7 16 -25 26 -40 24 -34 23 -45 -7 -84 -27 -36 -30 -44 -15 -44 5 0 20 16 34 36 l24 36 24 -36 c13 -20 29 -36 36 -36 14 0 11 7 -20 52 -24 34 -23 45 7 84 27 36 30 44 15 44 -5 0 -20 -16 -34 -36 l-24 -36 -24 36 c-13 20 -29 36 -36 36 -8 0 -10 -5 -6 -12z"/>
              </g>
              </svg></span>
            </div>
        </div>
        <div v-if="index==0" class="add-task">
          <input type="text" @keyup.enter="addTask(column._id)" v-model="newTaskTitle" class="rounded" placeholder="Enter task title" />
          <button @click="addTask(column._id)" class="rounded">Add Task</button>
        </div>
      </div>
      <div class="column">
        <h3>اضافة عمود جديد</h3>
        <div class="add-column ">
          <input type="text" v-model="newColumnTitle" class="rounded" placeholder="Enter column title" />
          <button @click="addColumn()" class="rounded">Add Column</button>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped>
.board {
  width: 100%;
  margin: 0 auto;
  max-width: none;
}

.columns {
  display: flex;
  justify-content: space-between;
}

.column {
  flex: 1;
  background-color: #f0f0f0;
  margin: 0 10px;
  padding: 10px;
  text-align: center;
}

.column h3 {
  margin-bottom: 10px;
}

.tasks {
  margin-bottom: 10px;
}

.task {
  background-color: #ffffff;
  border: 1px solid #cccccc;
  padding: 5px;
  margin-bottom: 5px;
  cursor: grab;
  position: relative;
}
.task span { 
  top: 5px;
  right: 5px;
  cursor: pointer;
  display: inline-block;
  width: 25px;
  position: absolute;
  height: 25px;
}
span svg { 
  width: 100%;
  height: 100%;
}
.add-task,
.add-column {
  display: flex;
  align-items: center;
  flex-direction: column;
}

.add-task input,
.add-column input {
  flex: 1;
  margin-right: 10px;
}

.add-task button,
.add-column button { 
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}
</style>