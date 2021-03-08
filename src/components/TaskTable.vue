<template>
    <div>
        <p v-if="deleteStatus === true" class="deleteMessage">Delete successfully</p>
        <p v-if="patchStatus === true" class="patchMessage">Patch successfully</p>
        <p v-if="errorStatus === true" class="errorMessage">Error occured</p>
        <p v-if="deleteVisual === true">Deleting task...</p>
        <p v-if="patchVisual === true">Patching task...</p>
        <table :class="enableModify"> 
            <tr>
                <th>Title</th>
                <th>Description</th>
                <th>Deadline</th>
                <th>Completed</th>
            </tr>
            <tr v-for="(task, index) in tasks" 
                :key="index" 
                @click="tripleClick(task,index)"
            >
                <td>
                    <input type="text" v-if="enableModify === index" 
                        v-model="task.title"
                        @click.stop>
                    <button v-if="enableModify === index" @click="comfirmModify(task, index)">Confirm</button>
                    <p>{{task.title}}</p>
                </td>
                <td>
                    <input type="text" v-if="enableModify === index" 
                        v-model="task.description"
                        @click.stop>
                    <p>{{task.description}}</p>
                    </td>
                <td>
                    <input type="datetime-local" v-if="enableModify === index" 
                        v-model="task.deadline"
                    @click.stop>
                    <p>{{task.deadline}}</p>
                </td>
                <td>
                    <select name="completed" id="completed"
                        v-if="enableModify === index"
                        v-model="task.completed"
                    >
                        <option value="N" selected>no</option>
                        <option value="Y">yes</option>
                    </select>
                    <button v-if="enableModify === index" @click="deleteTask(task, index)">Delete task</button>
                    <p>{{task.completed}}</p>
                </td>
            </tr>
        </table>
    </div>
</template>

<script>
export default {
    emits: ['delete:task', 'patch:task'],
    props: {
        url: String,
        tasks: Array,
        userLogin: Boolean,
        deleteStatus: Boolean,
        patchStatus: Boolean,
        errorStatus: Boolean
    },
    data() {
        return {
            count: 0,
            timer: null,
            date: '',
            time: '',
            deleteVisual: false,
            patchVisual: false,
            enableModify: -1,
        }
    },
    watch: {
        patchStatus(value) {
            if(value === true) {
                this.enableModify = -1
                this.patchVisual = false
            }
        },
        deleteStatus(value) {
            if(value === true) {
                this.enableModify = -1
                this.deleteVisual = false
            }
        }
    },
    methods: {
        tripleClick(task, index) {
            if(this.userLogin === false) return
            this.count = this.count + 1
            if(this.count === 1) {
                this.timer = setTimeout(() => {
                    this.count = 0
                    clearTimeout(this.timer)
                }, 500)
            } else {
                if(this.count === 3)
                    this.enableModify = index
                    task.deadline = new Date(task.deadline).toISOString()
            }
        },
        deleteTask(task, index) {
            this.$emit('delete:task', {task: task, index: index})
            this.deleteVisual = true
        },
        comfirmModify(task, index) {
            if(task.deadline.length < 19) 
                task.deadline = task.deadline.split("T").join(' ') + ':00'
            this.patchVisual = true
            this.$emit('patch:task', {task: task, index: index})
        }
    }
}
</script>

<style scoped>

table {
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

td, th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {
  background-color: #ececec;
}

p {
    margin: 0
}

.deleteMessage,
.patchMessage,
.errorMessage {
    font-weight: 700;
}
table input,
table select,
table button {
    margin-bottom: 5px;
}


</style>