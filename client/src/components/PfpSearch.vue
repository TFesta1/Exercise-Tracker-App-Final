<script setup lang="ts">
    import { defineComponent, ref, onMounted, onUnmounted, inject, computed } from 'vue';
    import { useSession, login } from "@/model/session"
    import { useRoute } from 'vue-router'
    // import type UserStore from '@/stores/user';
    import type UserStore from '@/stores/user';
    import { getActivities } from '@/model/activities';
    const route = useRoute()
    const activities = getActivities()
    // import userStore from "@/stores/user";


    
  
   


    const isDropdownOpen = ref(false); //true/false dictating of t he dropdoown is open or not
    const profileDropdown = ref<HTMLElement | null>(null); //ref to the profile. It's on the button, but we didn't really make the pfp dropdown yet

    const handleDocumentClick = (event: MouseEvent) => {
        // If the dropdown is open, and the profileDropdown is open, and the profileDropdown does not contain the target (the thing you clicked on), then close the dropdown
      if (isDropdownOpen.value && profileDropdown.value && !profileDropdown.value.contains(event.target as HTMLElement)) {
        isDropdownOpen.value = false;
      }
    };

    onMounted(() => {
      profileDropdown.value = document.querySelector('.profile-dropdown') as HTMLElement; //assigning the profileDropdown to the profileDropdown ref
      document.addEventListener('click', handleDocumentClick);
    });

    onUnmounted(() => {
      document.removeEventListener('click', handleDocumentClick); //remove the event listener when the component is unmounted (removed from the scene)
    });

    function logout() {
        session.user = null;
        // console.log(`Logged out ${session.user}`)
    }
    const userStore = inject('userStore') as typeof UserStore;
    const pfp = ref("")

    onMounted(userStore.getUser)
    const username = ref("Hello ")
    const usernameRaw = ref("")

    async function logUserName() {
        const result = await userStore.getUserName();
        username.value += result
        // usernameRaw.value = result
        // console.log(`usernameRaw ${usernameRaw.value}`); // logs "tanner"

        // Given the username, log our PFP
        // pfp.value = map()

        for (const activity of activities)
        {
            // console.log(activity.username)
            // console.log(result)
            if (activity.username == result) {
                pfp.value = activity.profilePicture
                // console.log(`pfp ${pfp.value}`)
            }
        }
    }
    logUserName();

    const dynamicLabel = computed(() => {
        const route = useRoute()
        if (route.path !== '/') {
            return ''
        } else {
            return username.value
        }
    })

    const session = useSession();


</script>

<template>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link rel="stylesheet" href="exerciseStyle.css">
        

        <!-- <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.2.1/css/all.min.css"/> -->
        <!-- jQuery -->
        <!-- <script src="https://code.jquery.com/jquery-3.6.3.min.js"></script> -->
        <title>Exercise Tracker</title>
    </head>
    <body>

        <!-- 

            :class="{ 'search': $route.path === '/search',
                                'hide-search': $route.path !== '/search'}"
         -->
        
        

    <section id="interface">
        <div class="navigation">
            <div class="">
                <!-- If the route is NOT search, hide the search bar. Otherwise, have the search bar-->
                <div :class="{ 'hide-search': $route.path !== '/search', 
                                'search': $route.path === '/search' }">
                    <!-- Icon -->
                    <i class="fa fa-search" aria-hidden="true"></i>
                    <!-- Input field -->
                    <input type="text" placeholder="Search">
                </div>
            </div>
            <div>
                <h3>{{ dynamicLabel }}</h3>
            </div>

            <!-- Bell icon and pfp on top right -->
            <div class="profile">
                <i class="fa fa-bell" aria-hidden="true"></i>
                <img :src="pfp" alt="pfp" ref="profileDropdown" @click="isDropdownOpen = !isDropdownOpen">
                
                
                <ul class="menu" v-if="isDropdownOpen">
                    <li><a href="#" @click="userStore.logout()">Logout</a></li>
                    <li><a href="#">Settings</a></li>
                </ul>
                

            </div>

                
        </div>
        

        <!-- The title -->
        <!-- <h3 class="i-name">
            <button class="add-workout">
                <i></i>
                Add Workout
            </button>
        </h3> -->

        

        

        
    </section>
        
        
    </body>
    </html>
</template>

<style scoped>

</style>