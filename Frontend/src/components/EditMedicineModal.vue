<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal-content">
      <h2>Edit Medicine</h2>

      <label>Dosage</label>
      <input type="text" v-model="localMed.dosage" />
      <label>End Date</label>
      <input type="date" v-model="localMed.endDate" />

      <div class="schedule-grid">
        <label><input type="checkbox" v-model="localMed.breakfast_before" /> Breakfast Before</label>
        <label><input type="checkbox" v-model="localMed.breakfast_after" /> Breakfast After</label>
        <label><input type="checkbox" v-model="localMed.lunch_before" /> Lunch Before</label>
        <label><input type="checkbox" v-model="localMed.lunch_after" /> Lunch After</label>
        <label><input type="checkbox" v-model="localMed.dinner_before" /> Dinner Before</label>
        <label><input type="checkbox" v-model="localMed.dinner_after" /> Dinner After</label>
      </div>

      <div class="modal-actions">
        <button @click="$emit('close')">Cancel</button>
        <button class="btn-save" @click="update">Save Changes</button>
      </div>
    </div>
  </div>
</template>

<script>
import { reactive, watch } from 'vue';

export default {
  name: 'EditMedicineModal',
  props: {
    medicine: {
      type: Object,
      required: true
    }
  },
  emits: ['close', 'update-medication'],
  setup(props, { emit }) {
    const localMed = reactive({});

    watch(
      () => props.medicine,
      (newVal) => {
        if (newVal) Object.assign(localMed, newVal);
      },
      { immediate: true }
    );

    function update() {
      emit('update-medication', localMed);
    }

    return { localMed, update };
  }
};
</script>

<style scoped>
.modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.6); display: flex; justify-content: center; align-items: center; }
.modal-content { background: white; padding: 2rem; border-radius: 10px; width: 400px; }
.modal-actions { margin-top: 1rem; display: flex; justify-content: flex-end; gap: 1rem; }
.btn-save { background: #28a745; color: white; padding: 8px 16px; border: none; cursor: pointer; }
</style>
