<template>
  <div>
    <h1>User List with Map</h1>
    <div v-for="user in users" :key="user.email" class="user-card">
      <div class="user-details">
        <h2>{{ user.name.first }} {{ user.name.last }}</h2>
        <p>Email: {{ user.email }}</p>
        <p>Gender: {{ user.gender }}</p>
       <p v-if="user.location">Country: {{ user.location.country }}</p>
        <p>Date of Birth: {{ formatDate(user.dob.date) }}</p>
      </div>
      <button @click="showOnMap(user)" class="show-map-button">Show on Map</button>
    </div>
    <button @click="fetchUsers" class="next-button">Next</button>
    <div id="map-container">
      <div id="map"></div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch } from 'vue';

export default {
  setup() {
    const users = ref([]);
    const currentPage = ref(1);
    const selectedUser = ref(null);
    const googleMapsLoaded = ref(false);
    const scriptLoaded = ref(false);

    const fetchUsers = async () => {
      const response = await fetch(
        `https://randomuser.me/api/?results=5&page=${currentPage.value}`
      );
      const data = await response.json();
      users.value = data.results;
      currentPage.value++;
    };

const showOnMap = (user) => {
  console.log('showOnMap - user:', user);
  
  if (user && user.location) {
    console.log('showOnMap - user.location:', user.location);
    
    selectedUser.value = user;
    if (googleMapsLoaded.value) {
      initializeMap(user);
    }
  }
};



    const formatDate = (value) => {
      return new Date(value).toLocaleDateString();
    };

    const initializeMap = (user) => {
      // eslint-disable-next-line no-undef
      if (typeof google === 'undefined' || typeof google.maps === 'undefined') {
        return;
      }

      // eslint-disable-next-line no-undef
      const map = new google.maps.Map(document.getElementById('map'), {
        center: {
          lat: parseFloat(user.location.coordinates.latitude),
          lng: parseFloat(user.location.coordinates.longitude),
        },
        zoom: 10,
      });

      // eslint-disable-next-line no-undef
      new google.maps.Marker({
        position: {
          lat: parseFloat(user.location.coordinates.latitude),
          lng: parseFloat(user.location.coordinates.longitude),
        },
        map,
      });
    };

  const loadGoogleMapsScript = () => {
  const script = document.createElement('script');
  script.src = `https://maps.googleapis.com/maps/api/js?key=${process.env.VUE_APP_GOOGLE_MAPS_API_KEY}&libraries=places`;

  script.onload = () => {
    scriptLoaded.value = true;
    initializeMap(selectedUser.value); // Initialize the map if a user is selected.
  };

  document.head.appendChild(script);
};


    onMounted(() => {
      loadGoogleMapsScript();
    });

 watch(scriptLoaded, (loaded) => {
  if (loaded) {
    googleMapsLoaded.value = true;
    fetchUsers();
  }
});

    return {
      users,
      fetchUsers,
      showOnMap,
      formatDate,
    };
  },
};
</script>
<style>
.user-card {
  display: inline-block;
  width: 20rem;
  height: 20rem;
  background-color: purple;
  color: white;
  margin: 10px;
  border-radius: 10px;
  text-align: center;
  overflow: hidden;
}

.user-details {
  padding: 10px;
}

.show-map-button {
  background-color: purple;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 5px;
  cursor: pointer;
}
.show-map-button {
  background-color: white;
  color: purple;
  border: none;
  padding: 8px 16px;
  border-radius: 5px;
  cursor: pointer;
  transform: translateX(-50%);
  transition: background-color 0.3s, color 0.3s;
}

.show-map-button:hover {
  background-color: purple;
  color: white;
}

.next-button {
  background-color: purple;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  margin-top: 20px;
  transition: background-color 0.3s, color 0.3s;
}

.next-button:hover {
  background-color: white;
  color: purple;
}
</style>