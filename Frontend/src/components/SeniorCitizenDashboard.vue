<template>
  <div class="dashboard-wrapper" :class="fontSizeClass">
    <header class="dashboard-header">
      <div class="greeting"></div>
      <Navbar />
    </header>

    <div class="header">
      <div class="header-left">
        <br>
        <h2 class="greeting">Hi, {{ userName }}</h2>
        <p class="date">{{ formattedDate }}</p>
      </div>

      <!-- Analog Clock moved to header -->
      <div class="analog-clock">
          <div class="hand hour-hand" :style="hourHandStyle"></div>
          <div class="hand minute-hand" :style="minuteHandStyle"></div>
          <div class="hand second-hand" :style="secondHandStyle"></div>
          <div class="pivot"></div>
      </div>

      <div class="header-right">
        <!-- Accessibility: Font Size Cycle Button -->
        <button class="accessibility-btn" @click="cycleFontSize" title="Cycle Font Size">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 12h16M4 18h16M4 6h16"/></svg>
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 12h16M4 18h16M4 6h16"/></svg>
        </button>
        <button class="assign-btn" @click="showAddMedicineModal = true">
          Assign Medicine
        </button>
        <button class="sos-button" @click="sendSOS" :disabled="sosLoading">
          {{ sosLoading ? '...' : 'SOS' }}
        </button>
      </div>
    </div>

    <div v-if="error" class="error-state">
      <h3>{{ error }}</h3>
    </div>

    <div v-else class="main-content">
      <div class="left-panel">
        <!-- Upcoming Medications Revamped & Relocated -->
        <div class="upcoming-meds-section">
            <h3 class="subheading">Next Up</h3>
            <div v-if="loading" class="med-card-placeholder">Loading...</div>
            <div v-else-if="upcomingMeds.length > 0">
              <div v-for="med in upcomingMeds" :key="med.medicineId" class="upcoming-med-card">
                <div class="icon-wrapper alert-icon-bg">
                  <!-- Bell Icon -->
                  <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/></svg>
                </div>
                <div class="med-details">
                  <span class="med-name">{{ med.medicineName }}</span>
                  <span class="med-dosage">{{ med.dosage }}</span>
                </div>
                <div class="med-time-section">
                  <span class="med-time">{{ med.time }}</span>
                </div>
                <button class="mark-taken-btn" @click="markMedicineTaken(med.medicineId, getSlot(med.slot))">
                  Mark as Taken
                </button>
              </div>
            </div>
            <div v-else class="med-card empty">
              <p>No medications in the current window.</p>
            </div>
        </div>

        <!-- Today's Medications Revamped -->
        <div class="todays-meds-section">
            <h3 class="subheading">Today's Schedule</h3>
            <div v-if="loading" class="med-card-placeholder">Loading schedule...</div>
            <div v-else class="todays-meds-container">
              <!-- Daytime Meds -->
              <div class="meds-category">
                <div class="category-header">
                  <div class="icon-wrapper day-icon-bg">
                    <!-- Sun Icon -->
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="5"/><line x1="12" y1="1" x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/><line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/></svg>
                  </div>
                  <span>Daytime Meds</span>
                </div>
                <p v-if="daytimeMeds.length === 0" class="no-meds-text">No daytime medications scheduled.</p>
                <div v-else class="med-list">
                  <div v-for="med in daytimeMeds" :key="med.id" class="med-card">
                     <div class="med-details">
                        <span class="med-name">{{ med.medicine_name || med.title }}</span>
                        <span class="med-dosage">{{ med.dosage }}</span>
                     </div>
                     <div class="med-time-section">
                        <span class="med-time">{{ med.times.join(", ") }}</span>
                     </div>
                  </div>
                </div>
              </div>

              <!-- Nighttime Meds -->
              <div class="meds-category">
                <div class="category-header">
                  <div class="icon-wrapper night-icon-bg">
                    <!-- Moon Icon -->
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
                  </div>
                  <span>Nighttime Meds</span>
                </div>
                <p v-if="nighttimeMeds.length === 0" class="no-meds-text">No nighttime medications scheduled.</p>
                <div v-else class="med-list">
                  <div v-for="med in nighttimeMeds" :key="med.id" class="med-card">
                     <div class="med-details">
                        <span class="med-name">{{ med.medicine_name || med.title || '-' }}</span>
                        <span class="med-dosage">{{ med.dosage || '-' }}</span>
                     </div>
                     <div class="med-time-section">
                        <span class="med-time">{{ med.times.join(", ") }}</span>
                     </div>
                  </div>
                </div>
              </div>
            </div>
        </div>
      </div>

      <div class="right-panel">
        <p class="motivational-quote">"The best way to predict the future is to create it."</p>
        
        <!-- Collapsible Music Player Card -->
        <div class="collapsible-card">
            <div class="collapsible-header" @click="showMusicPlayer = !showMusicPlayer">
                <h4 class="collapsible-title">Soothing Music</h4>
                <svg :class="['toggle-icon', { 'rotated': showMusicPlayer }]" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>
            </div>
            <transition name="slide-fade">
                <div v-if="showMusicPlayer" class="collapsible-content">
                    <MusicPlayer />
                </div>
            </transition>
        </div>
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
import { ref, onMounted, onUnmounted, computed } from 'vue';
import Navbar from './Navbar.vue';
import AppCalendar from './AppCalendar.vue';
import apiService from '@/services/apiService'
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
const showMusicPlayer = ref(true); // Default: shown

