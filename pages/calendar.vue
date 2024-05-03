<script setup lang="ts">
import { ref, onMounted } from 'vue';

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
  addExternalGoogleApiJSScripts()
  // 
  handleAuthClick()
  listUpcomingEvents().then((events)=> {
    console.log(events)
  })
  events.value  = [
    {
      start: '2024-04-19 10:35',
      end: '2024-04-19 11:30',
      title: 'Test to see if the update works'
    }
  ]
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


// add the credentials to the root but we're going
// have to change this in the future so that it's using
// environment variables.

const CLIENT_ID = '333010274801-dncilk7m3tqusr5ecbepjrd7trgdmtps.apps.googleusercontent.com';
const API_KEY = 'AIzaSyBB7boCn8jCthR0IOD8a1CEhMu3Z_0-Sk0';

const DISCOVERY_DOC = 'https://www.googleapis.com/discovery/v1/apis/calendar/v3/rest';
const SCOPES = 'https://www.googleapis.com/auth/calendar.readonly';

let tokenClient: any;
let gapiInited = false;
let gisInited = false;

let apiReady = ref(false)

watch(apiReady, (newValue, oldValue) => {
  console.log("Watch!")
  listUpcomingEvents()
  // console.log(`Input value changed from "${oldValue}" to "${newValue}"`);
});


function gapiLoaded() {
    window.gapi.load('client', initializeGapiClient);
}

async function initializeGapiClient() {
    await window.gapi.client.init({
        apiKey: API_KEY,
        discoveryDocs: [DISCOVERY_DOC],
    });
    gapiInited = true;
    maybeEnableButtons();
}

const gisLoaded = () => {
    console.log('gisLoaded')
    tokenClient = window.google.accounts.oauth2.initTokenClient({
        client_id: CLIENT_ID,
        scope: SCOPES,
        callback: '',
    });
    gisInited = true;
    maybeEnableButtons();
}

const  maybeEnableButtons = () =>  {
    if (gapiInited && gisInited) {
        console.log('GAPI initialised')
        apiReady.value = true
    }
}

const handleAuthClick = () =>  {
    if (gapiInited && gisInited) {
      tokenClient.callback = async (resp) => {
          if (resp.error !== undefined) {
          throw (resp);
          }
          await listUpcomingEvents();
      };

      if (window.gapi.client.getToken() === null) {
          tokenClient.requestAccessToken({prompt: 'consent'});
      } else {
          tokenClient.requestAccessToken({prompt: ''});
      }
    }
}

const handleSignoutClick = () => {
    const token = window.gapi.client.getToken();
    if (token !== null) {
        window.google.accounts.oauth2.revoke(token.access_token);
        window.gapi.client.setToken('');
    }
}

 const listUpcomingEvents = async () => {
  let response;
  try {
      const request = {
      'calendarId': 'primary',
      // 'timeMin': (new Date()).toISOString(),
      // 'showDeleted': false,
      // 'singleEvents': true,
      // 'maxResults': 10,
      // 'orderBy': 'startTime',
      };
      response = await window.gapi.client.calendar.events.list(request);
  } catch (err) {
    console.log(err.body)
    console.log('GAPI error:' + err.message)
    return;
  }

  const events = response.result.items;
  if (!events || events.length == 0) {
    console.log('GAPI: No events found.')
    return;
  }
  const output = events.reduce(
    (str, event) => `${str}${event.summary} (${event.start.dateTime || event.start.date})\n`,
    'Events:\n');
  console.log('GAPI: ' + output)
}
/**
 * Adds the external script for the google api.
 */
const addExternalGoogleApiJSScripts = () => {
  let gapiScript = document.createElement('script')
  gapiScript.defer = true
  gapiScript.async = true
  gapiScript.onreadystatechange = gapiScript.onload = function () {
    const interval = setInterval(function () {
      if (!gapiScript.readyState || /loaded|complete/.test(gapiScript.readyState)) {
        clearInterval(interval)
        if (window.gapi) {
          gapiLoaded()
        } else {
          console.log('Failed to load gapi')
        }
      }
    }, 100)
  }
  gapiScript.src = 'https://apis.google.com/js/api.js'
  document.head.appendChild(gapiScript)

  let gisScript = document.createElement('script')
  gisScript.defer = true
  gisScript.async = true
  gisScript.onreadystatechange = gisScript.onload = function () {
    const interval = setInterval(function () {
      if (!gisScript.readyState || /loaded|complete/.test(gisScript.readyState)) {
        clearInterval(interval)
        if (window.google && window.google.accounts) {
          gisLoaded()
        } else {
          console.log('Failed to load gis')
        }
      }
    }, 100)
  }
  gisScript.src = 'https://accounts.google.com/gsi/client'
  document.head.appendChild(gisScript)
}

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