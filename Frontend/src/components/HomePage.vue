 <template>
  <div class="dashboard-wrapper">
    <header class="dashboard-header">
      <div class="greeting"></div>
      <Navbar/>
    </header>

    <main class="main-content">
      <div class="medications-column">
        <div class="header-section">
          <p class="date">{{ formattedDate }}</p>
          <h5 class="greeting">Greetings, {{ caregiverName }}</h5>
        </div>

        <div v-if="dependents.length === 0">
        <h3>
          It looks like you haven’t added any dependents yet.
        </h3>
        </div>
        <div v-else>
          <div v-for="dependent in dependents" :key="dependent.id" class="chart-card">
            <header class="dependent-header">
              <h3>{{ dependent.username }}'s Next Medications</h3>
              <a href="#" class="view-all-link" @click.prevent="openMedicationModal(dependent)">View All</a>
            </header>

            <div v-if="dependent.medications && dependent.medications.length > 0">
  <div v-for="med in dependent.medications" 
       :key="med.medicine_id"
       :class="['med-info-card', dependent.color]">
       
    <span class="med-name">
      <span class="pill-icon">💊</span>{{ med.medicine_title }}
    </span>

    <span class="med-dosage">{{ med.dosage }}</span>

    <span class="med-time">
      <span class="icon">⏰</span>{{ med.reminder_slot }}
    </span>

    <button class="poke-button" 
            :disabled="!isPokeEnabled(med.reminder_slot)"
            @click="poke(dependent,med)">
      POKE
    </button>

  </div>
</div>
</div>
</div>
</div>

      <div class="calendar-column">
        <AppCalendar />
      </div>
    </main>

    <!-- Medication Modal -->
    <div v-if="isModalVisible" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content">
        <strong><h3 class="modal-title">{{ selectedDependent.username }}'s medications</h3></strong>

        <div v-if="isModalLoading" class="modal-loading">Loading medications...</div>

        <div v-else class="modal-meds-list">
          <div class="modal-meds-category">
            <div class="category-header"><h4><span class="category-icon">☀️</span> Daytime Meds</h4></div>
            <p v-if="modalMeds.daytime.length === 0" class="no-meds-text">No daytime medications scheduled.</p>
            <div v-else v-for="med in modalMeds.daytime" :key="med.id" class="modal-med-card day">
              <span class="med-name"><span class="pill-icon">💊</span>{{ med.medicine_name }}</span>
              <span class="med-dosage">{{ med.dosage }}</span>
              <span class="med-time"><span class="icon">⏰</span>{{ med.times.join(', ') }}</span>
            </div>
          </div>

          <div class="modal-meds-category">
            <div class="category-header"><h4><span class="category-icon">🌙</span> Nighttime Meds</h4></div>
            <p v-if="modalMeds.nighttime.length === 0" class="no-meds-text">No nighttime medications scheduled.</p>
            <div v-else v-for="med in modalMeds.nighttime" :key="med.id" class="modal-med-card night">
              <span class="med-name"><span class="pill-icon">💊</span>{{ med.medicine_name }}</span>
              <span class="med-dosage">{{ med.dosage }}</span>
              <span class="med-time"><span class="icon">⏰</span>{{ med.times.join(', ') }}</span>
            </div>
          </div>
        </div>
        <button class="modal-close-button" @click="closeModal">Close</button>
      </div>
    </div>

    <div v-if="pokeMessage" class="poke-alert" @click="pokeMessage = ''">
      {{ pokeMessage }}
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import Navbar from './Navbar.vue';
import AppCalendar from '@/components/AppCalendar.vue';
import { fetchMyDependents } from '@/services/profileService';
import apiService from '@/services/apiService'; // existing api helper
import { useRouter } from 'vue-router';
const router = useRouter();

const caregiverName = ref(sessionStorage.getItem('user_name') || 'Caregiver');
const dependents = ref([]);
const loading = ref(true);
const pokeMessage = ref('');
const cardColors = ['color-yellow', 'color-purple', 'color-blue'];

const isModalVisible = ref(false);
const selectedDependent = ref(null);
const modalMeds = ref({ daytime: [], nighttime: [] });
const isModalLoading = ref(false);

