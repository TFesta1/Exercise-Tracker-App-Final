<script setup lang="ts">
    import { ref, reactive, inject, computed, onMounted, watch } from 'vue'
    import { type Workout, removeWorkout, addToWorkout, getUserWorkouts, getUsers } from '@/model/workouts';
    import type UserStore from '@/stores/user';
    import { getActivities, type Activity } from '@/model/activities';
    import listWorkouts from '@/components/listWorkouts.vue';

    const activities = ref({} as Activity[])
    
    const searchTerm = ref("")

    

    getActivities().then((data) => {
        activities.value = data.data;
    });
    const streak = ref(0);
    const rest = ref(0);
    


    const clickedAddWorkout = ref(false)
    // Switch the value whenever we either click AddWorkout, or "Close" so that we can open the modal and hide the add-workout button accordingly
    async function addWorkout() {
        clickedAddWorkout.value = !clickedAddWorkout.value;
        // await getUsername() //Updates the table
    }
    // const workouts = getWorkouts();
    
    const state = reactive({
        newWorkout: {
            username: '',
            workoutType: '',
            description: '',
            intensity: ''
        }
    });

    const changingWorkouts = ref({} as Workout[]);

    const router = inject('router') as any;
    const userStore = inject('userStore') as typeof UserStore;
    const username = ref("")
    async function getUsername() {
        const result = await userStore.getUserName();
        username.value = result;
        getUserWorkouts(result).then((data) => {
            changingWorkouts.value = data.data;
        });
        getActivities().then((data) => {
            const activities : Activity[] = data.data;

            activities.forEach((activityArray: Activity) => {
                // console.log(activityArray.username)
                // activityArray.forEach((activity: Activity) => {
                //     console.log(activity.username);
                // });
                // for(let i = 0; i < activities.length; i++)
                // {
                //     const activity : Activity = activityArray[i]
                //     console.log(activity.username)
                //     console.log("hi")

                //     // console.log(item)
                if (activityArray.username == result)
                {
                    // console.log(activityArray.username)
                    streak.value = activityArray.streak;
                    rest.value = activityArray.rest;
                    // console.log(streak.value)
                }
                // }
                // activityArray.forEach((activity: Activity) => {
                // //     console.log(activity.username);
                // });
            });
            
            
                
                // console.log(data.data[item].values)
                
                // console.log("hi")
                // if (data.data[item] == result)
                // {
                //     streak.value = data[item].streak;
                //     rest.value = data[item].rest;
                // }
        });            
    }
    const allFilteredUsernames = ref([] as String[])

    async function filterUsernames(searchTerm: string) {
      if(searchTerm == "") {
        allFilteredUsernames.value = [];
        return;
      }
      let matches = 0;
      const usernames = await getUsers().then((data) => {
        return data.data;
      });

      // Filter the usernames based on the search term and matches limit
      const filteredUsernames = usernames.filter(username => {
        console.log(username);
        if (username.toLowerCase().includes(searchTerm.toLowerCase()) && matches < 10) {
          matches++;
          return true;
        }
        return false;
      });

      allFilteredUsernames.value = filteredUsernames;
    }

    const searchUsers = computed(() => {
      return allFilteredUsernames.value;
    });

    watch(searchTerm, filterUsernames, { immediate: true });


    // console.log(searchUsers.value, 'searchUsers')
    
    

    

    async function userWorkouts() {
        
        const user = await userStore.getUserName();
        // changingWorkouts.value = getUserWorkouts(user);
        getUserWorkouts(user).then((data) => {
            changingWorkouts.value = data.data;
        });

        // removeWorkout

        return changingWorkouts;
    };
    const usernames = ref()

    onMounted(async () => {
        // console.log("hello")
        getUsers().then((data) => {
            console.log(data.data)
            usernames.value = data.data;
        });
        console.log(usernames.value)
        await getUsername()
    });

    const form = reactive({
        name: '',
        description: '',
        intensity: '',
        tag: ''
    })

    async function onSubmit() 
    {
        // console.log(form.name)
        // console.log(form.description)
        // console.log(form.intensity)
        if(form.name == '' || form.description == '' || form.intensity == '') {
            alert('Please fill out all fields')
            return
        }
        const workout = {
            username: await userStore.getUserName(),
            workoutType: form.name,
            description: form.description,
            intensity: form.intensity
        }
        await addToWorkout(workout)
        await getUsername()
    }

    // async function asyncRemove(i: number){
    //     await removeWorkout(i)
    //     getUserWorkouts(await userStore.getUserName()).then((data) => {
    //         changingWorkouts.value = data.data;
    //     });
    //     await getUsername() //Updates the table
    //     // getUsername()
    // }

    const showContent = computed(() => {
      return changingWorkouts.value !== null && username !== undefined;
    });
    getUsername()
    

