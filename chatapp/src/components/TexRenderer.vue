<template>
    <div ref="container" class="math-container"></div>
  </template>
  
  <script setup>
  import { onMounted, ref, defineProps } from 'vue';
  import katex from 'katex';
  import 'katex/dist/katex.min.css'; // KaTeX のスタイルをインポート
  
  const props = defineProps({
    formula: {
      type: String,
      required: true
    }
  });
  
  const container = ref(null);
  
  onMounted(() => {
    if (container.value) {
      try {
        // KaTeX を使って数式をレンダリング
        katex.render(props.formula, container.value, {
          throwOnError: false,
          // ここでカスタマイズオプションを設定できます
          displayMode: true, // ブロック数式として表示
        });
      } catch (err) {
        console.error('KaTeX error:', err);
      }
    }
  });
  </script>
  
  <style scoped>
  .math-container {
    font-size: 16px;
    margin: 10px 0;
  }
  </style>
  