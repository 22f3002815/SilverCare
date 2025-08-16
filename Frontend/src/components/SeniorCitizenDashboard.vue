<template>
  <div class="dashboard-wrapper">
    <header class="dashboard-header">
      <div class="greeting"></div>
      <Navbar/>
    </header>

    <div class="header">
      <div class="date-section">
        <p class="today-label">Today</p>
        <p class="date">{{ formattedDate }}</p>
      </div>
      <div class="greeting-row">
        <button class="assign-btn" @click="showAddMedicineModal = true">
          Assign Medicine
        </button>
        <button class="sos-button" @click="sendSOS" :disabled="sosLoading">
          {{ sosLoading ? '...' : 'SOS' }}
        </button>
      </div>
    </div>

    <div v-if="error" style="margin-top:10%;margin-left:30%;margin-right:30%">
      <h3>{{ error }}</h3>
    </div>

    <div v-else class="main-content">
      <div class="left-panel">
        <p class="motivational-quote">"The best way to predict the future is to create it."</p>
        <h2 class="greeting">Greetings, {{ userName }}</h2>
        <h3 class="subheading">Upcoming Medications</h3>
        <div v-if="loading" class="upcoming-med-card placeholder">Loading...</div>
        <div v-else-if="upcomingMeds.length > 0">
          <div v-for="med in upcomingMeds" :key="med.medicineId" class="upcoming-med-card">
            <span class="med-name"><span class="icon pill-icon">💊</span>{{ med.medicineName }}</span>
            <span class="med-dosage">{{ med.dosage }}</span>
            <span class="med-time"><span class="icon clock-icon">⏰</span>{{ med.time }}</span>
            <button class="mark-taken-btn" @click="markMedicineTaken(med.medicineId, getSlot(med.slot))">Mark as Taken</button>
          </div>
        </div>
        <div v-else class="upcoming-med-card empty">
          <h5 style="margin-left:0%">No medications in the current window.</h5>
        </div>

        <h3 class="subheading today-meds-title">Today's medications</h3>
        <div v-if="loading" class="todays-meds-container placeholder">Loading schedule...</div>

        <div v-else class="todays-meds-container">
          <div class="meds-category">
            <div class="category-header"><span class="category-icon">☀️</span> Daytime Meds</div>
            <p v-if="daytimeMeds.length === 0" class="no-meds-text">No daytime medications.</p>
            <div v-else>
              <div v-for="med in daytimeMeds" :key="med.id" class="med-card day">
                <span class="med-name"><span class="icon pill-icon">💊</span>{{ med.medicine_name || med.title }}</span>
                <span class="med-dosage">{{ med.dosage || '-' }}</span>
                <span class="med-time"><span class="icon clock-icon">⏰</span> {{ med.times.join(", ") }}</span>
              </div>
            </div>
          </div>

          <div class="meds-category">
            <div class="category-header"><span class="category-icon">🌙</span> Nighttime Meds</div>
            <p v-if="nighttimeMeds.length === 0" class="no-meds-text">No nighttime medications.</p>
            <div v-else>
              <div v-for="med in nighttimeMeds" :key="med.id" class="med-card night">
                <span class="med-name"><span class="icon pill-icon">💊</span>{{ med.medicine_name || med.title || '-' }}</span>
                <span class="med-dosage">{{ med.dosage || '-' }}</span>
                <span class="med-time"><span class="icon clock-icon">⏰</span> {{ med.times.join(", ") }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="right-panel">
        <MusicPlayer />
        <AppCalendar />
      </div>

      <AddMedicineModal
        v-if="showAddMedicineModal"
        @close="showAddMedicineModal = false"
        @add-medication="handleAddMedication"
      />

      <div v-if="sosMessage" class="sos-alert" @click="sosMessage = ''">{{ sosMessage }}</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import Navbar from './Navbar.vue';
import AppCalendar from './AppCalendar.vue';
import apiService from '@/services/apiService';
import MusicPlayer from './MusicPlayer.vue';
import AddMedicineModal from './AddMedicineModal.vue';

const userName = ref(sessionStorage.getItem('user_name') || 'User');
const upcomingMeds = ref([]);
const daytimeMeds = ref([]);
const nighttimeMeds = ref([]);
const loading = ref(true);
const error = ref(null);
const sosLoading = ref(false);
const sosMessage = ref('');
const showAddMedicineModal = ref(false);

const formattedDate = computed(() =>
  new Date().toLocaleDateString('en-US', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  })
);
const slotMap = {
  "Breakfast before": "breakfast_before",
  "Breakfast after": "breakfast_after",
  "Lunch before": "lunch_before",
  "Lunch after": "lunch_after",
  "Dinner before": "dinner_before",
  "Dinner after": "dinner_after"
};

