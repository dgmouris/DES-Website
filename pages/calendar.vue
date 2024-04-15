<script setup lang="ts">
import { ref, onMounted} from 'vue';

// import the calendar.
import VueCal from 'vue-cal' 
import 'vue-cal/dist/vuecal.css'

const group = {
  name: 'Calendar',
  items: []
}

const title = 'Calendar'
const description = 'List of all upcoming events in Edmonton'

// Learn how to use this piece.
useServerSeoMeta({
  title,
  description,
})


onMounted(()=> {
  // fetch the events using the google api.
  // change the events ref to use the google api
  // profit.
})

// have some events in a ref.
const events = ref([
  {
    start: '2024-04-19 10:35',
    end: '2024-04-19 11:30',
    title: 'Doctor appointment'
  },
  {
    start: '2024-04-19 18:30',
    end: '2024-04-19 19:15',
    title: 'Dentist appointment'
  },
  {
    start: '2024-04-20 18:30',
    end: '2024-04-20 20:30',
    title: 'Crossfit'
  },
  {
    start: '2024-04-21 11:00',
    end: '2024-04-21 13:00',
    title: 'Brunch with Jane'
  },
  {
    start: '2024-04-21 19:30',
    end: '2024-04-21 23:00',
    title: 'Swimming lesson'
  },
  {
    start: '2019-09-30 19:30',
    end: '2019-09-30 23:00',
    title: 'Swimming lesson'
  },
  {
    start: "2024-04-19 12:00",
    end: "2024-04-19 14:00",
    title: "LUNCH",
    class: "lunch",
    background: true
  },
  {
    start: "2024-04-20 12:00",
    end: "2024-04-20 14:00",
    title: "LUNCH",
    class: "lunch",
    background: true
  }
])


</script>


<template>
  <main class="mt-8 max-w-lg mx-auto px-8">
    <ProseH1 class="mb-8 text-center">
      Events Calendar
    </ProseH1>
    <vue-cal :events="events"></vue-cal>
  </main>
</template>

<style>
  .vuecal {
    height: 90vh;
    width: 90vh;
  }
</style>