<template>
  <div>
    <LoginForm :url="url" @login="setLogin" @setJwt="setJwt"/>
    <TaskForm :url="url" :userLogin="userLogin" :jwt="jwt" @add:task="addTask" />
    <p>Triple click to edit after login</p>
    <TaskTable 
      :url="url" 
      @delete:task="deleteTask"
      @patch:task="patchTask"
      :deleteStatus="deleteStatus" 
      :patchStatus="patchStatus"
      :errorStatus="errorStatus"
      :userLogin="userLogin" 
      :tasks="tasks"/>
    
  </div>
</template>

<script>
import TaskTable from './components/TaskTable.vue'
import TaskForm from './components/TaskForm.vue'
import LoginForm from './components/LoginForm.vue'
export default {
  name: 'App',
  components: {
    TaskTable,
    TaskForm,
    LoginForm
  },
  data() {
    return {
      url: 'https://api.taskapi.tk',
      userLogin: false,
      tasks: [],
      deleteStatus: false,
      patchStatus: false,
      errorStatus: false,
      jwt: '' 
    }
  },
  created() {
    this.getTasks()
  },
  methods: {
    setLogin(value) {
      this.userLogin = value
    },
    setJwt(jwt) {
      // save jwt after auto login
      this.jwt = jwt
    },
    addTask(data) {
      this.tasks.push(data.data.task)
    },
    async deleteTask(obj) {
      this.deleteStatus = false
      const task = obj.task
      const index = obj.index
      const res = await fetch(this.url + '/' + task.id, {
        method: 'DELETE',
        headers: {
          'Authorization': this.jwt
        }
      })
      const data = await res.json()
      if(data.statusCode === 200) {
        this.deleteStatus = true
        this.tasks = this.tasks.filter((task, i) => {
          return this.tasks[i] !== this.tasks[index]
        })
      } else{
        this.errorStatus = true
      }
    },
    async patchTask(obj) {
      this.errorStatus = false
      const task = obj.task
      const res = await fetch(this.url + '/' + task.id, {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': this.jwt
        },
        body: JSON.stringify(task)
      })
      const data = await res.json()
      if(data.statusCode === 200) this.patchStatus = true
      else this.errorStatus = true
    },
    async getTasks() {
      try {
        const res = await fetch(this.url)
        const data = await res.json()
        this.tasks = data.data
      } catch(err) {
        console.log(err)
      }        
    }
  }
}
</script>

<style>
#app {
  font-family: 'Montserrat', sans-serif;
  color: #2c3e50;
  margin-top: 10px;
  padding: 10px;
}

button {
  cursor: pointer;
}

form {
  width: 280px;
}

form input,
form select {
  float: right;
}
</style>
