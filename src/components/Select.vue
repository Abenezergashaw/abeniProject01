<script setup>
import { ref, watch, onMounted } from "vue";

const props = defineProps({
  options: {
    type: Array,
    required: true,
  },
  modelValue: {
    type: String,
    default: "",
  },
});

console.log(props.modelValue);

const emit = defineEmits(["update:modelValue"]);

const selected = ref("");

// set default value
onMounted(() => {
  if (props.modelValue) {
    selected.value = props.modelValue;
  } else if (props.options.length > 0) {
    selected.value = props.options[0];
    emit("update:modelValue", selected.value);
  }
});

// watch local changes and emit back to parent
watch(selected, (val) => {
  emit("update:modelValue", val);
});
</script>

<template>
  <select v-model="selected" class="border p-[5.4px] rounded w-full">
    <option v-for="opt in options" :key="opt" :value="opt">
      {{ opt }}
    </option>
  </select>
</template>