function getSlot(reminderSlot) {
  if (!reminderSlot) return null;
  return slotMap[reminderSlot] || slotMap[reminderSlot.charAt(0).toUpperCase() + reminderSlot.slice(1)] || null;
}


async function getUpcomingMedication() {
  try {
    const res = await apiService.get('/sc/upcoming-medications');
    const data = res.data;

    if (res.status === 200) {
      return {
        success: true,
        meds: data.upcoming_medications || [],
        count: data.count || 0,
      };
    } else {
      return {
        success: false,
        message: data.error || data.message || "Failed to get medications",
      };
    }
  } catch (err) {
    if (err.response && err.response.status === 404) {
      return {
        success: true,
        meds: [],
        count: 0,
      };
    }
    console.error("🔴 Error fetching upcoming medications:", err);
    return {
      success: false,
      message: err.message,
    };
  }
}

async function fetchAllDataForSenior() {
  loading.value = true;
  error.value = null;
  try {
    const response = await getUpcomingMedication();
    if (!response.success) throw new Error(response.message);

    upcomingMeds.value = response.meds.map(med => ({
      medicineId: med.medicine_id,
      medicineName: med.medicine_title,
      dosage: med.dosage,
      time: med.reminder_slot,
      slot: med.reminder_slot
    }));

    const todayRes = await apiService.get('/sc/todays-medications');
    if (todayRes.status === 200 && todayRes.data.medications) {
      daytimeMeds.value = todayRes.data.medications.filter(m => m.category === 'daytime');
      nighttimeMeds.value = todayRes.data.medications.filter(m => m.category === 'nighttime');
    } else {
      daytimeMeds.value = [];
      nighttimeMeds.value = [];
    }
  } catch (err) {
    error.value = "No medicines yet — click Assign Medicine to get started.";
  } finally {
    loading.value = false;
  }
}

async function assignMedicineSchedule(payload) {
  try {
    const requestBody = { ...payload };
    const response = await apiService.post('/sc/assign-medicine', requestBody);
    return { success: true, data: response.data };
  } catch (error) {
    console.error('Error assigning medicine schedule:', error.response?.data || error.message);
    return {
      success: false,
      error: error.response?.data?.message || error.message || 'Failed to assign medicine schedule',
      validationErrors: error.response?.data?.errors || null,
    };
  }
}

async function handleAddMedication(payload) {
  showAddMedicineModal.value = false;
  const finalPayload = { ...payload };
  console.log('Payload sent:', finalPayload);
  const result = await assignMedicineSchedule(finalPayload);

  if (result.success) {
    alert('Medicine assigned successfully.');
    fetchAllDataForSenior();
  } else {
    console.error('Validation errors:', result.validationErrors);
    alert('Failed to assign medicine: ' + result.error);
  }
}

async function markMedicineTaken(medicineId, slot) {
  console.log("Medicine ID:", medicineId);
  console.log("Slot:", slot);

  const validSlots = [
    "breakfast_before", "breakfast_after",
    "lunch_before", "lunch_after",
    "dinner_before", "dinner_after"
  ];

  if (!medicineId || !slot || !validSlots.includes(slot)) {
    alert(`Invalid medicine or slot value: ${slot}`);
    return;
  }

  try {
    const res = await apiService.put('/sc/mark-medicine-taken', {
      medicine_id: medicineId,
      slot: slot,
    });

    if (res.status === 200) {
      alert('Medicine marked as taken.');
      fetchAllDataForSenior();
    } else {
      alert('Failed to mark medicine as taken.');
    }
  } catch (error) {
    console.error('Error marking medicine as taken:', error);
    alert(error.response?.data?.error || 'Error marking medicine as taken.');
  }
}

async function sendSOS() {
  sosLoading.value = true
  sosMessage.value = ''

  try {
    const res = await apiService.post('/sc/send-sos')
    if (res.status === 200 && res.data.status === 'SOS sent successfully') {
      console.log('✅ SOS Alert Sent:', res.data.alerts_sent)
      sosMessage.value = '🚨 SOS Alert sent to your caregivers.'
    } else {
      sosMessage.value = res.data.message || '⚠️ Could not send SOS alert.'
    }
  } catch (err) {
    console.error('❌ Error sending SOS:', err)
    sosMessage.value = err.response?.data?.error || '❌ Failed to send SOS alert.'
  } finally {
    sosLoading.value = false
    setTimeout(() => {
      sosMessage.value = ''
    }, 4000)
  }
}

