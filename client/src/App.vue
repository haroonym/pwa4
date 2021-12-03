<template>
  <div
    id="app"
    class="container-fluid d-flex flex-column justify-content-center align-items-center mt-5"
  >
    <h5>Willkommen bei der Service Worker Untersuchung!!</h5>
    <img
      src="../public/images/employees.jpg"
      height="300px"
      class="mx-auto d-block mt-5"
      alt="picture of employee"
    />
    <h5 class="mb-5 mt-5">Hier könnte Ihre Werbung stehen!</h5>

    <div
      class="alert alert-danger text-center"
      v-if="this.offline == true"
      style="width: 100%"
      role="alert"
    >
      You are offline...
    </div>
    <ButtonGet @get="fetchData"></ButtonGet>
    <CardView :employees="employees" @del="delEmployee"></CardView>
  </div>
</template>

<script>
import ButtonGet from '@/components/ButtonGet.vue';
import CardView from '@/components/CardView.vue';
import axios from 'axios';

export default {
  name: 'app',
  components: {
    ButtonGet,
    CardView,
  },
  data() {
    return {
      employees: [],
      serverAddress: process.env.VUE_APP_SERVER,
      once: false,
      offline: false,
    };
  },
  methods: {
    async getEmployees() {
      try {
        const { data } = await axios({
          url: this.serverAddress + '/employees',
          method: 'GET',
        });
        this.employees = data;
      } catch (error) {
        console.error(error);
      }
    },
    async delEmployee(e) {
      try {
        console.log(`${this.serverAddress}/employees/${e.id}`);
        await axios({
          url: `${this.serverAddress}/employees/${e.id}`,
          method: 'DELETE',
        });
        this.getEmployees();
      } catch (error) {
        console.error(error);
      }
    },
    fetchData() {
      console.log('fetchData called');
      this.getEmployees();
    },
    updateAvailable() {
      alert('Update vorhanden, bitte App neu starten!');
    },
    async subscribe() {
      if (!('serviceWorker' in navigator)) {
        console.log('no service worker!');
        return;
      }
      const registration = await navigator.serviceWorker.ready;
      const subscription = await registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: this.urlBase64ToUint8Array(publicVapidKey),
      });
      await axios.post('/subscribe', subscription);
    },
  },
  created() {
    document.addEventListener('swUpdated', this.updateAvailable, { once: true });
    window.addEventListener('online', () => (this.offline = false));
    window.addEventListener('offline', () => (this.offline = true));
  },
};
</script>

<style></style>