</script>

<template>
    <div class="values">
        <div class="val-box">
            <i class="fas fa-users"></i>
            <div>
                <h3>Legs</h3>
                <span>Workout Today</span>
            </div>
        </div>
        <div class="val-box">
            <i class="fas fa-level-up"></i>
            <div>
                <h3>Normal</h3>
                <span>Intensity</span>
            </div>
        </div>
        <div class="val-box">
            <i class="fas fa-globe"></i>
            <div>
                <h3>In {{ rest }} Days</h3>
                <span>Rest Day</span>
            </div>
        </div>
        <div class="val-box">
            <i class="fas fa-thermometer-three-quarters"></i>
            <div>
                <h3>For {{ streak }} Days</h3>
                <span>Streak</span>
            </div>
        </div>
        <!-- display:flex to show it, use Vue3 to show/hide it -->
        <div class="modal-container" :style="{ display: clickedAddWorkout ? 'flex' : 'none' }">
            <div class="modal">
                <h2>Add Workout</h2>
                <form @submit.prevent="onSubmit">
                    <label>
                        Name:
                        <input v-model="form.name" type="text" name="name" autocomplete="off" />
                    </label>
                    <label>
                        Description:
                        <textarea v-model="form.description" name="description" class="description-box"></textarea>
                    </label>
                    <label>
                        Intensity:
                        <select class="intensity-dropdown" v-model="form.intensity">
                            <option>High</option>
                            <option>Normal</option>
                            <option>Easy</option>
                        </select>
                    </label>

                    <label>
                        Tag:
                        <input v-model="searchTerm" type="text" name="tag" autocomplete="on" class="move" />
                    </label>
                    <ul v-if="allFilteredUsernames.length">
                        
                        <li class="usernames"
                            v-for="name in allFilteredUsernames"
                        >
                            {{ name }}
                        </li>
                    </ul>
                    <button type="submit" class="submit-modal" @click="addWorkout">Submit</button>
                </form>
                <button class="close-modal" @click="addWorkout">Close</button>
            </div>
            <div class="modal-background"></div>
        </div>
        <h3 class="i-name">
            <button class="add-workout" @click="addWorkout" :style="{ display: clickedAddWorkout ? 'none' : 'flex' }">
                <i></i>
                Add Workout
            </button>
        </h3>
        
        
    </div>
    
    <!-- <div class="workoutValues" v-for="workout,i in changingWorkouts">
        <h2>{{ workout.workoutType }}</h2>
        <br>

        <p>{{ workout.description.replace(/,/g, ', ') }}</p>
        
        
        <div class="intensity">
            <br>
            Intensity: {{ workout.intensity }}
        </div> 

        <button class="trash-button" @click="asyncRemove(i)" @change="userWorkouts()">
            <span class="icon">
                <i class="fas fa-trash"></i>
            </span>
        </button>
    </div> -->

    <!-- <div v-if="showContent"> -->
    <listWorkouts :username="username" :showTrash="true" :changeWorkoutsUpdate="changingWorkouts"></listWorkouts>
    <!-- </div> -->
    
    
</template>

<style scoped>
.intensity-dropdown
{
    width: 100%;
    height: 30px;
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 5px;
    margin-bottom: 10px;
}

.description-box 
{
    width: 100%;
    height: 90px;
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 3px;
}

.trash-button {
    background-color: #f44336;
    border: none;
    color: white;
    padding: 5px 10px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
    border-radius: 5px;
    position: relative;
    width: 150px;
    position: relative;
    right: -2%;
    top: 10px;
}

.close-modal {
    background-color: #f44336;
    border: none;
    color: white;
    padding: 5px 10px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
    border-radius: 5px;
    position: relative;
    top: -20px;
    left: 220px;
}

.move {
    position: relative;
    bottom: -10px;
}

.usernames {
  padding: 10px;
  border: 1px solid #ccc;
  background-color: #f5f5f5;
  color: black;
  list-style: none;
  
}

</style>