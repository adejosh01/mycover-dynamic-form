<template>
  <div class="min-h-screen app-root dark bg-gray-900 text-gray-100">
    <header class="border-b">
      <div class="max-w-4xl mx-auto py-6 px-4 flex items-center gap-4">
        <div>
          <h1 class="text-2xl font-semibold text-gray-800 dark:text-gray-100">
            Dynamic Forms
          </h1>
          <p class="text-sm text-gray-500 dark:text-gray-400">
            Auto-generated form preview
          </p>
        </div>
      </div>
    </header>

    <main class="max-w-4xl mx-auto py-10 px-4">
      <section class="mb-6">
        <label class="block text-sm font-medium text-gray-200 mb-2"
          >Upload JSON schema</label
        >
        <div class="flex items-center gap-3">
          <input
            id="schemaFile"
            type="file"
            accept="application/JSON,.json"
            @change="onFileChange"
            class="form-control w-auto p-1"
          />

          <button
            v-if="schema"
            @click="clearSchema"
            class="px-3 py-1 rounded border text-gray-200"
          >
            Clear
          </button>
        </div>
        <p v-if="fileName" class="text-xs muted mt-2">
          Selected: {{ fileName }}
        </p>
        <p v-if="uploadError" class="text-xs text-red-400 mt-2">
          {{ uploadError }}
        </p>
        <button
          :disabled="!pendingSchema"
          @click="loadUploaded"
          :class="
            pendingSchema
              ? 'px-3 py-1 rounded bg-green-600 mt-6 text-white'
              : 'px-3 py-1 rounded my-4 bg-gray-600 mt-6 text-gray-300 cursor-not-allowed'
          "
        >
          Load uploaded
        </button>
      </section>

      <div v-if="schema">
        <h2 v-if="(schema as any)?.title" class="text-xl font-semibold mb-2">
          {{ (schema as any).title }}
        </h2>
        <p
          v-if="(schema as any)?.description"
          class="text-sm text-gray-600 mb-4"
        >
          {{ (schema as any).description }}
        </p>
        <DynamicForm :schema="schema" />
      </div>
      <div v-else class="text-sm muted">
        Upload a JSON schema or click "Load sample" to preview the form.
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import DynamicForm from "@/components/DynamicForm.vue";

const schema = ref(null);
const fileName = ref("");
const uploadError = ref("");
const pendingSchema = ref(null);

// async function loadSample() {
//   try {
//     const res = await fetch('/sample-schema.json')
//     schema.value = await res.json()
//     fileName.value = 'sample-schema.json'
//     uploadError.value = ''
//   } catch (e) {
//     uploadError.value = 'Failed to load sample schema.'
//   }
// }

function clearSchema() {
  schema.value = null;
  fileName.value = "";
  uploadError.value = "";
}

function onFileChange(e: Event) {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (!file) return;
  fileName.value = file.name;
  const reader = new FileReader();
  reader.onload = () => {
    try {
      const txt = reader.result as string;
      const parsed = JSON.parse(txt);
      // store parsed file as pending until user clicks 'Load uploaded'
      pendingSchema.value = parsed;
      uploadError.value = "";
    } catch (err) {
      uploadError.value = "Invalid JSON file. Please upload a valid schema.";
    }
  };
  reader.readAsText(file);
}

function loadUploaded() {
  if (!pendingSchema.value) return;
  schema.value = pendingSchema.value;
  uploadError.value = "";
}
</script>
