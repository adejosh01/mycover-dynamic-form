<template>
  <UFormItem :label="field?.['x-label']" :required="required" :error="error">
    <input
      type="file"
       class="form-control"
      :accept="field?.['x-source']?.data?.accept || '*/*'"
      @change="onFileChange"
    />
    <p v-if="localValue" class="text-sm text-gray-500">{{ localValue.name }}</p>
  </UFormItem>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

const props = defineProps({
  modelValue: File,
  field: Object,
  error: String,
  required: Boolean
})
const emit = defineEmits(['update:modelValue'])

const localValue = ref<File | null>(props.modelValue ?? null)

watch(
  () => props.modelValue,
  (val) => {
    localValue.value = val ?? null
  }
)

const onFileChange = (e: Event) => {
  const file = (e.target as HTMLInputElement).files?.[0] ?? null
  localValue.value = file
  emit('update:modelValue', file)
}
</script>
