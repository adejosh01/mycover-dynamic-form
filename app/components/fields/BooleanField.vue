<template>
  <UFormItem
    :label="field?.['x-label']"
    :required="required"
    :error="error"
  >
    <div class="flex gap-4">
      <label
        v-for="opt in field?.['x-source']?.data || []"
        :key="opt.value"
        class="flex items-center gap-2"
      >
        <input
          type="radio"
          :value="opt.value"
          :checked="opt.value === localValue"
          @change="onChange(opt.value)"
        >
        {{ opt.label }}
      </label>
    </div>
  </UFormItem>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

const props = defineProps({
  modelValue: [String, Number, Boolean],
  field: Object,
  error: String,
  required: Boolean
})
const emit = defineEmits(['update:modelValue'])

const localValue = ref(props.modelValue)

watch(() => props.modelValue, val => (localValue.value = val))

const onChange = (value: any) => {
  localValue.value = value
  emit('update:modelValue', value)
}
</script>
