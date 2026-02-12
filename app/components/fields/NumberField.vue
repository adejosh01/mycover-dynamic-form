<template>
  <div>
    <input
      type="number"
      :value="localValue"
      :placeholder="field?.['x-description'] || ''"
      class="form-control"
      @input="onInput"
    >
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

const props = defineProps({
  modelValue: Number,
  field: Object,
  error: String,
  required: Boolean
})
const emit = defineEmits(['update:modelValue'])

const localValue = ref(props.modelValue)

watch(() => props.modelValue, val => (localValue.value = val))

const onInput = (e: Event) => {
  const value = (e.target as HTMLInputElement).value
  // To align with the expected type (number | undefined), use undefined instead of null
  localValue.value = value ? Number(value) : undefined
  emit('update:modelValue', localValue.value)
}
</script>