// --- Accessibility & Clock Features State ---
const fontSizeLevel = ref(0); // 0: normal, 1: large, 2: larger
const hourHandStyle = ref({});
const minuteHandStyle = ref({});
const secondHandStyle = ref({});
let clockInterval = null;

// Computed property to apply the correct font size class
const fontSizeClass = computed(() => `font-size-level-${fontSizeLevel.value}`);

// Function to cycle through font sizes
function cycleFontSize() {
  fontSizeLevel.value = (fontSizeLevel.value + 1) % 3; // Cycles through 0, 1, 2
}

const formattedDate = computed(() =>
  new Date().toLocaleDateString('en-US', {
    weekday: 'long',
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

// --- Updated Clock Function for Analog Clock ---
function updateTime() {
    const now = new Date();
    const seconds = now.getSeconds();
    const minutes = now.getMinutes();
    const hours = now.getHours();

    const secondsDegrees = ((seconds / 60) * 360) + 90;
    const minutesDegrees = ((minutes / 60) * 360) + ((seconds/60)*6) + 90;
    const hoursDegrees = ((hours / 12) * 360) + ((minutes/60)*30) + 90;

    secondHandStyle.value = { transform: `rotate(${secondsDegrees}deg)` };
    minuteHandStyle.value = { transform: `rotate(${minutesDegrees}deg)` };
    hourHandStyle.value = { transform: `rotate(${hoursDegrees}deg)` };
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
  // Start the clock
  updateTime();
  clockInterval = setInterval(updateTime, 1000);
});

onUnmounted(() => {
    // Clean up the clock interval
    clearInterval(clockInterval);
});
</script>

<style scoped>
/* --- General & Layout --- */
.dashboard-wrapper {
  /* Font size scaling system using CSS variables */
  --font-scale: 1;

  background: url('https://images.unsplash.com/photo-1530305408560-82d13781b33a?q=80&w=2072&auto=format&fit=crop');
  background-size: cover;
  background-position: center;
  background-attachment: fixed;

  min-height: 100vh;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  padding: 2rem;
  transition: font-size 0.3s ease;
}

/* Accessibility Classes for Font Scaling */
.dashboard-wrapper.font-size-level-0 { --font-scale: 1; }
.dashboard-wrapper.font-size-level-1 { --font-scale: 1.1; }
.dashboard-wrapper.font-size-level-2 { --font-scale: 1.2; }


.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
  flex-wrap: wrap;
}

.header-left {
    flex: 1;
    min-width: 250px;
}
.header-left .greeting {
  font-family: 'Georgia', serif;
  font-size: calc(2.2rem * var(--font-scale));
  margin: 0;
  color: #e7ebee;
}
.header-left .date {
  font-size: calc(1rem * var(--font-scale));
  color: #6c757d;
  margin-top: 0.25rem;
}

.header-right {
    display: flex;
    align-items: center;
    flex: 1;
    justify-content: flex-end;
    min-width: 300px;
}

.subheading {
  font-weight: 600;
  color: #343a40;
  font-size: calc(1.2rem * var(--font-scale));
  margin: 0 0 1rem 0;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid #dee2e6;
}

.main-content {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 2rem;
  align-items: flex-start;
}

.left-panel {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.right-panel {
  position: sticky;
  top: 2rem;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.motivational-quote {
    background-color: #e9ecef;
    border-left: 4px solid #007bff;
    padding: 1rem;
    border-radius: 8px;
    font-style: italic;
    color: #495057;
    font-size: calc(0.95rem * var(--font-scale));
}

/* --- Analog Clock in Header --- */
.analog-clock {
    width: 80px;
    height: 80px;
    border-radius: 50%;
    border: 3px solid #343a40;
    position: relative;
    background-color: #f8f9fa;
    box-shadow: 0 0 15px rgba(0,0,0,0.1);
}
.analog-clock .hand {
    width: 50%;
    background: #343a40;
    position: absolute;
    top: 50%;
    transform-origin: 100%;
    transform: rotate(90deg);
    transition: transform 0.05s cubic-bezier(0.4, 2.3, 0.6, 1);
    border-top-right-radius: 4px;
    border-bottom-right-radius: 4px;
}
.analog-clock .hour-hand {
    width: 35%;
    left: 15%;
    height: 4px;
    top: calc(50% - 2px);
}
.analog-clock .minute-hand {
    height: 3px;
    top: calc(50% - 1.5px);
}
.analog-clock .second-hand {
    width: 45%;
    left: 5%;
    height: 1px;
    top: calc(50% - 0.5px);
    background: #dc3545;
}
.analog-clock .pivot {
    width: 8px;
    height: 8px;
    background: #343a40;
    border-radius: 50%;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 10;
}


/* --- Buttons --- */
.accessibility-btn {
    background-color: #6c757d;
    color: white;
    border: none;
    width: 44px;
    height: 44px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    margin-right: 1rem;
    transition: all 0.2s ease-in-out;
}
.accessibility-btn:hover {
    background-color: #5a6268;
    transform: translateY(-2px);
}
.accessibility-btn svg {
    width: 16px;
    height: 16px;
}
.accessibility-btn svg:first-child {
    width: 12px;
    height: 12px;
    margin-right: 2px;
}

.assign-btn, .sos-button {
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 50px;
  font-weight: 600;
  font-size: calc(0.9rem * var(--font-scale));
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  letter-spacing: 0.5px;
}

.assign-btn {
  background-color: #007bff;
  color: white;
  margin-right: 1rem;
}
.assign-btn:hover {
  background-color: #0056b3;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 123, 255, 0.2);
}

.sos-button {
  background-color: #dc3545;
  color: white;
}
.sos-button:hover {
  background-color: #c82333;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(220, 53, 69, 0.2);
}
.sos-button:disabled {
  background-color: #e7a1a1;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

/* --- NEW MEDICATION CARD STYLES --- */
.upcoming-meds-section, .todays-meds-section {
    background-color: #ffffff;
    padding: 1.5rem;
    border-radius: 16px;
    border: 1px solid #e9ecef;
    box-shadow: 0 4px 6px rgba(0,0,0,0.03);
}

.icon-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 38px;
  height: 38px;
  border-radius: 50%;
  margin-right: 0.8rem;
}
.icon-wrapper svg {
  width: 18px;
  height: 18px;
}

.upcoming-med-card {
  display: flex;
  align-items: center;
  padding: 0.75rem 1rem;
  border-radius: 12px;
  margin-bottom: 0.75rem;
  background-color: #e7f3ff;
  border: 1px solid #b3d7ff;
}
.upcoming-med-card .alert-icon-bg {
  background-color: #cce5ff;
  color: #004085;
}

.todays-meds-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
}

.meds-category {
  background-color: transparent;
  padding: 0;
  border: none;
  box-shadow: none;
}

.category-header {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  font-size: calc(1.1rem * var(--font-scale));
  font-weight: 600;
  color: #343a40;
  margin-bottom: 1rem;
}

.day-icon-bg { background-color: #fff4d5; color: #ffc107; }
.night-icon-bg { background-color: #e9e7fd; color: #6f42c1; }

.med-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.med-card {
  display: flex;
  align-items: center;
  padding: 0.8rem 1rem;
  border-radius: 10px;
  background-color: #f8f9fa;
  border: 1px solid #e9ecef;
}

.med-card.empty {
  justify-content: center;
  padding: 2rem;
  text-align: center;
  color: #6c757d;
  background-color: #f8f9fa;
  border: 1px dashed #ced4da;
}

.med-details {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

.med-name {
  font-weight: 600;
  color: #212529;
  font-size: calc(0.95rem * var(--font-scale));
}

.med-dosage {
  font-size: calc(0.8rem * var(--font-scale));
  color: #6c757d;
}

.med-time-section {
  text-align: right;
  min-width: 70px;
}

.med-time {
  font-weight: 500;
  color: #495057;
  font-size: calc(0.85rem * var(--font-scale));
  background-color: #e9ecef;
  padding: 0.2rem 0.6rem;
  border-radius: 6px;
}

.no-meds-text {
  font-style: italic;
  color: #6c757d;
  text-align: center;
  padding: 1rem 0;
  font-size: calc(0.9rem * var(--font-scale));
}

.mark-taken-btn {
  margin-left: 1rem;
  background-color: #28a745;
  border: none;
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 600;
  font-size: calc(0.85rem * var(--font-scale));
  white-space: nowrap;
  transition: background-color 0.2s ease;
}

.mark-taken-btn:hover {
  background-color: #218838;
}

/* --- Other Components --- */
.sos-alert {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: #212529;
  color: white;
  padding: 1rem 2rem;
  border-radius: 8px;
  z-index: 1000;
  box-shadow: 0 5px 15px rgba(0,0,0,0.2);
}

.error-state {
  text-align: center;
  padding: 3rem;
  background-color: #f8d7da;
  color: #721c24;
  border-radius: 15px;
  border: 1px solid #f5c6cb;
  margin: 2rem;
}

.collapsible-card {
  background-color: #ffffff;
  border-radius: 16px;
  border: 1px solid #e9ecef;
  box-shadow: 0 4px 6px rgba(0,0,0,0.03);
  overflow: hidden;
}

.collapsible-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 1.5rem;
  cursor: pointer;
  user-select: none;
}

.collapsible-title {
  margin: 0;
  font-size: calc(1.1rem * var(--font-scale));
  font-weight: 600;
  color: #343a40;
}

.toggle-icon {
  width: 20px;
  height: 20px;
  color: #6c757d;
  transition: transform 0.3s ease;
}

.toggle-icon.rotated {
  transform: rotate(180deg);
}

.slide-fade-enter-active {
  transition: all 0.3s ease-out;
}
.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateY(-10px);
  opacity: 0;
}

/* Responsive adjustments */
@media (max-width: 992px) {
    .main-content {
        grid-template-columns: 1fr;
    }
    .right-panel {
        position: static;
    }
    .header {
        gap: 1.5rem;
    }
    .header-left, .header-right {
        flex-basis: 45%;
    }
    .analog-clock {
        order: -1;
        flex-basis: 100%;
        margin: 0 auto;
    }
}

@media (max-width: 768px) {
    .header {
        flex-direction: column;
        align-items: center;
        gap: 1.5rem;
    }
    .header-left, .header-right {
        text-align: center;
        width: 100%;
        justify-content: center;
    }
    .todays-meds-container {
        grid-template-columns: 1fr;
    }
}
</style>
