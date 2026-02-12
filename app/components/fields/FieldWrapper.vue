<template>
  <div class="flex flex-col mb-4">
    <!-- Label -->
    <label class="block text-sm font-medium text-gray-900 dark:text-gray-100 mb-2">
      {{ field?.['x-label'] }}
      <span
        v-if="required"
        class="text-red-500"
      >
        *
      </span>
    </label>

    <!-- Field Component -->
    <component
      :is="componentType"
      v-model="localValue"
      :field="field"
      :form-state="formState"
      :required="required"
      :error="error"
    />

    <!-- Error -->
    <p
      v-if="error"
      class="text-red-500 text-sm mt-1"
    >
      {{ error }}
    </p>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

const props = defineProps({
  modelValue: [String, Number, File, Boolean],
  field: Object,
  error: String,
  required: Boolean,
  formState: Object,
  componentType: [String, Object]
})

const emit = defineEmits(['update:modelValue'])

const localValue = ref(props.modelValue)

watch(() => props.modelValue, val => (localValue.value = val))
watch(localValue, val => emit('update:modelValue', val))
</script>
