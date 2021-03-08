<template>
    <div>
        <div v-if="loginUsername.length > 0">
            <p class="hello">Hello {{loginUsername}}</p>
            <button class="logout" @click="logout">Logout</button>
        </div>
        <form @submit.prevent="onSubmit" v-else>
            <div>
                <label for="username">Username</label>
                <input type="text" name="username" id="username" v-model="username">
            </div>
            <div>
                <label for="password">Password</label>
                <input type="password" name="password" id="password" v-model="password">  
            </div> 
            <div>
                <button type="submit">Login</button>
            </div>
        </form>   
    </div>
</template>

<script>
export default {
    emits: ['login', 'setJwt'],
    props: {
        url: String
    },
    data() {
        return {
            username: '',
            password: '',
            loginUsername: ''
        }
    },
    created() {
        this.loginOnLoad()
    },
    methods: {
        setCookie(jwt) {
            const d = new Date();
            d.setTime(d.getTime() + 60*60*24*1000);
            const expires = "expires=" + d.toUTCString();
            document.cookie = `auth=${jwt};${expires};path=/`
            this.jwt = jwt
        },
        async onSubmit() {
            try {
                const res = await fetch(this.url + '/user/login', {
                    method: 'POST',
                    header: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({username: this.username, password: this.password})
                })
                const data = await res.json()
                if(data.data.jwt) {
                    this.setCookie(data.data.jwt)
                    this.loginUsername = data.data.username
                    this.$emit('login', true)
                }
            }catch(err) {   
                console.error(err)
            }
            this.username = ''
            this.password = ''
        },
        async loginOnLoad() {
            const cookie = document.cookie
            const jwt = cookie.split('auth=')[1]
            try {
                const res = await fetch(this.url + '/user/confirm', {
                    method: 'GET',
                    headers: {
                        'Authorization': `${jwt}`
                    }
                })
                const data = await res.json()
                if(data.success) {
                    this.loginUsername = data.data.authentication.username
                    this.$emit('login', true)
                    this.$emit('setJwt', jwt)
                }
            }catch(err) {
                this.$emit('login', false)
            }
        },
        logout() {
            document.cookie = `auth=;expires=1;path=/`
            this.loginUsername = ''
            this.$emit('login', false)
        }
    }   
}
</script>

<style scope>
    form {
        margin-bottom: 50px;
    }

    div {
        margin-bottom: 10px;
    }

    .hello {
        display: inline-block;
        margin-right: 10px;
    }
</style>