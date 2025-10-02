<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  modelValue: {
    type: String,
    default: new Date().toISOString().split("T")[0], // fallback = today
  },
});

const emit = defineEmits(["update:modelValue"]);

const selectedDate = ref(props.modelValue);

// keep local value in sync if parent changes
watch(
  () => props.modelValue,
  (newVal) => {
    selectedDate.value = newVal;
  }
);

// emit updates when input changes
watch(selectedDate, (newVal) => {
  emit("update:modelValue", newVal);
});
</script>

<template>
  <div>
    <input
      type="date"
      v-model="selectedDate"
      class="border p-1 rounded w-full"
    />
  </div>
</template>
