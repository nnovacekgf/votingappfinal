<template>
  <div class="app">
    <h1>Contador de Votos</h1>
    <input type="file" @change="handleFileUpload" accept=".csv,.xls,.xlsx,.json" />
    <div v-if="error" class="error">{{ error }}</div>
    <div v-if="data.length">
      <h2>Dados Carregados</h2>
      <ul>
        <li v-for="(item, index) in data" :key="index">{{ item }}</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import * as XLSX from 'xlsx'

const data = ref([])
const error = ref('')

function handleFileUpload(event) {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    const result = e.target.result
    try {
      let workbook
      if (file.name.endsWith('.json')) {
        data.value = JSON.parse(result)
      } else {
        const binary = new Uint8Array(result)
        workbook = XLSX.read(binary, { type: 'array' })
        const sheetName = workbook.SheetNames[0]
        const worksheet = workbook.Sheets[sheetName]
        data.value = XLSX.utils.sheet_to_json(worksheet, { header: 1 })
      }
    } catch (err) {
      error.value = 'Erro ao processar o arquivo.'
    }
  }
  if (file.name.endsWith('.json')) {
    reader.readAsText(file)
  } else {
    reader.readAsArrayBuffer(file)
  }
}
</script>

<style scoped>
.app {
  max-width: 600px;
  margin: auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}
.error {
  color: red;
  margin-top: 10px;
}
</style>