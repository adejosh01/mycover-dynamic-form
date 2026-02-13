<template>
  <div class="card p-6 rounded-lg">
    <form
      class="space-y-4"
      @submit.prevent="submitForm"
    >
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
          <FieldWrapper
            v-for="(item, idx) in visibleFields"
            :key="item[0]"
            :model-value="formState[item[0]]"
            :field="item[1]"
            :component-type="resolveComponent(item[1])"
            :required="isRequired(item[0], item[1])"
            :error="errors[item[0]]"
            :form-state="formState"
            @update:model-value="value => (formState[item[0]] = value)"
          />
      </div>

      <div class="flex items-center justify-end">
        <button
          type="submit"
          class="inline-flex items-center gap-2 bg-linear-to-r from-green-500 to-green-600 hover:from-green-600 hover:to-green-700 text-white font-medium px-5 py-2 rounded-lg shadow"
        >
          Submit
        </button>
      </div>
    </form>

    <div
      v-if="submittedData"
      class="mt-4 p-4 bg-gray-800 text-gray-100 rounded"
    >
      <h3 class="font-semibold mb-2">
        Submitted data
      </h3>
      <pre class="text-sm overflow-auto">{{ JSON.stringify(submittedData, null, 2) }}</pre>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, computed, watch, ref } from 'vue'
import dayjs from 'dayjs'

import FieldWrapper from './fields/FieldWrapper.vue'
import TextField from './fields/TextField.vue'
import NumberField from './fields/NumberField.vue'
import DateField from './fields/DateField.vue'
import FileField from './fields/FileField.vue'
import SelectField from './fields/SelectField.vue'
import BooleanField from './fields/BooleanField.vue'

const props = defineProps({ schema: Object })

const formState = reactive<Record<string, any>>({})
const errors = reactive<Record<string, string | undefined>>({})
const submittedData = ref(null)

const sortedFields = computed<[string, Record<string, any>][]>(() => {
  return Object.entries(props.schema?.properties || {}).sort(
    ([, a], [, b]) => ((a as Record<string, any>)['x-order'] ?? 999) - ((b as Record<string, any>)['x-order'] ?? 999)
  ) as [string, Record<string, any>][]
})

const visibleFields = computed<[string, Record<string, any>][]>(() => {
  return sortedFields.value.filter(([key, field]) => isVisible(key, field))
})

const resolveComponent = (field: Record<string, any>) => {
  const type = field['x-source']?.type
  if (type === 'array') return SelectField
  if (type === 'file') return FileField
  if (type === 'date') return DateField
  if (type === 'number') return NumberField
  if (field.type === 'boolean') return BooleanField
  return TextField
}

const isRequired = (key: string, field: Record<string, any>): boolean => {
  // 1) global required in schema
  if (props.schema?.required?.includes(key)) return true

  // 2) support top-level if-then-else (simple const checks)
  try {
    const ifBlock = props.schema?.if
    if (ifBlock && ifBlock.properties) {
      // check each property condition in if.properties
      let matched = true
      for (const [propName, cond] of Object.entries(ifBlock.properties)) {
        if ((cond as { const?: any }).const !== undefined) {
          if (formState[propName] !== (cond as { const: any }).const) {
            matched = false
            break
          }
        }
      }
      if (matched) {
        if (props.schema?.then?.required?.includes(key)) return true
      } else {
        if (props.schema?.else?.required?.includes(key)) return true
      }
    }
  } catch {
    // ignore and continue
  }

  // 3) field-level x-required_if
  const rule = field?.['x-required_if']
  if (!rule) return false

  const dep: string = rule?.field
  const expected = rule?.value
  const operator = rule?.operator || 'eq'

  const actual = formState[dep]
  if (operator === 'eq') return actual === expected
  if (operator === 'neq') return actual !== expected
  return false
}

const isVisible = (key: string, field: Record<string, any>): boolean => {
  if (!field) return true
  // Use x-required_if as visibility rule as well
  const rule = field?.['x-required_if']
  if (!rule) return true

  const dep: string = rule?.field
  const expected = rule?.value
  const operator = rule?.operator || 'eq'

  const actual = formState[dep]
  if (operator === 'eq') return actual === expected
  if (operator === 'neq') return actual !== expected
  return true
}

const validateForm = () => {
  Object.keys(errors).forEach(k => {
    errors[k] = undefined
  })
  for (const [key, field] of sortedFields.value) {
    const value = formState[key]
    if (isRequired(key, field) && (!value || value === '')) {
      errors[key] = `${field['x-label']} is required`
    }
    if (field.pattern && value && !new RegExp(field.pattern).test(value)) {
      errors[key] = field.errorMessage?.pattern || 'Invalid format'
    }
    if (field.minLength && value?.length < field.minLength) {
      errors[key] = `${field['x-label']} must be at least ${field.minLength} chars`
    }
    if (field['x-source']?.type === 'date' && value && field['x-source'].data?.minAge) {
      const age = dayjs().diff(dayjs(value), 'year')
      if (age < field['x-source'].data.minAge) {
        errors[key] = `Must be at least ${field['x-source'].data.minAge} years old`
      }
    }
  }
  return Object.values(errors).filter(Boolean).length === 0
}

const submitForm = () => {
  if (!validateForm()) return
  const payload = JSON.parse(JSON.stringify(formState))
  console.log('Form Submitted:', payload)
  submittedData.value = payload
}

// dependent dropdowns
watch(() => formState, () => {
  for (const [key, field] of sortedFields.value) {
    // dependent dropdowns: clear value if no longer present
    if (field['x-source']?.data?.dependsOn) {
      const dep = field['x-source'].data.dependsOn
      const options = field['x-source'].data.options[formState[dep]] || []
      if (!options.find((o: any) => o.value === formState[key])) {
        formState[key] = null
        errors[key] = undefined
      }
    }

    // visibility: clear value and errors when field is hidden
    if (!isVisible(key, field)) {
      if (formState[key] !== undefined) {
        formState[key] = null
      }
      errors[key] = undefined
    }
  }
}, { deep: true })

// show submitted data below the form for quick inspection
// rendered as formatted JSON when present
</script>
