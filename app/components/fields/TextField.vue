<template>
  <div>
    <input
      type="text"
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
  modelValue: String,
  field: Object,
  error: String,
  required: Boolean
})
const emit = defineEmits(['update:modelValue'])

const localValue = ref(props.modelValue)

watch(() => props.modelValue, val => (localValue.value = val))

const onInput = (e: Event) => {
  const value = (e.target as HTMLInputElement).value
  localValue.value = value
  emit('update:modelValue', value)
}
</script>
