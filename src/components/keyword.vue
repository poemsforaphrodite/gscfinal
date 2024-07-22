<script setup lang="ts">
import { ref } from 'vue'
import { Client } from '@gradio/client'
import { useColorMode } from '@vueuse/core'
import { Card, CardHeader, CardTitle, CardContent } from '../components/ui/card'
import { Label } from '../components/ui/label'
import { Textarea } from '../components/ui/textarea'
import { Input } from '../components/ui/input'
import { Button } from '../components/ui/button'
import { Table, TableHeader, TableBody, TableHead, TableRow, TableCell } from '../components/ui/table'
import { Icon } from '@iconify/vue'
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger } from '../components/ui/dropdown-menu'

const urls = ref<string>('')
const keywords = ref<string>('')
const result = ref<any>(null)
const mode = useColorMode()

const handleSubmit = async () => {
  try {
    const app = await Client.connect("poemsforaphrodite/keyword")
    const prediction = await app.predict("/predict", {
      urls: urls.value.split('\n').join('\n'),
      pasted_content: "",
      keywords: keywords.value,
      keywords_file: null,
      model_type: "Multilingual",
    }) as any
    console.log(prediction)
    // Bound relevancy score to 2 decimal points
    prediction.data[0].data = prediction.data[0].data.map((row: any) => [
      row[0],
      Number(row[1]).toFixed(2),
      row[2]
    ])
    result.value = prediction
  } catch (error) {
    console.error("Error during prediction:", error)
  }
}

const setColorMode = (newMode: 'light' | 'dark' | 'auto') => {
  mode.value = newMode
}
</script>
<template>
  <div class="container mx-auto p-4">
    <Card class="w-full max-w-2xl mx-auto">
      <CardHeader class="flex justify-between items-center">
        <CardTitle class="text-2xl">Welcome!</CardTitle>
        <DropdownMenu>
          <DropdownMenuTrigger as-child>
            <Button variant="outline" size="icon">
              <Icon icon="radix-icons:moon" class="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
              <Icon icon="radix-icons:sun" class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
              <span class="sr-only">Toggle theme</span>
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            <DropdownMenuItem @click="setColorMode('light')">
              Light
            </DropdownMenuItem>
            <DropdownMenuItem @click="setColorMode('dark')">
              Dark
            </DropdownMenuItem>
            <DropdownMenuItem @click="setColorMode('auto')">
              System
            </DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
      </CardHeader>
      <CardContent>
        <form @submit.prevent="handleSubmit">
          <div class="space-y-4">
            <div class="space-y-2">
              <Label for="urls" class="font-medium">URLs (one per line):</Label>
              <Textarea id="urls" v-model="urls" placeholder="Enter URLs..." />
            </div>
            <div class="space-y-2">
              <Label for="keywords" class="font-medium">Keywords:</Label>
              <Input id="keywords" v-model="keywords" placeholder="Enter keywords..." />
            </div>
            <Button type="submit" class="font-semibold">Submit</Button>
          </div>
        </form>

        <div v-if="result" class="mt-8">
          <h2 class="text-2xl font-bold mb-4">Prediction Results</h2>
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead class="font-semibold">Keyword</TableHead>
                <TableHead class="font-semibold">Relevance Score</TableHead>
                <TableHead class="font-semibold">Content</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              <TableRow v-for="(row, index) in result.data[0].data" :key="index">
                <TableCell>{{ row[0] }}</TableCell>
                <TableCell>{{ row[1] }}</TableCell>
                <TableCell>
                  <a :href="row[2]" target="_blank" rel="noopener noreferrer" class="text-blue-600 dark:text-blue-400 hover:underline">{{ row[2] }}</a>
                </TableCell>
              </TableRow>
            </TableBody>
          </Table>

          <h3 class="text-xl font-bold mt-6 mb-2">CSV File</h3>
          <a
            :href="result.data[1].url"
            download="relevance_scores.csv"
            target="_blank"
            rel="noopener noreferrer"
            class="text-blue-600 dark:text-blue-400 hover:underline font-medium"
          >
            Download CSV
          </a>
        </div>
      </CardContent>
    </Card>
  </div>
</template>

<style>
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/font.otf') format('opentype');
  font-weight: normal;
  font-style: normal;
}

body {
  font-family: 'CustomFont', 'Arial', sans-serif;
}
</style>