<script setup lang="ts">
    import { defineComponent, ref, onMounted, onUnmounted, inject, computed, watchEffect, watch } from 'vue';
    import type { Ref } from 'vue'
    import { useSession } from "@/model/session"
    import { useRoute } from 'vue-router'
    // import type UserStore from '@/stores/user';
    import type UserStore from '@/stores/user';
    import { getActivities, type Activity } from '@/model/activities';
    import NavBar from './NavBar.vue';
    import Notifications from './Notifications.vue';

    const route = useRoute()
    const activities = ref({} as Activity[]) 
    getActivities().then((data) => {
        activities.value = data.data;
    });

    // const activitiesRaw = computed(() => {
    //     return activities.value ? activities.value._value : undefined;
    // })

    const menuActiveOnShortScreen = ref(false);
    const buttonClickRef = ref<HTMLElement | null>(null);
    
    const menuComponentRef = ref(NavBar);


    const props = defineProps({
        menuRef: {
            type: Object as () => HTMLElement,
            required: true,
        },
    });

    // update-menu-ref is the name of the event we're going to emit to 
    const emit = defineEmits(['update-menu-ref']);
    

    // const toggleMenu = () => {
    //   menuRef.value.classList.toggle('show');
    // };
    function toggleMenu() {
        const screenWidth = window.innerWidth;
        const menuElem = props.menuRef
        if (screenWidth <= 769) {
            // If we're in a smaller screen
            if (menuActiveOnShortScreen.value == false)
            {
                menuElem.style.left = '0';
                menuActiveOnShortScreen.value = true;
                

            }
            else if (menuActiveOnShortScreen.value == true)
            {
                menuElem.style.left = '-270px';
                menuActiveOnShortScreen.value = false;
                // if (buttonClickRef.value != null)
                // {
                //     buttonClickRef.value.style.position = 'absolute';
                //     buttonClickRef.value.style.right = '500px';
                // }
            }
            

        }
        
        console.log(menuActiveOnShortScreen.value)
        
        
        
    }

    watchEffect(() => {
        const screenWidth = window.innerWidth;
        const menuElem = props.menuRef
        

        if (menuActiveOnShortScreen.value == true)
        {
            if (buttonClickRef.value != null)
            {
                buttonClickRef.value.style.position = 'absolute';
                buttonClickRef.value.style.left = '500px';
                buttonClickRef.value.style.bottom = "-7px";
            }
            

        }
        else 
        {
            if (buttonClickRef.value != null)
            {
                buttonClickRef.value.style.position = 'absolute';
                buttonClickRef.value.style.left = '0px';
                buttonClickRef.value.style.bottom = "-7px";
            }
            // if (screenWidth > 769)
            // {
            //     menuElem.style.left = '0px';
            //     emit('update-menu-ref', menuElem);
            // }
            // else
            // {
            //     menuElem.style.left = '-270px';
            //     emit('update-menu-ref', menuElem);
            // }
            // 

        }

        if (screenWidth > 769) {
            menuElem.style.left = '0px';
            emit('update-menu-ref', menuElem);
            menuActiveOnShortScreen.value = false;
        }  
        

        
        
        // $emit('update-menu-ref', menuElem);

        // your watchEffect code here
        // console.log(props.menuRef)
        // if (screenWidth <= 769) {
            // props.menuRef.style.left = '-270px';
        // menuElem.classList.remove('menu');
        // Show
        // menuElem.style.left = '0px';
        // console.log(menuElem.style.left);

        // Hide
        // menuElem.style.left = '-270px';

        // Since this is a reference, we can't directly modify the real element. Thus, we emit it
        // emit('update-menu-ref', menuElem);

        // } else {
        //     menuElem.classList.remove('menu');
        // }
    })

    const userStore = inject('userStore') as typeof UserStore;
    const pfp = ref("")
        
    // watchEffect(async () => {
    //     const result = await userStore.getUserName();
    //     // console.log(result);
    //     getActivities().then((data) => {
    //     //         activities.value = data;
    //     // activities.forEach((activity) => {
    //         console.log(data[0].username)
            
    //         if (data[0].username == result)
    //         {
    //             pfp.value = data[0].profilePicture;
    //         }
    //     });
    // });


    
  
   


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

        // console.log(activitiesRaw.value);


        getActivities().then((data) => {
        //         activities.value = data;
        // activities.forEach((activity) => {
            for(const item of data.data)
            {
                // console.log(item)
                if (item.username == result)
                {
                    pfp.value = item.profilePicture;
                }
            }
            
            
        });
    }

    // async function updatePfp()
    // {
    //     const result = await userStore.getUserName();

    //     getActivities().then((data) => {
    //     //         activities.value = data;
    //     // activities.forEach((activity) => {
    //         // console.log(data)
            
    //         if (data.data[0].username == result)
    //         {
    //             pfp.value = data.data[0].profilePicture;
    //         }
    //     });

    // }

    // setInterval(updatePfp, 1);


    logUserName();

    const dynamicLabel = computed(() => {
        const route = useRoute()
        if (route.path !== '/') {
            return ''
        } else {
            return username.value
        }
    })
    // onMounted(async () => {
    //     const result = await userStore.getUserName();


    //     getActivities().then((data) => {
    //     //         activities.value = data;
    //     // activities.forEach((activity) => {
    //         // console.log(data)
            
    //         if (data[0].username == result)
    //         {
    //             console.log("pfp user: " + data[0].username)
    //             pfp.value = data[0].profilePicture;
    //         }
    //     });
    // })
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
            <div class="menuBar" @click="toggleMenu" style="position: fixed;">
                <i class="fas fa-bars" ref="buttonClickRef"></i>
            </div>
            

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
                <!-- <i class="fa fa-bell" aria-hidden="true"></i> -->
                <Notifications />
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