<script setup lang="ts">

</script>

<template>
    <!DOCTYPE html>
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link rel="stylesheet" href="auth.css">
        <title>Login</title>
    </head>

    <body>

    </body>
    <form action="/register" method="post">
        <div>
            <!-- {{#if error}}
            <p class="err-msg">Username or password is incorrect</p>
            {{/if}} -->
            <h1>Register</h1>
        </div>
        <div class="form-group">
            <label>Username: </label>
            <input 
                type="text" 
                name="username" 
                id="username" 
                placeholder="Create your username" 
                required/>
        </div>
        <div class="form-group">
            <label>Password:</label>
            <input 
                type="password" 
                name="password" 
                id="password" 
                placeholder="Create your password" 
                required/>
        </div>
        <!--  -->
        <div class="form-group">
            <label>Re-Type Password:</label>
            <input 
                type="password" 
                name="password" 
                id="password" 
                placeholder="Re-enter your password" 
                required/>
        </div>

        <div>
            <input type="submit" value="Login" />
        </div>

    </form>
    
</template>

<style scoped>
    @import "@/assets/auth.css";


</style>