onMounted(() => {
  fetchAllDataForSenior();
});
</script>

<style scoped>
/* Remove styles related to the collapsible header */
.collapsible-card {
  /* No longer needed */
}

.collapsible-header {
  /* No longer needed */
}

.collapsible-title {
  /* No longer needed */
}

.toggle-icon {
  /* No longer needed */
}

.toggle-icon.rotated {
  /* No longer needed */
}

.slide-fade-enter-active {
  /* No longer needed */
}

.slide-fade-leave-active {
  /* No longer needed */
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  /* No longer needed */
}

/* Existing styles below this line remain the same */
.dashboard-wrapper {
  background-color: #eef7f2;
  min-height: 100vh;
  font-family: "Georgia", serif;
  padding: 2rem;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.date-section {
  text-align: center;
}

.today-label {
  font-weight: bold;
  font-size: 1rem;
  color: #555;
  margin-bottom: 0.2rem;
}

.date {
  font-size: 1rem;
  color: #777;
}

.greeting {
  font-family: 'Georgia', serif;
  font-size: 2.2rem;
  margin: 0.5rem 0 1rem;
  color: #2c3e50;
  text-align: center;
}

.subheading {
  font-weight: bold;
  color: #333;
  font-size: 1.3rem;
  margin: 2rem 0 1rem;
}

.today-meds-title {
  margin-top: 2rem;
}

.upcoming-med-card {
  padding: 1rem 0.75rem;
  border-radius: 50px;
  display: grid;
  grid-template-columns: 2.5fr 1fr 2fr auto;
  align-items: center;
  text-align: center;
  gap: 1rem;
  font-size: 1.05rem;
  font-weight: 500;
  margin-bottom: 1rem;
  border: 2px solid #6498f8;
  background-color: #96ace9;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  color: #3c3c3c;
}

.upcoming-med-card .med-name {
  flex-grow: 1;
}

.upcoming-med-card .med-time {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  justify-content: center;
}

.icon {
  font-size: 1.3rem;
}

.main-content {
  display: flex;
  flex-wrap: wrap;
  gap: 2rem;
  align-items: flex-start;
}

.left-panel {
  flex: 2;
  min-width: 0;
}

.right-panel {
  flex: 1;
  min-width: 300px;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.todays-meds-container {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.meds-category {
  background-color: #ffffff;
  padding: 1.5rem 2rem;
  border-radius: 20px;
  border: 1px solid #dbe8d9;
  box-shadow: 0 4px 10px rgba(0,0,0,0.05);
}

.category-header {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  font-size: 1.3rem;
  font-weight: bold;
  color: #333;
  margin-bottom: 1rem;
}

.category-icon {
  font-size: 1.5rem;
}

.no-meds-text {
  font-style: italic;
  color: #666;
}

.med-card {
  display: grid;
  grid-template-columns: 2fr 1fr 2fr;
  align-items: center;
  text-align: center;
  padding: 1rem 1.2rem;
  border-radius: 50px;
  font-size: 1.05rem;
  font-weight: 500;
  margin-bottom: 1rem;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.08);
}

.med-card .med-name {
  flex-grow: 0.2;
}

.med-card .med-time {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  justify-content: center;
}

.med-card.day {
  background-color: #fff9c4;
  color: #5d4037;
  border: 1px solid #f4e04d;
}

.med-card.night {
  background-color: #f3e8fd;
  color: #4a148c;
  border: 1px solid #c69be4;
}

.sos-button {
  background-color: #d9534f;
  color: white;
  border: none;
  padding: 0.6rem 1.4rem;
  border-radius: 50px;
  font-weight: bold;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.3s;
}

.sos-button:disabled {
  background-color: #e7a1a1;
  cursor: not-allowed;
}

.sos-alert {
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

.error-state {
  text-align: center;
  padding: 3rem;
  background-color: #f8d7da;
  color: #721c24;
  border-radius: 15px;
  border: 1px solid #f5c6cb;
}
.assign-btn {
  margin: 1rem 0;
  background: #6498f8;
  color: white;
  font-weight: bold;
  border: none;
  border-radius: 50px;
  padding: 0.7rem 1.6rem;
  font-size: 1rem;
  cursor: pointer;
}
.assign-btn:hover {
  background: #4169e1;
}
.mark-taken-btn {
  margin-left: 0.5rem;
  background-color: #28a745;
  border: none;
  color: white;
  padding: 0.4rem 0.5rem;
  border-radius: 20px;
  cursor: pointer;
  font-weight: bold;
}

.mark-taken-btn:hover {
  background-color: #218838;
}
</style>
