<template>
  <div>
    <select :value="localValue" @change="onChange($event)" class="form-control">
      <option value="" disabled>Select {{ field?.['x-label'] }}</option>
      <option v-for="opt in options" :key="opt.value" :value="opt.value">
        {{ opt.label }}
      </option>
    </select>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, computed } from 'vue'

const props = defineProps({
  modelValue: [String, Number],
  field: Object,
  error: String,
  required: Boolean,
  formState: Object
})
const emit = defineEmits(['update:modelValue'])

const localValue = ref(props.modelValue)

watch(() => props.modelValue, (val) => (localValue.value = val))

const options = computed(() => {
  const data = props.field?.['x-source']?.data
  if (!data) return []

  // Dependent dropdown
  if (data.dependsOn && props.formState) {
    const depVal = props.formState[data.dependsOn]
    return (data.options?.[depVal] || []).map((o: any) => ({
      label: o.label,
      value: o.value
    }))
  }

  if (Array.isArray(data)) return data.map((o: any) => ({ label: o.label, value: o.value }))

  return []
})

const onChange = (e: Event) => {
  const value = (e.target as HTMLSelectElement).value
  localValue.value = value
  emit('update:modelValue', value)
}
</script>