const formattedDate = computed(() =>
  new Date().toLocaleDateString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })
);

async function fetchAllDependentsData() {
  loading.value = true;
  try {
    dependents.value = await fetchMyDependents() || [];
if (!dependents.value.length) {
  router.push('/manage-dependents');
  loading.value = false;
  return; 
}


    let upcomingAll = [];
try {
  const upcomingRes = await apiService.get('/sc/upcoming-medications');
  upcomingAll = upcomingRes.data?.upcoming_medications || [];
} catch (e) {
  console.error('Upcoming meds fetch failed', e);
}

let todayAll = [];
try {
  const todayRes = await apiService.get('/sc/todays-medications');
  todayAll = todayRes.data?.medications || [];
} catch (e) {
  console.error('Today meds fetch failed', e);
}

    dependents.value = dependents.value.map((dep, idx) => {
      const depUpcoming = (upcomingAll || []).filter(m => m.user_id === dep.id);
const depTodayDaytime = (todayAll || []).filter(m =>
  m.category === 'daytime' && m.user_id === dep.id
);
const depTodayNighttime = (todayAll || []).filter(m =>
  m.category === 'nighttime' && m.user_id === dep.id
);



      return {
        ...dep,
        medications: depUpcoming,
        todayDaytime: depTodayDaytime,
        todayNighttime: depTodayNighttime,
        color: cardColors[idx % cardColors.length]
      };
    });
  } catch (err) {
    console.error('Error fetching dependents data:', err);
    dependents.value = [];
  } finally {
    loading.value = false;
  }
}

function isPokeEnabled(slot) {
  const hour = new Date().getHours();
  if (!slot) return false;
  const slotLower = slot.toLowerCase();
  if (slotLower.includes('breakfast')) return hour >= 4 && hour <= 10;
  if (slotLower.includes('lunch')) return hour >= 10 && hour <= 15;
  if (slotLower.includes('dinner')) return hour >= 16 && hour <= 23;
  return false;
}

function poke(dependent, med) {
  apiService.post('/sc/send-poke', {
    senior_id: dependent.id,
    medicine_id: med.medicine_id,
    slot: med.reminder_slot
  }).then(() => {
    pokeMessage.value = `A reminder notification has been sent to ${dependent.username}.`;
    setTimeout(() => (pokeMessage.value = ''), 4000);
  }).catch(err => {
    console.error('Error sending poke:', err);
  });
}

function openMedicationModal(dependent) {
  if (!dependent) return;
  selectedDependent.value = dependent;
  isModalVisible.value = true;
  modalMeds.value = {
    daytime: dependent.todayDaytime || [],
    nighttime: dependent.todayNighttime || []
  };
}


function closeModal() {
  isModalVisible.value = false;
  selectedDependent.value = null;
}

onMounted(() => {
  fetchAllDependentsData();
});
</script>

<style scoped>
.dashboard-wrapper {
  background-color: #f0f8f1;
  min-height: 100vh;
  font-family: "Times New Roman", serif;
  padding: 0;
}

