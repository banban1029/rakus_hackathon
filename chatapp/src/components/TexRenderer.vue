<template>
    <span v-html="renderedFormula"></span>
  </template>
  
  <script setup>
  import { ref, watch } from 'vue'
  import katex from 'katex'
  import 'katex/dist/katex.min.css'
  
  const props = defineProps({
    formula: String,
  })
  
  const renderedFormula = ref('')
  
  // LaTeXをレンダリングする関数
  const renderFormula = (formula) => {
    try {
      return katex.renderToString(formula, {
        throwOnError: false,
      })
    } catch (error) {
      console.error('Error rendering formula:', error)
      return formula
    }
  }
  
  // フォーミュラの変更を監視し、再レンダリングする
  watch(() => props.formula, (newFormula) => {
    renderedFormula.value = renderFormula(newFormula)
  }, { immediate: true })
  </script>
  