<template>
    <div>
        <form @submit.prevent="onSubmit" v-if="userLogin">
            <div>
                <label for="title">Title</label>
                <input type="text" name="title" id="title" v-model="userTitle" required>
            </div>
            <div>
                <label for="description">Description</label>
                <input type="text" name="description" id="description" v-model="userDescription">   
            </div>
            <div>
                <label for="deadlineDate">Date</label>
                <input type="datetime-local" name="deadlineDate" id="deadlineDate" v-model="userDeadlineDate">   
            </div>
            <div>
                <label for="completed">Completed</label>
                <select name="completed" id="completed" v-model="userCompleted">
                    <option value="N" selected>no</option>
                    <option value="y">yes</option>
                </select>
            </div>
            <div>
                <button type="submit">Create task</button>
            </div>
        </form>   
    </div>
</template>

<script>
export default {
    emits: ['add:task'],
    props: {
        url: String,
        userLogin: Boolean,
        jwt : String
    },
    data() {
        return {
            userTitle: '',
            userDescription: '',
            userDeadlineDate: '',
            userCompleted: ''
        }
    },
    computed: {
        task: function() {
            const temp = {}
            temp.title = this.userTitle
            this.userDescription ? (temp.description = this.userDescription) : null
            this.userDeadlineDate ? 
            (temp.deadline = this.userDeadlineDate.split("T").join(' ') + ':00') : null
            this.userCompleted ? (temp.completed = this.userCompleted) : null
            return temp
        }
    },
    methods: {
        async onSubmit() {
            try{
                const res = await fetch(this.url, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': this.jwt
                    },
                    body: JSON.stringify(this.task)
                })
                const data = await res.json()
                console.log(data)
                this.$emit('add:task', data)
            }catch(err) {
                console.err(err)
            }
        }
    }
}
</script>

<style>

</style>