.topbar {
  background-color: #0097a7;
  color: white;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.topbar .icon {
  font-size: 1.4rem;
  cursor: pointer;
}

.right-icons {
  display: flex;
  gap: 1.5rem;
  font-size: 1.5rem;
  align-items: center;
}

.logout-button {
  background: none;
  border: none;
  color: white;
  font-size: 1rem;
  cursor: pointer;
}

.header-section {
  padding: 2rem 3rem 0;
}

.date {
  font-size: 1rem;
  color: #555;
  margin-bottom: 0.5rem;
}

.greeting {
  font-size: 2.2rem;
  color: #333;
  margin-top: 0;
}

.main-content {
  display: flex;
  gap: 3rem;
  padding: 2rem 3rem;
}

.medications-column {
  flex: 3;
}

.calendar-column {
  flex: 2;
  min-width: 350px;
}

.subheading {
  font-size: 1.4rem;
  font-weight: bold;
  color: #333;
  margin: 2rem 0 1.5rem;
}

.chart-card {
  background-color: white;
  border-radius: 18px;
  padding: 1.5rem;
  box-shadow: 0 4px 10px rgba(0,0,0,0.05);
  margin-bottom: 2rem;
}

.med-info-card {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  padding: 1rem 1.5rem;
  border-radius: 50px;
  font-size: 1.1rem;
  font-weight: 500;
  border: 2px solid;
    margin-bottom: 0.75rem;
}

.pill-icon, .icon {
  font-size: 1.4rem;
}

.med-name {
  flex-grow: 1;
  font-weight: bold;
}

.color-yellow { background-color: #fffde7; border-color: #fbc02d; }
.color-purple { background-color: #f3e5f5; border-color: #ab47bc; }
.color-blue { background-color: #e3f2fd; border-color: #42a5f5; }

.poke-button {
  background-color: #d9534f;
  color: white;
  border: none;
  border-radius: 20px;
  padding: 0.6rem 1.5rem;
  font-weight: bold;
  font-size: 1rem;
  cursor: pointer;
}

.poke-button:disabled {
  background-color: #ed6e7b;
  cursor: not-allowed;
  opacity: 0.9;
}

.poke-alert {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: #2c3e50;
  color: white;
  padding: 1rem 2rem;
  border-radius: 8px;
  z-index: 1000;
}

.dependent-header {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 0.5rem;
}

.view-all-link {
  font-size: 1rem;
  color: #007bff;
  font-weight: bold;
  text-decoration: none;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: #ffffff;
  padding: 2rem 2.5rem;
  border-radius: 20px;
  width: 90%;
  max-width: 500px;
}

.modal-title {
  text-align: center;
  font-size: 1.8rem;
  color: #2c3e50;
  margin-bottom: 1.5rem;
}

.modal-loading {
  text-align: center;
  font-style: italic;
  font-size: 1.2rem;
  padding: 2rem;
}

.modal-meds-category {
  margin-bottom: 1.5rem;
}

.modal-med-card {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  border-radius: 50px;
  font-size: 1rem;
  border: 2px solid;
  margin-bottom: 0.75rem;
}

.modal-med-card.day { background-color: #fffde7; border-color: #fbc02d; }
.modal-med-card.night { background-color: #f3e5f5; border-color: #ab47bc; }

.modal-close-button {
  background-color: #0d6efd;
  color: white;
  padding: 0.8rem 2rem;
  border: none;
  border-radius: 10px;
  font-size: 1.1rem;
  cursor: pointer;
  margin-top: 2rem;
  display: block;
  margin-left: auto;
  margin-right: auto;
}
.modal-overlay {
  overflow-y: auto; /* allow scrolling on overlay if needed */
  -webkit-overflow-scrolling: touch; /* smooth scrolling on iOS */
}

.modal-content {
  max-height: 80vh; /* limit height to 80% of the viewport */
  overflow-y: auto; /* scroll vertically when content overflows */
  /* optionally keep your existing styles */
  background-color: #ffffff;
  padding: 2rem 2.5rem;
  border-radius: 20px;
  width: 90%;
  max-width: 800px;
}
/* Main "next medication" card inside chart-card */
.med-info-card {
  display: grid;
  grid-template-columns: 2fr 1fr 2fr auto; /* name | dosage | time | poke btn */
  align-items: center;
  text-align: center;
  padding: 1rem 1.5rem;
  border-radius: 50px;
  font-size: 1.1rem;
  font-weight: 500;
}

/* Ensure name, dosage, time each get proper column space */
.med-name, .med-dosage, .med-time {
  text-align: center;
}

/* Modal medicine rows */
.modal-med-card {
  display: grid;
  grid-template-columns: 2fr 1fr 2fr; /* name | dosage | times */
  align-items: center;
  text-align: center;
  padding: 1rem;
  border-radius: 50px;
  font-size: 1rem;
  border: 2px solid;
  margin-bottom: 0.75rem;
}

/* Ensure modal text is centered too */
.modal-med-card .med-name,
.modal-med-card .med-dosage,
.modal-med-card .med-time {
  text-align: center;
}

</style>
